---
title: "告警自愈依赖的脚本下发执行模块"
weight: 15
---

## 概述

所谓的告警自愈，典型手段是在告警触发时自动回调某个webhook地址，在这个webhook里写告警自愈的逻辑，夜莺默认支持这种方式。另外，夜莺还可以更进一步，配合ibex这个模块，在告警触发的时候，自动去告警的机器执行某个脚本，这种机制可以大幅简化构建运维自愈链路的工作量，毕竟，不是所有的运维人员都擅长写http server，但所有的运维人员，都擅长写脚本。这种方式是典型的物理机时代的产物，希望各位朋友用不到这个工具（说明贵司的IT技术已经走得非常靠前了）。

## 架构

ibex模块，类似之前夜莺v3版本中的job模块，可以批量执行脚本，其架构非常简单，包括server和agentd两个模块，agentd周期性调用server的rpc接口，询问有哪些任务要执行，如果有分配给自己的任务，就从server拿到任务脚本信息，在本地fork一个进程运行，然后将结果上报给服务端。为了简化部署，server和agentd融合成了一个二进制，就是ibex，通过传入不同的参数来启动不同的角色。ibex架构图如下：

![ibex架构图](/install/ibex.png?width=500px)

## 项目地址

- Git仓库：[https://gitee.com/cnperl/ibex](https://gitee.com/cnperl/ibex) 编译方法[看这里]({{%relref "compile" %}}) Linux 下编译好的包 [在这里](/tarball/ibex-1.0.0.tar.gz)

## 安装启动

下载安装包之后，解压缩，在etc下可以找到服务端和客户端的配置文件，在sql目录下可以找到初始化sql脚本。

#### 初始化sql

```bash
mysql < sql/ibex.sql
```

#### 启动server

server的配置文件是`etc/server.conf`，注意修改里边的mysql连接地址，配置正确的mysql用户名和密码。然后就可以直接启动了：

```bash
nohup ./ibex server &> server.log &
```

ibex没有web页面，只提供api接口，鉴权方式是http basic auth，basic auth的用户名和密码默认都是ibex，在`etc/server.conf`中可以找到，如果ibex部署在互联网，一定要修改默认用户名和密码，当然，因为n9e要调用ibex，所以n9e的server.conf和webapi.conf中也配置了ibex的basic auth账号信息，要改就要一起改啦。

#### 启动agentd

客户端的配置非常非常简单，agentd.conf内容如下：

```toml
# debug, release
RunMode = "release"

# task meta storage dir
MetaDir = "./meta"

[HTTP]
Enable = true
# http listening address
Host = "0.0.0.0"
# http listening port
Port = 2090
# https cert file path
CertFile = ""
# https key file path
KeyFile = ""
# whether print access log
PrintAccessLog = true
# whether enable pprof
PProf = false
# http graceful shutdown timeout, unit: s
ShutdownTimeout = 30
# max content length: 64M
MaxContentLength = 67108864
# http server read timeout, unit: s
ReadTimeout = 20
# http server write timeout, unit: s
WriteTimeout = 40
# http server idle timeout, unit: s
IdleTimeout = 120

[Heartbeat]
# unit: ms
Interval = 1000
# rpc servers
Servers = ["10.2.3.4:20090"]
# $ip or $hostname or specified string
Host = "telegraf01"
```

客户端的HTTP接口用处不大，可以把Enable设置为false，关闭监听，重点关注Heartbeat这个部分，Interval是心跳频率，默认是1000毫秒，如果机器量比较小，比如小于1000台，维持1000没问题，如果机器量比较大，可以适当调大这个频率，比如2000或者3000，可以减轻服务端的压力。Servers是个数组，配置的是ibex-server的地址，ibex-server可以启动多个，多个地址都配置到这里即可，Host这个字段，是本机的唯一标识，有三种配置方式，如果配置为`$ip`，系统会自动探测本机的IP，如果是`$hostname`，系统会自动探测本机的hostname，如果是其他字符串，那就直接把该字符串作为本机的唯一标识。每个机器上都要部署ibex-agentd，不同的机器要保证Host字段获取的内容不能重复。

另外，Telegraf的配置文件中，有下面这么一段：

```toml
[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  hostname = ""
  omit_hostname = false
```

其中hostname默认留空，表示自动探测本机的hostname，如果写了具体的某个字符串，那就把写的那个字符串作为监控数据的host字段的内容，这个hostname字段要和ibex的agentd.conf中的Host字段保持一致，典型的做法有：

- Telegraf中把hostname配置为空，Telegraf自动获取本机hostname，ibex的Host配置为`$hostname`，ibex也会自动获取本机hostname，这样Telegraf和ibex可以获取到相同的标识内容
- Telegraf中手工把hostname配置为本机的ip，ibex则把Host配置为`$ip`，这样二者也可以获取到相同的标识内容
- Telegraf和ibex都使用某个特定写死的字符串来作为标识信息，这样也OK，但是要保证不同的机器，这个字符串不能重复

下面是启动ibex-agentd的命令：

```bash
nohup ./ibex agentd &> agentd.log &
```

另外，细心的读者应该会发现ibex的压缩包里的etc目录下有个service目录，里边准备好了两个service样例文件，便于大家使用systemd来管理ibex进程，生产环境，建议使用systemd来管理。
