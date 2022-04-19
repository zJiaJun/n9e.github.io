---
weight: 2
title: "安装部署"
---

如果您对Docker的使用非常熟悉，建议利用Docker compose的方式快速启动测试，请参考[使用Docker Compose快速部署]({{%relref "compose"%}})，如果对Docker不熟悉，那就用二进制方式部署，也非常简单，最小的可运行环境是Prometheus+MySQL+Redis+Nightingale，请参考[快速在生产环境部署启动单机版]({{%relref "standalone"%}})。这个最小的环境只有Prometheus采集到的自身的一些监控指标，略显单薄，此时，我们可以引入Telegraf，采集机器、网络设备、各类中间件的指标，请参考[使用Telegraf采集监控数据]({{%relref "telegraf"%}})。


如果您运行和维护着K8s集群，并且希望把Nightingale部署到K8s中，我们推荐您使用[n9e-helm-chart](https://github.com/flashcatcloud/n9e-helm)来部署、升级、管理。


如果公司体量很大，建议把单机版本的Prometheus替换为VictoriaMetrics，请参考[使用VictoriaMetrics作为时序库]({{%relref "victoriametrics"%}})。或者直接部署多个Prometheus，按照业务线或者按照地域来划分集群，此时你可能需要[接入多个Prom/VM/M3DB集群]({{%relref "multitsdb"%}})，在引入多个TSDB的过程中，就要同步使用夜莺的多Server部署模型了，请参考[生产环境部署高可用集群版]({{%relref "clusters"%}})。
