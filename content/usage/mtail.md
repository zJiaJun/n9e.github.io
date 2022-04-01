---
weight: 305
title: "监控日志"
---

## 前言

说到日志监控，大家第一反应的可能是ELK的方案，或者Loki的方案，这两个方案都是把日志采集了发到中心，在中心存储、查看、分析，不过这个方案相对比较重量级一些，如果我们的需求只是从日志中提取一些metrics数据，比如统计一些日志中出现的Error次数之类的，则有一个更简单的方案。

这个方案在夜莺v4版本中是有的，不过后来推荐大家客户端使用Telegraf，Telegraf没有这个能力，所以v5版本的夜莺没法监控日志，怎么办呢？这里给大家介绍一个Google出品的小工具，[mtail](https://github.com/google/mtail)，mtail和夜莺v4的方案类似，就是流式读取日志，通过正则表达式匹配的方式从日志中提取metrics指标，这种方式可以利用目标机器的算力，不过如果量太大，可能会影响目标机器上的业务程序，另外一个好处是无侵入性，不需要业务埋点，如果业务程序是第三方供应商提供的，我们改不了其代码，mtail此时就非常合适了。当然了，如果业务程序是我们公司的人自己写的，那还是建议用埋点的方式采集指标，mtail只是作为一个补充吧。

## mtail简介

mtail的使用方案，参考如下两个文档（下载的话参考[Releases页面](https://github.com/google/mtail/releases)）：

- [Deploying](https://github.com/google/mtail/blob/main/docs/Deploying.md)
- [Programming Guide](https://google.github.io/mtail/Programming-Guide.html)

我们拿mtail的启动命令来举例其用法：

```bash
mtail --progs /etc/mtail --logs /var/log/syslog --logs /var/log/ntp/peerstats
```

通过 `--progs` 参数指定一个目录，这个目录里放置一堆的`*.mtail`文件，每个mtail文件就是描述的正则提取规则，通过 `--logs` 参数来指定要监控的日志目录，可以写通配符，`--logs` 可以写多次，上例中只是指定了 `--progs` 和 `--logs` ，没有其他参数，mtail启动之后会自动监听一个端口3903，在3903的`/metrics`接口暴露符合Prometheus协议的监控数据，Prometheus（或者Telegraf）就可以从 `/metrics` 接口提取监控数据。

这样看起来，原理就很清晰了，mtail启动之后，根据 `--logs` 找到相关日志文件文件，seek到文件末尾，开始流式读取，每读到一行，就根据 `--progs` 指定的那些规则文件做匹配，看是否符合某些正则，从中提取时序数据，然后通过3903的`/metrics`暴露采集到的监控指标。当然，除了Prometheus这种`/metrics`方式暴露，mtail还支持把监控数据直接推给graphite或者statsd，具体可以参考：[这里](https://github.com/google/mtail/blob/main/docs/Interoperability.md)

## mtail样例

这里我用mtail监控一下n9e-server的日志，从中提取一下各个告警规则触发的notify的数量，这个日志举例：

```
2021-12-27 10:00:30.537582 INFO engine/logger.go:19 event(cbb8d4be5efd07983c296aaa4dec5737 triggered) notify: rule_id=9 [__name__=net_response_result_code author=qin ident=10-255-0-34 port=4567 protocol=tcp server=localhost]2@1640570430
```

很明显，日志中有这么个关键字：`notify: rule_id=9`，可以用正则来匹配，统计出现的行数，ruleid也可以从中提取到，这样，我们可以把ruleid作为标签上报，于是乎，我们就可以写出这样的mtail规则了：

```bash
[root@10-255-0-34 nightingale]# cat /etc/mtail/n9e-server.mtail
counter mtail_alert_rule_notify_total by ruleid

/notify: rule_id=(?P<ruleid>\d+)/ {
    mtail_alert_rule_notify_total[$ruleid]++
}
```

然后启动也比较简单，我这里就用nohup简单来做：

```bash
nohup mtail -logtostderr --progs /etc/mtail --logs server.log &> stdout.log &
```

mtail没有指定绝对路径，是因为我把mtail的二进制直接放在了 `/usr/bin` 下面了，mtail默认会监听在3903，所以我们可以用如下命令验证：

```bash
curl -s localhost:3903/metrics
```

可以看到输出如下内容：

```
# HELP mtail_alert_rule_notify_total defined at n9e-server.mtail:1:9-37
# TYPE mtail_alert_rule_notify_total counter
mtail_alert_rule_notify_total{prog="n9e-server.mtail",ruleid="9"} 6
```

上面的输出只是挑选了部分内容，没有全部放出来哈，这就表示正常采集到了，如果n9e的server.log中当前没有打印notify相关的日志，那请求`/metrics`接口是没法得到上面的输出的，可以手工配置一条必然会触发的规则，待日志里有相关输出的时候再次请求 `/metrics` 接口，应该就有了

最后，我们使用Telegraf来采集一下 `localhost:3903/metrics` 这个地址的输出，在telegraf.conf中添加如下配置：

```toml
[[inputs.prometheus]]
urls = ["http://localhost:3903/metrics"]
```

完事重启Telegraf或者给Telegraf进程发一个SIGHUP信号：

```
kill -HUP `pidof telegraf`
```

等一会，就可以在页面上查到相关指标了，我们拿着mtail_alert_rule_notify_total这个指标去即时查询里查，会发现查不到数据，而是出现了一个mtail_alert_rule_notify_total_counter这样的指标，看起来像是Telegraf对于Prometheus协议的监控数据，自动加了后缀，无所谓了，大家注意一下就好。如果在prometheus.yaml中配置scrape_config来抓取mtail，应该不会自动加上`_counter`的后缀。

另外，mtail的配置文件如果发生变化，是需要重启mtail才能生效的，或者也是类似Telegraf那样发一个SIGHUP信号给mtail，mtail收到信号就会重新加载配置。

## mtail更多样例

mtail的github repo中有一个[examples](https://github.com/google/mtail/tree/main/examples)，里边有挺多例子，大家可以参考。我在这里再给大家举1个简单例子，比如我们要统计/var/log/messages文件中的 `Out of memory` 关键字，mtail规则应该怎么写呢？其实比上面举例的mtail_alert_rule_notify_total还要更简单：

```
counter mtail_oom_total

/Out of memory/ {
    mtail_oom_total++
}
```

## 关于时间戳

最后说一下时间戳的问题，日志中每一行一般都是有个时间戳的，夜莺v4版本在页面上配置采集规则的时候，就是要选择时间戳的，但是mtail，上面的例子中没有处理时间戳，为啥？其实mtail也可以支持从日志中提取时间戳，如果没有配置的话，就用系统当前时间，个人认为，用系统当前时间就可以了，从日志中提取时间稍微还有点麻烦，当然，系统当前时间和日志中的时间可能稍微有差别，但是不会差很多的，可以接受，examples中的mtail样例，也基本都没有给出时间戳的提取，估计这就是最佳实践。


