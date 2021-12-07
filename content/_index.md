+++
title = "夜莺手册"
+++


> 夜莺是新一代国产智能监控系统。对云原生场景、传统物理机虚拟机场景，都有很好的支持，10分钟完成搭建，1小时熟悉使用，经受了滴滴生产环境海量数据的验证，希望打造国产监控的标杆之作，一起参与进来吧！

## 新版简介

Nightingale在2020.3.20发布v1版本，目前是v5.0版本，从这个版本开始，与Prometheus、VictoriaMetrics、Grafana、Telegraf等生态做了协同集成，力争打造国内最好用的开源运维监控系统。

#### 与Open-Falcon的区别

因为开发Open-Falcon和Nightingale的是一拨人，所以很多社区伙伴会比较好奇，为何要新做一个监控开源软件。核心点是Open-Falcon和Nightingale的差异点实在是太大了，Nightingale并非是Open-Falcon设计逻辑的一个延续，就看做两个不同的软件就好。

Open-Falcon是14年开发的，当时是想解决Zabbix的一些容量问题，可以看做是物理机时代的产物，整个设计偏向运维视角，虽然数据结构上已经开始设计了标签，但是查询语法还是比较简单，无法应对比较复杂的场景。

Nightingale直接支持PromQL，支持Prometheus、M3DB、VictoriaMetrics多种时序库，支持Telegraf做监控数据采集，支持Grafana看图，整个设计更加云原生，虽然也保留了机器归组的逻辑以应对物理机时代的需求，但是设计上，更倾向于使用标签来分组，而不是HostGroup或者树形结构。

#### 与Prometheus的区别

Nightingale可以简单看做是Prometheus的一个企业级版本，把Prometheus当做Nightingale的一个内部组件-时序库，当然，也不是必须的，时序库除了Prometheus，还可以使用VictoriaMetrics、M3DB等。各种Exporter也可以继续使用，不过我们更推荐使用All-in-one的Telegraf，运维代价会更小一些。

Nightingale可以接入多个Prometheus/M3DB/VictoriaMetrics，可以允许用户在页面上配置告警规则、屏蔽规则、订阅规则，在页面上查看告警事件，配置告警自愈机制，管理监控对象，配置监控大盘等，就把Nightingale看做是Prometheus的一个WEBUI也是可以的，不过实际上，它远远不止是一个WEBUI，用一下就会深有感触。

## 项目代码

- 后端：[💡 https://github.com/didi/nightingale](https://github.com/didi/nightingale)
- 前端：[💡 https://github.com/n9e/fe-v5](https://github.com/n9e/fe-v5)

## 系统架构

夜莺5.1的设计非常简单，核心是server和webapi两个模块，webapi无状态，放到中心端，承接前端请求，将用户配置写入数据库；server是告警引擎和数据转发模块，一般随着时序库走，一个时序库就对应一套server，每套server可以只用一个server实例，也可以多个实例组成集群，server可以接收Telegraf上报的数据，写入后端时序库，周期性从数据库同步告警规则，然后查询时序库做告警判断。每套server依赖一个redis。架构图如下：

![](/n9e-arch-server.png)

## 加入社区

- 微信公号:`__n9e__`（夜莺监控）
- 知识星球：夜莺开源社区

钉钉交流群：

![](/dingtalk.png?width=350)
