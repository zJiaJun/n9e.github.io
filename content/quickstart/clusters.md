---
title: "生产环境部署高可用集群版"
weight: 9
---

对于规模相对较小的公司，比如几百台机器这个体量，个人认为[单机版]({{%relref "standalone" %}})足够用了，使用云主机部署，性能不足可以直接升配，存储使用云存储保证，硬件故障云平台也会自动把虚拟机热迁移走，非常省心。那如果咱们体量确实比较大，或者没有云主机这种基础设施，这里会讲解集群版的部署方式。

### 时序库

时序库的集群部署，就看时序库自身的机制了，这里不展开，如果是使用 Prometheus，Prometheus 没有集群版，VictoriaMetrics、M3DB、Thanos 等都是有集群版的，请参考他们的文档

### MySQL

MySQL 一般部署成主从，这个请你们的 DBA 来搞吧，或者直接使用云上 RDS，这里就不展开了

### Redis

夜莺当前 n9e-webapi 和 n9e-server 都依赖 redis，redis 一般有三种模式，单机模式、集群模式、哨兵模式，夜莺当前只支持单机模式。具体使用几个 redis，要看大家的环境情况，如果图省事，全局就使用一个 redis 也是可以的。也可以把 n9e-webapi 用的 redis 和 n9e-server 用的 redis 分开，可以参考文档：[接入多个 Prom/VM/M3DB 集群]({{%relref "multitsdb" %}})

### n9e-webapi

这个模块是放中心的，可以部署多个实例，前面统一放置 nginx 或者 lvs，某个 n9e-webapi 实例如果挂了，nginx、lvs 都可以自动摘除，保证了高可用。

### n9e-server

n9e-server 是随着时序库走的，每个时序库对应一套 n9e-server，一套 n9e-server 可以只有一个 n9e-server 实例，也可以部署多个保证高可用和性能。同一套 n9e-server 内部的多个实例，其配置文件 server.conf 中的 ClusterName 字段，要配置成一样的字符串。

