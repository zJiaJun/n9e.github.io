---
title: "快速在生产环境部署启动单机版"
weight: 2
---

> 本节讲述如何部署单机版，单机版对于很多中小公司足够用了，简单高效、快速直接，建议使用云主机，性能不够了直接升配，可以应对每秒上报的数据点小于100万的情形，如果只是监控机器（每台机器每个周期大概采集200个数据点）采集周期频率设置10秒的话，支撑上限是5万台

如果仅仅是为了快速测试，Docker部署方式是最快的，不过很多朋友未必有Docker环境，另外为了减少引入更多技术栈，增强生产环境稳定性，有些朋友可能也不愿意用Docker，那本篇就来讲解如何快速部署单机版，单机版的配套时序库是使用Prometheus。如果要监控的机器有几千台，服务有几百个，单机版的容量无法满足，可以上集群版，集群版的时序库建议使用VictoriaMetrics，也可以使用M3DB，不过M3DB的架构更复杂，很多朋友无法搞定，选择简单的VictoriaMetrics，对大部分公司来讲，足够用了。我们先来看一下服务端架构：

![夜莺服务端架构](/n9e-arch-server.png)

- 核心模块server：server是用来做告警的，会从数据库中同步告警规则，然后读取Prometheus的数据做告警判断。server也可以接收监控数据上报，然后通过remote write协议写入多个时序库。server也依赖redis，用redis存储了server本身以及监控对象的心跳信息
- 核心模块webapi：提供restful api，用于和前端JavaScript交互，把一些用户配置类的信息写入mysql，鉴权采用jwt，jwt的token使用redis存储，在单机部署的方式下，server的redis和webapi的redis可以复用

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

tarball=n9e-5.0.0-ga-05.tar.gz
urlpath=https://github.com/didi/nightingale/releases/download/v5.0.0-ga-05/${tarball}
wget $urlpath || exit 1

tar zxvf ${tarball}

mysql -uroot -p1234 < docker/initsql/a-n9e.sql

nohup ./n9e server &> server.log &
nohup ./n9e webapi &> webapi.log &

# check logs
# check port
```

如果启动成功，server默认会监听在19000端口，webapi会监听在18000端口，且日志没有报错。上面使用nohup简单演示，生产环境建议用systemd托管，相关service文件可以在etc/service目录下找到。


配置文件etc/server.conf和etc/webapi.conf中都含有mysql的连接地址配置，检查一下用户名和密码，prometheus如果使用上面的脚本安装，默认会监听本机9090端口，server.conf和webapi.conf中的prometheus相关地址都不用修改就是对的。

好了，浏览器访问webapi的端口（默认是18000）就可以体验相关功能了，默认用户是`root`，密码是`root.2020`。如果安装过程出现问题，可以参考 [视频教程](https://www.bilibili.com/video/BV1HL4y1H7Yc/) 

接下来，你可能需要：

- [安装Telegraf采集更多监控数据]({{%relref "telegraf"%}})


