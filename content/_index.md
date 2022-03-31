+++
title = "夜莺手册"
+++


## 夜莺简介

夜莺（ Nightingale ）是一款国产开源、云原生监控系统，Nightingale 在 2020.3.20 发布 v1 版本，目前是 v5 版本，从这个版本开始，与 Prometheus、VictoriaMetrics、Grafana、Telegraf、Datadog 等生态做了协同集成，力争打造国内最好用的开源运维监控系统。出自 Open-Falcon 研发团队。

## 项目代码

- 后端：[💡 https://github.com/didi/nightingale](https://github.com/didi/nightingale)
- 前端：[💡 https://github.com/n9e/fe-v5](https://github.com/n9e/fe-v5)

前后端都是开源的，如果觉得不错，欢迎 star 一下，给我们持续坚持的动力！

## 产品截图

查看监控数据，即监控大盘页面：

![](/intro-dash.png)

配置告警规则的列表页面：

![](/intro-alerts.png)

活跃告警列表页面，即当前未恢复的告警页面：

![](/intro-events.png)

#### 与 Open-Falcon 的区别

因为开发 Open-Falcon 和 Nightingale 的是一拨人，所以很多社区伙伴会比较好奇，为何要新做一个监控开源软件。核心点是 Open-Falcon 和 Nightingale 的差异点实在是太大了，Nightingale 并非是 Open-Falcon 设计逻辑的一个延续，就看做两个不同的软件就好。

Open-Falcon 是 14 年开发的，当时是想解决 Zabbix 的一些容量问题，可以看做是物理机时代的产物，整个设计偏向运维视角，虽然数据结构上已经开始设计了标签，但是查询语法还是比较简单，无法应对比较复杂的场景。

Nightingale 直接支持 PromQL，支持 Prometheus、M3DB、VictoriaMetrics 多种时序库，支持 Telegraf、Datadog-Agent、Grafana-Agent 做监控数据采集，支持 Grafana 看图，整个设计更加云原生。

#### 与 Prometheus 的区别

Nightingale 可以简单看做是 Prometheus 的一个企业级版本，把 Prometheus 当做 Nightingale 的一个内部组件（时序库），当然，也不是必须的，时序库除了 Prometheus，还可以使用 VictoriaMetrics、M3DB 等，各种 Exporter 采集器也可以继续使用。

Nightingale 可以接入多个 Prometheus，可以允许用户在页面上配置告警规则、屏蔽规则、订阅规则，在页面上查看告警事件、做告警事件聚合统计，配置告警自愈机制，管理监控对象，配置监控大盘等，就把 Nightingale 看做是 Prometheus 的一个 WEBUI 也是可以的，不过实际上，它远远不止是一个 WEBUI，用一下就会深有感触。

## 系统架构

夜莺 v5 的设计非常简单，核心是 server 和 webapi 两个模块，webapi 无状态，放到中心端，承接前端请求，将用户配置写入数据库；server 是告警引擎和数据转发模块，一般随着时序库走，一个时序库就对应一套 server，每套 server 可以只用一个实例，也可以多个实例组成集群，server 可以接收 Telegraf、Grafana-Agent、Datadog-Agent、Falcon-Plugins 上报的数据，写入后端时序库，周期性从数据库同步告警规则，然后查询时序库做告警判断。每套 server 依赖一个 redis。架构图如下：

![](/intro-arch.png)

## 加入社区

微信公众号:`cloudmon`（[云原生监控](/cloudmon.png)）这是夜莺的大本营，可以在公众号菜单里找到加群入口、社区答疑入口，关注起来吧！我们团队做运维监控这个事情差不多有 10 年了，一直在坚持，希望能把这个事做到极致，欢迎加入我们一起！
