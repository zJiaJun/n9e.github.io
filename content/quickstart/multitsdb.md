---
title: "接入多个Prom/VM/M3DB集群"
weight: 8
---

由于Prometheus没有集群版本，受限于容量问题，很多公司会搭建多套Prometheus，比如按照业务拆分，不同的业务使用不同的Prometheus集群，或者按照地域拆分，不同的地域使用不同的Prometheus集群。这里是以Prometheus来举例，VictoriaMetrics、M3DB都有集群版本，不过有时为了不相互干扰和地域网络问题，也会拆成多个集群。对于多集群的协同，需要在夜莺里做一些配置，回顾一下夜莺的架构图：

![夜莺架构图](/intro/arch-system.png)

图上分了 3 个 region，每个 region 一套时序库，每个 region 一套 n9e-server，n9e-server 依赖 redis，所以每个 region 一个 redis，n9e-webapi 和 mysql 放到中心，n9e-webapi 也依赖一个 redis，所以中心端放置的是 n9e-webapi、redis、mysql，如果想图省事，redis 也是可以复用的，各个 region 的 n9e-server 都连接中心的 redis 也是可以的。

为了高可用，各个 region 的 n9e-server 可以多部署几个实例组成一个集群，集群中的所有 n9e-server 的配置文件 server.conf 中的 ClusterName 要设置成一样的字符串。

假设，我们有两个时序库，在北京搭建了一个 Prometheus，在广州搭建了一个 VictoriaMetrics，n9e-webapi 会把这两个时序库作为 DataSource，所以在 n9e-webapi 的配置文件中，要配置上这俩存储的地址，举例：

```toml
[[Clusters]]
# cluster name
Name = "Prom-Beijing"
# Prometheus APIs base url
Prom = "http://10.2.3.4:9090"
# Basic auth username
BasicAuthUser = ""
# Basic auth password
BasicAuthPass = ""
# timeout settings, unit: ms
Timeout = 30000
DialTimeout = 10000
TLSHandshakeTimeout = 30000
ExpectContinueTimeout = 1000
IdleConnTimeout = 90000
# time duration, unit: ms
KeepAlive = 30000
MaxConnsPerHost = 0
MaxIdleConns = 100
MaxIdleConnsPerHost = 100

[[Clusters]]
# cluster name
Name = "VM-Guangzhou"
# Prometheus APIs base url
Prom = "http://172.21.0.8:8481/select/0/prometheus"
# Basic auth username
BasicAuthUser = ""
# Basic auth password
BasicAuthPass = ""
# timeout settings, unit: ms
Timeout = 30000
DialTimeout = 10000
TLSHandshakeTimeout = 30000
ExpectContinueTimeout = 1000
IdleConnTimeout = 90000
# time duration, unit: ms
KeepAlive = 30000
MaxConnsPerHost = 0
MaxIdleConns = 100
MaxIdleConnsPerHost = 100
```

另外图上也可以看出，一个 n9e-server 对应一个时序库，所以在 n9e-server 的配置文件中，也需要配置对应的时序库的地址，比如北京的 server，配置如下，Writers 下面的 Url 配置的是 remote write 的地址，而 Reader 下面配置的 Url 是实现Prometheus 原生查询接口的 BaseUrl。

```toml
[Reader]
# prometheus base url
Url = "http://127.0.0.1:9090"
# Basic auth username
BasicAuthUser = ""
# Basic auth password
BasicAuthPass = ""
# timeout settings, unit: ms
Timeout = 30000
DialTimeout = 10000
TLSHandshakeTimeout = 30000
ExpectContinueTimeout = 1000
IdleConnTimeout = 90000
# time duration, unit: ms
KeepAlive = 30000
MaxConnsPerHost = 0
MaxIdleConns = 100
MaxIdleConnsPerHost = 10

[[Writers]]
Url = "http://127.0.0.1:9090/api/v1/write"
# Basic auth username
BasicAuthUser = ""
# Basic auth password
BasicAuthPass = ""
# timeout settings, unit: ms
Timeout = 30000
DialTimeout = 10000
TLSHandshakeTimeout = 30000
ExpectContinueTimeout = 1000
IdleConnTimeout = 90000
# time duration, unit: ms
KeepAlive = 30000
MaxConnsPerHost = 0
MaxIdleConns = 100
MaxIdleConnsPerHost = 100
```

假设上海区域用的是 VictoriaMetrics，所以 Url 略有不同，配置如下：

```toml
[Reader]
# prometheus base url
Url = "http://127.0.0.1:8481/select/0/prometheus"
# Basic auth username
BasicAuthUser = ""
# Basic auth password
BasicAuthPass = ""
# timeout settings, unit: ms
Timeout = 30000
DialTimeout = 10000
TLSHandshakeTimeout = 30000
ExpectContinueTimeout = 1000
IdleConnTimeout = 90000
# time duration, unit: ms
KeepAlive = 30000
MaxConnsPerHost = 0
MaxIdleConns = 100
MaxIdleConnsPerHost = 10

[[Writers]]
Url = "http://127.0.0.1:8480/insert/0/prometheus/api/v1/write"
# Basic auth username
BasicAuthUser = ""
# Basic auth password
BasicAuthPass = ""
# timeout settings, unit: ms
Timeout = 30000
DialTimeout = 10000
TLSHandshakeTimeout = 30000
ExpectContinueTimeout = 1000
IdleConnTimeout = 90000
# time duration, unit: ms
KeepAlive = 30000
MaxConnsPerHost = 0
MaxIdleConns = 100
MaxIdleConnsPerHost = 100
```

n9e-webapi 是要响应前端 ajax 请求的，前端会从 n9e-webapi 查询监控数据，n9e-webapi 自身不存储监控数据，而是仅仅做了一个代理，把请求代理给后端的时序库，前端读取数据时会调用 Prometheus 的那些原生接口，即：`/api/v1/query` `/api/v1/query_range` `/api/v1/labels` 这种接口，所以注意啦，n9e-webapi 中配置的 Clusters 下面的Url，都是要支持Prometheus 原生接口的 BaseUrl。

对于 n9e-server，有两个重要作用，一个是接收监控数据，然后转发给后端多个 Writer，所以，Writer 可以配置多个，配置文件是 toml 格式，`[[Writers]]`双中括号这种就表示数组，数据写给后端存储，走的协议是 Prometheus 的 Remote Write，所以，所有支持 Remote Write 的存储，都可以使用。n9e-server 的另一个重要作用，是做告警判断，会周期性从 mysql 同步告警规则，然后根据用户配置的 PromQL 调用时序库的 `query` 接口，所以 n9e-server 的 Reader 下面的 Url，也是要配置支持 Prometheus 原生接口的 BaseUrl。另外注意，Writer 可以配置多个，但是 Reader 只能配置一个。比如监控数据可以写一份到Prometheus 存储近期数据用于告警判断，再写一份到 OpenTSDB 存储长期数据，Writer 就可以配置为 Prometheus 和 OpenTSDB 这两个，而 Reader 只配置 Prometheus 即可。


