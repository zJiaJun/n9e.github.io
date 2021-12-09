---
weight: 110
title: "内嵌Prometheus SDK做APM"
---

Google提出了应用监控的4个黄金指标，分别是：流量、延迟、错误、饱和度，其中前面3个指标都可以通过内嵌SDK的方式埋点采集。夜莺核心模块有两个，webapi主要是提供http接口给JavaScript调用，server主要是负责接收监控数据，处理告警规则，这两个模块都引入了Prometheus的Go的SDK，用此方式做App Performance监控，本节以夜莺的代码为例，讲解如何使用Prometheus的SDK。

## webapi监控

webapi模块主要统计两个内容，一个是请求的数量统计，一个是请求的延迟统计，统计时，要用不同的Label做维度区分，后面就可以通过不同的维度做多种多样的统计分析，对于HTTP请求，规划4个核心Label，分别是：service、code、path、method。service标识服务名称，要求全局唯一，便于和其他服务名称区分开，比如webapi模块，就定义为n9e-webapi，code是http返回的状态码，200就表示成功数量，其他code就是失败的，后面我们可以据此统计成功率，method是HTTP方法，GET、POST、PUT、DELETE等，比如新增用户和获取用户列表可能都是`/api/n9e/users`，从路径上无法区分，只能再加上method才能区分开。

path着重说一下，表示请求路径，比如上面提到的`/api/n9e/users`，但是，在restful实践中，url中经常会有参数，比如获取编号为1的用户的信息，接口是`/api/n9e/user/1`，获取编号为2的用户信息，接口是`/api/n9e/user/2`，如果这俩带有用户编号的url都作为Label，会造成时序库索引爆炸，而且从业务方使用角度来看，我们也不关注编号为1的用户获取请求还是编号为2的用户获取请求，而是关注整体的`GET /api/n9e/user/:id`这个接口的监控数据。所以我们在设置Label的时候，要把path设置为`/api/n9e/user/:id`，而不是那具体的带有用户编号的url路径。夜莺用的gin框架，gin框架有个FullPath方法就是获取这个信息的，比较方便。

首先，我们在webapi下面创建一个stat包，放置相关统计变量：

```golang
package stat

import (
	"time"

	"github.com/prometheus/client_golang/prometheus"
)

const Service = "n9e-webapi"

var (
	labels = []string{"service", "code", "path", "method"}

	uptime = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "uptime",
			Help: "HTTP service uptime.",
		}, []string{"service"},
	)

	RequestCounter = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "http_request_count_total",
			Help: "Total number of HTTP requests made.",
		}, labels,
	)

	RequestDuration = prometheus.NewHistogramVec(
		prometheus.HistogramOpts{
			Buckets: []float64{.01, .1, 1, 10},
			Name:    "http_request_duration_seconds",
			Help:    "HTTP request latencies in seconds.",
		}, labels,
	)
)

func Init() {
	// Register the summary and the histogram with Prometheus's default registry.
	prometheus.MustRegister(
		uptime,
		RequestCounter,
		RequestDuration,
	)

	go recordUptime()
}

// recordUptime increases service uptime per second.
func recordUptime() {
	for range time.Tick(time.Second) {
		uptime.WithLabelValues(Service).Inc()
	}
}
```

uptime变量是顺手为之，统计进程启动了多久时间，不用太关注，RequestCounter和RequestDuration，分别统计请求流量和请求延迟。Init方法是在webapi模块进程初始化的时候调用，所以进程一起，就会自动注册好。

然后我们写一个middleware，在请求进来的时候拦截一下，省的每个请求都要去统计，middleware方法的代码如下：

```golang
import (
	...
	promstat "github.com/didi/nightingale/v5/src/webapi/stat"
)

func stat() gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		c.Next()

		code := fmt.Sprintf("%d", c.Writer.Status())
		method := c.Request.Method
		labels := []string{promstat.Service, code, c.FullPath(), method}

		promstat.RequestCounter.WithLabelValues(labels...).Inc()
		promstat.RequestDuration.WithLabelValues(labels...).Observe(float64(time.Since(start).Seconds()))
	}
}
```

有了这个middleware之后，new出gin的engine的时候，就立马Use一下，代码如下：

```golang
...
r := gin.New()
r.Use(stat())
...
```

最后，监控数据要通过`/metrics`接口暴露出去，我们要暴露这个请求端点，代码如下：

```golang
import (
	...
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

func configRoute(r *gin.Engine, version string) {
	...
	r.GET("/metrics", gin.WrapH(promhttp.Handler()))
}
```

如上，每个webapi的接口的流量和成功率都可以监控到了。如果你也部署了夜莺，请求webapi的端口(默认是18000)的`/metrics`接口看看吧。

{{% notice info %}}
如果服务部署多个实例，甚至多个region，多个环境，上面的4个Label就不够用了，因为只有这4个Label不足以唯一标识一个具体的实例，此时需要env、region、instance这种Label，这些Label不需要在代码里埋点，在采集的时候一般可以附加额外的标签，通过附加标签的方式来处理即可
{{% /notice %}}


## server监控

server模块的监控，和webapi模块的监控差异较大，因为关注点不同，webapi关注的是HTTP接口的请求量和延迟，而server模块关注的是接收了多少监控指标，内部事件队列的长度，从数据库同步告警规则花费多久，同步了多少条数据等，所以，我们也需要在server的package下创建一个stat包，stat包下放置stat.go，内容如下：

```golang
package stat

import (
	"github.com/prometheus/client_golang/prometheus"
)

const (
	namespace = "n9e"
	subsystem = "server"
)

var (
	// 各个周期性任务的执行耗时
	GaugeCronDuration = prometheus.NewGaugeVec(prometheus.GaugeOpts{
		Namespace: namespace,
		Subsystem: subsystem,
		Name:      "cron_duration",
		Help:      "Cron method use duration, unit: ms.",
	}, []string{"cluster", "name"})

	// 从数据库同步数据的时候，同步的条数
	GaugeSyncNumber = prometheus.NewGaugeVec(prometheus.GaugeOpts{
		Namespace: namespace,
		Subsystem: subsystem,
		Name:      "cron_sync_number",
		Help:      "Cron sync number.",
	}, []string{"cluster", "name"})

	// 从各个接收接口接收到的监控数据总量
	CounterSampleTotal = prometheus.NewCounterVec(prometheus.CounterOpts{
		Namespace: namespace,
		Subsystem: subsystem,
		Name:      "samples_received_total",
		Help:      "Total number samples received.",
	}, []string{"cluster", "channel"})

	// 产生的告警总量
	CounterAlertsTotal = prometheus.NewCounterVec(prometheus.CounterOpts{
		Namespace: namespace,
		Subsystem: subsystem,
		Name:      "alerts_total",
		Help:      "Total number alert events.",
	}, []string{"cluster"})

	// 内存中的告警事件队列的长度
	GaugeAlertQueueSize = prometheus.NewGaugeVec(prometheus.GaugeOpts{
		Namespace: namespace,
		Subsystem: subsystem,
		Name:      "alert_queue_size",
		Help:      "The size of alert queue.",
	}, []string{"cluster"})
)

func Init() {
	// Register the summary and the histogram with Prometheus's default registry.
	prometheus.MustRegister(
		GaugeCronDuration,
		GaugeSyncNumber,
		CounterSampleTotal,
		CounterAlertsTotal,
		GaugeAlertQueueSize,
	)
}
```

定义一个监控指标，除了name之外，还可以设置namespace、subsystem，最终通过`/metrics`接口暴露的时候，可以发现：监控指标的最终名字，就是$namespace_$subsystem_$name，三者拼接在一起。webapi模块的监控代码中我们看到了counter类型和histogram类型的处理，这次我们拿GaugeAlertQueueSize举例，这是个GAUGE类型的统计数据，起一个goroutine周期性获取队列长度，然后Set到GaugeAlertQueueSize中：

```golang
package engine

import (
	"context"
	"time"

	"github.com/didi/nightingale/v5/src/server/config"
	promstat "github.com/didi/nightingale/v5/src/server/stat"
)

func Start(ctx context.Context) error {
	...
	go reportQueueSize()
	return nil
}

func reportQueueSize() {
	for {
		time.Sleep(time.Second)
		promstat.GaugeAlertQueueSize.WithLabelValues(config.C.ClusterName).Set(float64(EventQueue.Len()))
	}
}

```

另外，Init方法要在server模块初始化的时候调用，server的router.go中要暴露`/metrics`端点路径，这些就不再详述了，大家可以扒拉一下夜莺的代码看一下。

## 数据抓取

应用自身的监控数据已经通过`/metrics`接口暴露了，后续采集规则可以在prometheus.yml中配置，prometheus.yml中有个section叫：scrape_configs可以配置抓取目标，这是Prometheus范畴的知识了，大家可以参考[Prometheus官网](https://prometheus.io/)。

## 参考资料

- https://prometheus.io/docs/instrumenting/clientlibs/
- https://github.com/prometheus/client_golang/tree/master/examples
