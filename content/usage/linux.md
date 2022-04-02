---
weight: 80
title: "监控Linux"
---

监控 Linux 常用的有两种方式，一个是通过部署 Telegraf，一个是通过 node_exporter，关注三个层面的问题：怎么部署？配置哪些告警规则？监控大盘如何配置？

## Telegraf

- 部署方式：[使用Telegraf采集监控数据]({{%relref "quickstart/telegraf"%}}) 
- 告警规则：[https://github.com/didi/nightingale/blob/main/etc/alerts/linux_by_telegraf.json](https://github.com/didi/nightingale/blob/main/etc/alerts/linux_by_telegraf.json)
- 监控大盘：[https://github.com/didi/nightingale/blob/main/etc/dashboards/linux_by_telegraf.json](https://github.com/didi/nightingale/blob/main/etc/dashboards/linux_by_telegraf.json)

## node_exporter

- 部署方式：[https://github.com/prometheus/node_exporter](https://github.com/prometheus/node_exporter)
- 告警规则：[https://github.com/didi/nightingale/blob/main/etc/alerts/node_by_exporter.json](https://github.com/didi/nightingale/blob/main/etc/alerts/node_by_exporter.json)
- 监控大盘：[https://github.com/didi/nightingale/blob/main/etc/dashboards/node_by_exporter.json](https://github.com/didi/nightingale/blob/main/etc/dashboards/node_by_exporter.json)

