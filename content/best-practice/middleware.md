---
weight: 200
title: "使用Telegraf和Exporter监控中间件"
---

对于MySQL、Redis、MongoDB、Tomcat、RabbitMQ、Ceph、Cassandra、Consul等等各类中间件、数据库，Telegraf都可以监控，Prometheus生态也提供了各类Exporter，直接用就好了，如果有优化建议，就提PR，夜莺生态再搞一套采集意义不大，能够良好的集成，与社区协同起来才是关键。

本节起了一个巨大的标题，不过并不准备事无巨细的讲解每个中间件的采集配置，Telegraf的入门在[这里]({{%relref "quickstart/telegraf" %}})，每个中间件都在[这里](https://github.com/influxdata/telegraf/tree/master/plugins/inputs)成一个目录，目录下README就是文档。如果有Telegraf解决不了的，Google一下Exporter解决方案，互相补充一下。

数据采集有了Telegraf和Exporter，问题就不大了，但是，对于某一个具体的监控对象，比如MySQL，各个指标是什么意思？应该着重关注哪些指标？哪些指标应该配置告警规则？报警的时候应该如何处理？这就是非常专业的领域知识了，欢迎大家写博客分享，并把博客链接放到本节下面 :)

- MySQL监控分享 By xxx