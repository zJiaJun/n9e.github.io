---
title: "接入多个Prom/VM/M3DB集群"
weight: 8
---

由于Prometheus没有集群版本，受限于容量问题，很多公司会搭建多套Prometheus，比如按照业务拆分，不同的业务使用不同的Prometheus集群，或者按照地域拆分，不同的地域使用不同的Prometheus集群。这里是以Prometheus来举例，VictoriaMetrics、M3DB都有集群版本，不过有时为了不相互干扰和地域网络问题，也会拆成多个集群。对于多集群的协同，需要在夜莺里做一些配置，架构图如下：

![夜莺读取多个时序库](/n9e-multi-cluster.png)

比如，我们有两个时序库，在北京搭建了一个Prometheus，在广州搭建了一个VictoriaMetrics，n9e-webapi会把这两个时序库作为DataSource，所以在n9e-webapi的配置文件中，要配置上这俩存储的地址，举例：

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

另外图上也可以看出，一个n9e-server对应一个时序库，所以在n9e-server的配置文件中，也需要配置对应的时序库的地址，比如北京的server，配置如下，Writers下面的Url配置的是remote write的地址，而Reader下面配置的Url是实现Prometheus原生查询接口的BaseUrl

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
Name = "prom"
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

上海区域用的是VictoriaMetrics，所以Url略有不同，配置如下：

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
Name = "vm"
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

n9e-webapi是要响应前端ajax请求的，前端会从n9e-webapi查询监控数据，n9e-webapi自身不存储监控数据，而是仅仅做了一个代理，把请求代理给后端的时序库，前端读取数据时会调用Prometheus的那些原生接口，即：`/api/v1/query` `/api/v1/query_range` `/api/v1/labels` 这种接口，所以注意啦，n9e-webapi中配置的Clusters下面的Url，都是要支持Prometheus原生接口的BaseUrl。

对于n9e-server，有两个重要作用，一个是接收监控数据，然后转发给后端多个Writer，所以，Writer可以配置多个，配置文件是toml格式，`[[Writers]]`双中括号这种就表示数组，数据写给后端存储，走的协议是Prometheus的Remote Write，所以，所有支持Remote Write的存储，都可以使用。n9e-server的另一个重要作用，是做告警判断，会周期性从mysql同步告警规则，然后根据用户配置的Promeql调用时序库的 `query` 接口，所以n9e-server的Reader下面的Url，也是要配置支持Prometheus原生接口的BaseUrl。另外注意，Writer可以配置多个，但是Reader只能配置一个。比如监控数据可以写一份到Prometheus存储近期数据用于告警判断，再写一份到OpenTSDB存储长期数据，Writer就可以配置为Prometheus和OpenTSDB这两个，而Reader只配置Prometheus即可。


