---
title: "生产环境部署高可用集群版"
weight: 9
---

对于规模相对较小的公司，比如几百台机器这个体量，个人认为[单机版]({{%relref "standalone" %}})足够用了，使用云主机部署，性能不足可以直接升配，存储使用云存储保证，硬件故障云平台也会自动把虚拟机热迁移走，非常省心。那如果咱们体量确实比较大，或者没有云主机这种基础设施，这里会讲解集群版的部署方式。

## 单中心，单集群，接入公司所有数据

首先，来讲解一种普适性比较高的架构方式，就是单中心大集群模式。比如某公司，虽然有3个机房，但是相互之间有专线打通，链路很好，为了便于维护，倾向于在中心搭建一套监控集群，其他机房的机器上只需要部署监控客户端（推荐[Telegraf]({{%relref "telegraf" %}})）即可，架构图如下：

![夜莺单中心架构方式](/n9e-single-center.png)

图中时序库使用Prometheus表示，当然，也可以选用[VictoriaMetrics]({{%relref "victoriametrics" %}})或者[M3DB](https://m3db.io/)等，n9e-server部署了3台机器，n9e-webapi也部署了3台机器，当然，可以混部，即一台机器上既部署n9e-server，也部署n9e-webapi，监控客户端Telegraf要上报监控数据所以要调用n9e-server的接口，为了高可用，可以在n9e-server前面架设负载均衡，比如LVS，[Telegraf]({{%relref "telegraf" %}})的output plugin配置vip即可。负责与前端交互的n9e-webapi也部署了多个，为了高可用也是在前面架设负载均衡，这样终端用户访问的时候，也是访问vip，某个n9e-webapi如果挂掉，负载均衡可以自动感知并摘除，用户无感。

图中redis画了两个，n9e-webapi依赖一个redis，n9e-server也依赖一个redis，在单中心、单集群模式下，可以复用一个redis。redis采用的是标准版，非集群版、非Sentinel版，这点大家要注意。

## 多套存储，多套server集群，单套webapi集群

在[接入多个Prom/VM/M3DB集群]({{%relref "multitsdb" %}})一节中，介绍了夜莺可以接入多个时序库，在夜莺的架构中，时序库和n9e-server是一一对应的，每接入一套时序库，就一定要部署一套n9e-server，所谓的这一套n9e-server，可以是单实例模式，也可以是多实例模式。

比如在北京机房部署了一套Prometheus时序库，就需要在北京机房部署一套n9e-server，在广州机房部署了一套VictoriaMetrics，就需要在广州机房也部署一套n9e-server，每套n9e-server都要对应一个redis，此时，ne9-server的redis不建议与n9e-webapi的redis复用（虽然从原理上也支持），建议n9e-server的redis部署到n9e-server所在的机房，避免机房割裂时n9e-server连不上redis。