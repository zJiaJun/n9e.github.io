---
weight: 1
title: "使用夜莺的的几种实践姿势"
---

## 姿势一：只是把Nightingale作为WEBUI

只部署n9e-webapi和n9e-server，配合Prometheus以及各类Exporter，可以在夜莺WEBUI上查看监控数据，配置监控大盘，配置告警规则、屏蔽规则、订阅规则，查看活跃告警、历史告警。且各类配置可以按照业务组拆分归组，相互之间没有影响。

说只是作为WEBUI，其实不止，这种工作模式实际是替换掉了Prometheus内置的告警引擎+alertmanager，只需用Prometheus存时序数据，用Exporter抓取数据即可。

## 姿势二：使用Telegraf替代各类Exporter

Telegraf是个all-in-one的架构，一个二进制可以搞定机器、网络设备、中间件、数据库、Statsd等各种采集能力，相比散落的各类Exporter而言，维护成本更低一些，Telegraf支持通过OpenTSDB这个output plugin来对接夜莺。

此时就可以解锁夜莺的对象管理能力，因为Telegraf采集的数据，会流经n9e-server，n9e-server就可以从监控数据中解析出监控对象信息（标签中的ident或host，就表示监控对象标识），知道最近有哪些监控对象在上报数据，哪些监控对象已经失联了，就可以顺道做NODATA逻辑。

对象管理页面中，可以给监控对象打标签，这些标签会附到相关时序数据上，即：某个对象如果打了`bg=cloud`标签，这个对象有监控数据上报的时候，比如上报了cpu_usage_idle的监控数据，系统就可以从监控数据中解析出ident，然后根据ident找到页面上打的标签，然后把标签附到时序数据上。

过半的监控数据其实都可以关联到某个监控对象，看图的时候先找到监控对象，再查监控数据是个挺顺畅的路径，所以，这就是对象视角看图页面的设计初衷，用了Telegraf，也就可以解锁使用这个页面的功能了。

## 姿势三：使用VictoriaMetrics替换Prometheus时序存储

因为Prometheus时序存储是单机版，对于大规模场景，推荐使用VictoriaMetrics或者M3DB，因为M3DB有些复杂，一般建议使用VictoriaMetrics，VictoriaMetrics和Prometheus的查询接口完全兼容，所以夜莺也可以对接VictoriaMetrics，通过Prometheus协议的查询接口来查询VictoriaMetrics的数据。

---

所以大规模集群环境建议的组合方式是Nightingale+Telegraf+VictorMetrics，简称**NTVM**，如果是规模不大，组合方式是Nightingale+Telegraf+Prometheus，简称**NTP**，如果Telegraf的采集能力在某些场景下不足，可以用对应场景的Exporter来补齐。