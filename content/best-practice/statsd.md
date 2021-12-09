---
weight: 112
title: "内嵌Statsd SDK做APM"
---

## [StatsD](https://github.com/statsd/statsd)简介

在[内嵌Prometheus SDK做APM]({{%relref "promapm" %}})一节中，我们介绍了业务进程内嵌Prometheus的SDK做埋点，这种方式，会把监控数据聚合计算的逻辑放在业务进程中，比如一些平均值、分位值的计算，可能会对业务进程造成影响，本节要介绍的StatsD的方式，理念是把指标聚合计算的逻辑挪到业务进程之外，业务进程调用埋点函数的时候，通过UDP推送给StatsD，即使StatsD挂了，也不会对业务进程造成影响。

StatsD的简介，网上一搜一大把，请大家自行Google，这里就不重复描述了。核心要理解一下StatsD的设计理念、协议、支持的各个语言的SDK（在附录里有）即可，下面直接拿一个小例子讲解如何利用Telegraf支持StatsD协议的数据，数据只要进了Telegraf了，就意味着可以推到夜莺了，夜莺就相当于支持了StatsD的埋点监控采集能力。

## Telegraf启用StatsD

在Telegraf的配置文件中搜索inputs.statsd就能看到对应的section：

```toml
[[inputs.statsd]]
protocol = "udp"
service_address = ":8125"
percentiles = [50.0, 90.0, 99.0, 99.9, 99.95, 100.0]
metric_separator = "_"
```

启用如上配置，percentiles略微有点多，可以配置的少一点，比如`percentiles = [50.0, 90.0, 99.0, 100.0]`，这样整体计算存储压力也会小一些。重启Telegraf，Telegraf就会在8125端口监听udp协议，接收业务埋点数据的上报。即，Telegraf实现了[StatsD](https://github.com/statsd/statsd)的协议，可以作为StatsD的Server使用。

## 在业务程序中埋点

附录里罗列了一些客户端SDK，这里笔者使用Go语言的一个SDK来测试，实现了一个很小的web程序，代码如下：

```golang
package main

import (
	"fmt"
	"math/rand"
	"net/http"
	"time"

	"github.com/smira/go-statsd"
)

var client *statsd.Client

func homeHandler(w http.ResponseWriter, r *http.Request) {
	start := time.Now()

	// random sleep
	num := rand.Int31n(100)
	time.Sleep(time.Duration(num) * time.Millisecond)
	fmt.Fprintf(w, "duration: %d", num)

	client.Incr("requests.counter,page=home", 1)
	client.PrecisionTiming("requests.latency,page=home", time.Since(start))
}

func main() {
	// init client
	client = statsd.NewClient("localhost:8125",
		statsd.TagStyle(statsd.TagFormatInfluxDB),
		statsd.MaxPacketSize(1400),
		statsd.MetricPrefix("http."),
		statsd.DefaultTags(statsd.StringTag("service", "n9e-webapi"), statsd.StringTag("region", "bj")),
	)

	defer client.Close()

	http.HandleFunc("/", homeHandler)
	http.ListenAndServe(":8000", nil)
}
```

这个web服务只有一个根路径，逻辑也很简单，就是随机sleep几十个毫秒当做业务处理时间。整体逻辑是这样的：首先，我们要通过statsd.NewClient初始化一个statsd的客户端，参数中指定了StatsD的Server地址（在本例中就是Telegraf的8125），指定了所有监控指标的前缀是`http.`，还指定了两个全局Tag，一个是`service=n9e-webapi`，另一个是`region=bj`，通过TagStyle指定了要发送的是InfluxDB样式（因为数据是发给Telegraf的，Telegraf是InfluxDB生态的）的标签。然后，在请求的具体处理逻辑里上报了两个监控指标，一个是`requests.counter`，另一个是`requests.latency`，并且，为这俩指标指定了一个指标级别的标签`page=home`，整体看起来还是比较简单的。

## 测试方法

上面的Go程序编译一下，启动，会作为一个web server监听在8000端口，然后周期性请求这个web server的地址做测试，这个web server接收到请求之后，就调用statsd的sdk，statsd的sdk的核心逻辑就是把数据发给Telegraf的8125，然后就是Telegraf处理聚合逻辑，聚合之后的数据每10s（默认flush频率）发给夜莺。

在页面上，应该可以看到http_requests_latency和http_requests_counter打头的相关指标，比如http_requests_latency_mean这个指标，会看到这个指标有如下几个标签：

- ident: VM-0-4-centos 这个标签其实是Telegraf原始的host标签，夜莺的规范里叫ident，所以做了一下rename
- metric_type: timing 这个显然是把statsd的数据类型也做为标签了，其他数据类型还有gauge、counter、set等
- page: home 这是我们代码里附到监控指标后面的标签，Telegraf自动帮解析出来了
- service: n9e-webapi NewClient时候附加的全局默认标签
- region: bj NewClient时候附加的全局默认标签


## 附录资料

- [Measure Anything, Measure Everything](https://codeascraft.com/2011/02/15/measure-anything-measure-everything/)
- [Statsd支持的的客户端SDK列表](https://github.com/statsd/statsd/wiki#client-implementations)

