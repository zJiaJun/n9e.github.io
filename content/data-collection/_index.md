---
weight: 50
title: "数据采集"
---

夜莺从5.1版本开始，拥抱开源社区，支持OpenTSDB的协议格式，支持[Telegraf](https://github.com/influxdata/telegraf)、Prometheus生态的exporter。具体落地实践时，建议优先选用Telegraf（all-in-one的架构，运维简单），如果Telegraf在某个采集目标上实在无法满足，再考虑Prometheus生态的各类exporter作为补充。下面是侧重说明采集的简版架构图，通过架构图可以较为清晰的了解夜莺是如何处理数据采集的。

![夜莺采集架构图](/n9e-arch-collect.png)

架构图示例中，采集和存储直接使用Prometheus，如果公司业务规模不大，采用单机版Prometheus足够了，如果业务规模较大，建议采用集群部署的[VictoriaMetrics]({{%relref "victoriametrics" %}})。Telegraf的入门安装参考[这里]({{%relref "telegraf" %}})。架构中包含Prometheus进程，所以，可以在prometheus.yml中直接配置scrape_configs来抓取各类exporter的数据，只要数据进了时序库了，夜莺就可以消费这些数据，包括告警判断、绘图展示等。

{{% notice info %}}
用了这套架构之后，未来你要是有什么东西要监控，直接百度一下“Telegraf监控xx” “Prometheus监控xx”即可，夜莺可以完全复用这些采集能力！
{{% /notice %}}