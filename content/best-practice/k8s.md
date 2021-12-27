---
weight: 250
title: "监控Kubernetes"
---

## kube-prometheus

[kube-prometheus](https://github.com/prometheus-operator/kube-prometheus) 这个项目大家可以参考下。Kubernetes集群本身的运维管理，应该由一个专门的团队来做，这个专门的团队应该比较资深，对Kubernetes、对Prometheus都是比较熟悉的，用operator这种方式信手拈来。所以，在这个场景下，其实不太需要用到夜莺，那如果大家仍然想用夜莺来配置Kubernetes的告警规则、查看Kubernetes相关的监控数据，也没啥问题，只要把使用kube-prometheus启动的Prometheus接入夜莺即可。

后面，我们会针对Kubernetes场景在夜莺里做专门的功能支持，不过目前尚未开始，这里把kube-prometheus项目中的各类告警规则，整理转换成了夜莺系统中可导入的告警规则JSON，大家如果需要的话可以自取。

- [Alertmanager](/json/Alertmanager.json)
- [KubeControlPlane](/json/KubeControlPlane.json)
- [KubePrometheus](/json/KubePrometheus.json)
- [KubeStateMetrics](/json/KubeStateMetrics.json)
- [NodeExporter](/json/NodeExporter.json)
- [PromOperator](/json/PromOperator.json)
- [PrometheusSelf](/json/PrometheusSelf.json)

如果用夜莺来做告警，其实就不需要Alertmanager了，所以下面kube-prometheus项目中提供的Alertmanager相关的告警规则，其实也用不上了，不过还是放在了这里，大家看自己的场景需求吧。

另外，kube-operator项目的告警规则，很多用到了recording rule，这些recording rule还是需要继续在Prometheus中配置，夜莺不提供recording rule的UI化管理。

## Telegraf

Telegraf监控Kubernetes有两个input plugin大家可以看看，一个是[kube_inventory](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/kube_inventory)，一个是[kubernetes](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/kubernetes) 具体我还没有研究，大家如果有已经研究过的，欢迎写文章分享哈
