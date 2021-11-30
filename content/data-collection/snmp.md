---
weight: 50
title: "使用SNMP采集网络设备的指标"
---

Telegraf内置支持snmp的采集，本节给一个入门例子，让大家快速上手，更多具体知识可以参考[这里](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/snmp)。在telegraf.conf中搜索inputs.snmp，即可找到对应的配置，例子如下：


```toml
[[inputs.snmp]]
agents = ["udp://172.25.79.194:161"]
timeout = "5s"
version = 3
agent_host_tag = "ident"
retries = 1
sec_name = "managev3user"
auth_protocol = "SHA"
auth_password = "example.Demo.c0m"

[[inputs.snmp.field]]
oid = "RFC1213-MIB::sysUpTime.0"
name = "uptime"

[[inputs.snmp.field]]
oid = "RFC1213-MIB::sysName.0"
name = "source"
is_tag = true

[[inputs.snmp.table]]
oid = "IF-MIB::ifTable"
name = "interface"
inherit_tags = ["source"]

[[inputs.snmp.table.field]]
oid = "IF-MIB::ifDescr"
name = "ifDescr"
is_tag = true
```

上面非常关键的部分是：`agent_host_tag = "ident"`，因为夜莺对ident这个标签会特殊对待处理，把携有这个标签的数据当做隶属某个监控对象的数据，机器和网络设备都是典型的期望作为监控对象来管理的，所以snmp的采集中，我们把网络设备的ip放到ident这个标签里带上去。

另外这个采集规则是v3的校验方法，不同的公司可能配置的校验方式不同，请各位参照telegraf.conf中那些snmp相关的注释仔细核对，如果是v2会简单很多，把上例中的如下部分：

```toml
version = 3
sec_name = "managev3user"
auth_protocol = "SHA"
auth_password = "example.Demo.c0m"
```

换成：

```toml
version = 2
community = "public"
```

即可，当然了，community要改成你们自己的，这里写的public只是举个例子。

`inputs.snmp.field`相关的那些配置，可以采集到各个网口的监控指标，更多的使用方式请参考[官网](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/snmp)

---

另外，snmp的采集，建议大家使用专门的Telegraf来做，因为和机器、中间件等的采集频率可能不同，比如边缘交换机，我们5min采集一次就够了，如果按照默认的配置可是10s采集一次，实在是太频繁了，可能会给一些老式交换机造成比较大的压力，采集频率在telegraf.conf的最上面`[agent]`部分，边缘交换机建议配置为：

```toml
[agent]
interval = "300s"
flush_interval = "300s"
```

核心交换机可以配置的频繁一些，比如60s或者120s，请各位网络工程师朋友自行斟酌。

