---
title: "使用二进制部署单机版服务端"
weight: 2
---

> 本节讲述如何部署单机版，单机版对于很多中小公司足够用了，简单高效、快速直接，建议使用云主机，性能不够了直接升配，可以应对每秒上报的数据点小于100万的情形，如果只是监控机器（每台机器每个周期大概采集200个数据点）采集周期频率设置10秒的话，支撑上限是5万台

如果仅仅是为了快速测试，Docker 部署方式是最快的，不过很多朋友未必有 Docker 环境，另外为了减少引入更多技术栈，增强生产环境稳定性，有些朋友可能也不愿意用 Docker，那本篇就来讲解如何快速部署单机版，单机版的配套时序库是使用 Prometheus。如果要监控的机器有几千台，服务有几百个，单机版的容量无法满足，可以上集群版，集群版的时序库建议使用 VictoriaMetrics，也可以使用 M3DB，不过 M3DB 的架构更复杂，很多朋友无法搞定，选择简单的 VictoriaMetrics，对大部分公司来讲，足够用了。我们先来看一下服务端架构：

![](/install/standalone.png?width=500px)

按照单机版本的这个架构图可以看出，服务端需要安装的组件有：MySQL、Redis、Prometheus、n9e-server、n9e-webapi，Agent 有多种选型，可以是 Telegraf、Datadog-Agent、Grafana-Agent 等，Agent 应该部署在所有的目标机器上，包括服务端的这台机器，Exporters 是指 Prometheus 生态的各类 Exporter 采集器，比如 mysqld_exporter、redis_exporter、blackbox_exporter 等，这些 Exporter 是非必须的，看各自公司的情况。

## 环境准备

依赖的组件有：mysql、redis、prometheus，这三个组件都是开源软件，请大家自行安装，其中prometheus在启动的时候要注意开启 `--enable-feature=remote-write-receiver` 这里也提供一个小脚本来安装这3个组件，大家可以参考：

```bash
# install prometheus
mkdir -p /opt/prometheus
wget https://s3-gz01.didistatic.com/n9e-pub/prome/prometheus-2.28.0.linux-amd64.tar.gz -O prometheus-2.28.0.linux-amd64.tar.gz
tar xf prometheus-2.28.0.linux-amd64.tar.gz
cp -far prometheus-2.28.0.linux-amd64/*  /opt/prometheus/

# service 
cat <<EOF >/etc/systemd/system/prometheus.service
[Unit]
Description="prometheus"
Documentation=https://prometheus.io/
After=network.target

[Service]
Type=simple

ExecStart=/opt/prometheus/prometheus  --config.file=/opt/prometheus/prometheus.yml --storage.tsdb.path=/opt/prometheus/data --web.enable-lifecycle --enable-feature=remote-write-receiver --query.lookback-delta=2m 

Restart=on-failure
SuccessExitStatus=0
LimitNOFILE=65536
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=prometheus


[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable prometheus
systemctl restart prometheus
systemctl status prometheus

# install mysql
yum -y install mariadb*
systemctl enable mariadb
systemctl restart mariadb
mysql -e "SET PASSWORD FOR 'root'@'localhost' = PASSWORD('1234');"

# install redis
yum install -y redis
systemctl enable redis
systemctl restart redis
```

上例中mysql的root密码设置为了1234，建议维持这个不变，后续就省去了修改配置文件的麻烦。

## 安装夜莺组件

```bash
mkdir -p /opt/n9e && cd /opt/n9e

# 去 https://github.com/didi/nightingale/releases 找最新版本的包，文档里的包地址可能已经不是最新的了
tarball=n9e-5.5.0.tar.gz
urlpath=https://github.com/didi/nightingale/releases/download/v5.5.0/${tarball}
wget $urlpath || exit 1

tar zxvf ${tarball}

mysql -uroot -p1234 < docker/initsql/a-n9e.sql

nohup ./n9e server &> server.log &
nohup ./n9e webapi &> webapi.log &

# check logs
# check port
```

如果启动成功，server 默认会监听在 19000 端口，webapi 会监听在 18000 端口，且日志没有报错。上面使用 nohup 简单演示，生产环境建议用 systemd 托管，相关 service 文件可以在 etc/service 目录下，供参考。

配置文件`etc/server.conf`和`etc/webapi.conf`中都含有 mysql 的连接地址配置，检查一下用户名和密码，prometheus 如果使用上面的脚本安装，默认会监听本机 9090 端口，server.conf 和 webapi.conf 中的 prometheus 相关地址都不用修改就是对的。

好了，浏览器访问 webapi 的端口（默认是18000）就可以体验相关功能了，默认用户是`root`，密码是`root.2020`。如果安装过程出现问题，可以参考公众号（云原生监控）的视频教程。

夜莺服务端部署好了，接下来要考虑监控数据采集的问题，如果是 Prometheus 重度用户，可以继续使用各类 Exporter 来采集，只要数据进了时序库了，夜莺就能够消费（判断告警、展示图表等）；如果想快速看到效果，可以使用 Telegraf 来采集监控数据，请参考后续文档章节。
