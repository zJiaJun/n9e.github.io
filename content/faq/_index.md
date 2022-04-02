---
weight: 501
title: "FAQ"
---


> 001. 多集群部署的时候MySQL、Redis分别部署几个？

公众号有个视频讲解了 [如何接入多个时序存储](https://mp.weixin.qq.com/s/ThFjha8ocjdz3sqI1Pjwjg)， 请先查看。MySQL是只需要一个，不管部署多少个n9e-webapi多少个n9e-server。redis也可以只用一个，所有的n9e-webapi和n9e-server共享，但是，如果部署多套n9e-server（一套n9e-server可能是多个n9e-server实例组成一个集群）分布在不同的地域，网络链路可能不太好，此时是建议一套n9e-server对应一个redis，n9e-webapi自己用一个redis

> 002. 架构中可以完全不用Prometheus吗？

原理上是可以的，Prometheus在夜莺的架构中，是作为时序库使用，除了Prometheus，也可以使用VictoriaMetrics、M3DB等，因为这些时序库都实现了Prometheus的Querier接口，而夜莺依赖这些接口拉取监控数据和做告警判断。

> 003. 机器只能归属一个业务组吗？

是的。

用户可能会继续追问：一台机器可能有混部的情况，同时部署多个服务，那如何来描述这种现实（毕竟，软件就是对现实的建模）呢？其实，这种关联信息在监控中是使用标签来反映的。比如某个监控数据带有这几个标签：host=cs-node001.hna service=n9e-server method=get api=/ping statuscode=200 表示：这个监控数据是n9e-server这个服务的，来自cs-node001.hna这个机器，这条监控数据描述的是/ping接口，/ping接口是个get接口，statuscode是200，这个信息非常丰富，里边既有服务名称，又有机器信息，通过这种方式我们就知道服务和机器的关联关系。

大家不要把CMDB中的机器分组需求放到夜莺中来维护，这是职责上的错配。

那为何还要提供业务组这个概念？岂不是多余了？业务组更多的是想处理权限的问题，比如我们的告警规则、屏蔽规则、订阅规则、监控大盘、自愈脚本等，只能由我们自己管理，不能被无关人员修改了或删除了。把这些规则类信息放到某个业务组中，只有这个业务组的管理员有权限管理，其他人就没有权限乱搞了。

那机器为何要归属业务组？其实也是从权限上考虑的，自愈脚本是归属业务组的，脚本的执行需要有权限控制，不能随便去机器上运行，现在的控制逻辑是脚本只能在自己的业务组下辖的机器上运行。

综上，坦白讲，我没有想到什么场景是必须让机器（即系统中的监控对象）归属到多个业务组的。关键原因是监控数据上报的时候就是自描述的，已经包含了各类信息了，不需要通过外挂的方式重新归类监控数据，而机器的分组，虽然有需求，但那是CMDB的需求，也不是监控要处理的。

更新：另外，机器（即系统中的监控对象）可以打标签，可以通过标签体现一定的分类信息，比如region=bj env=prod表示bj区的生产环境的机器。标签是K=V的格式，K可以体现维度信息，大部分归类需求都可以通过标签来解决，唯独比较麻烦的是，在同一个维度有多个值的情况，比如K=V1 K=V2这种情况，这种情况目前不支持，因为标签信息会被附到监控对象的时序数据上，时序数据的标签是map结构，所以K=V1 K=V2这种K相同的情况会产生覆盖，故而页面上监控对象打标签的时候压根就不允许这种标签。

> 004. 为何我的target_up指标一会是0一会是1，导致一会有告警一会又恢复了

这个很可能是因为调整了客户端采集上报频率导致的，默认Telegraf上报频率是10s，不会有这个问题，n9e-server每15s生成一次target_up的值，标识机器是否在正常上报数据，如果发现机器有指标在上报，target_up就置为1，否则就是0，如果把客户端采集上报频率调大，比如改成60s，那n9e-server在某些周期检查的时候，确实就发现客户端没有上报数据，毕竟上报频率太大了。此时可以调整server.conf的配置，把检测客户端是否存活的周期也放大一些，比如调整为60秒：

```toml
[NoData]
Metric = "target_up"
# unit: second
Interval = 60
```

告警规则也可以针对这种情况做一些调整，比如改成：

```
# 告警规则的持续时长设置为0，该PromQL表示130s内一直都没有监控数据上报，故而要报警。
max_over_time(target_up[130s]) == 0
```

target_up指标是0，还有可能的原因是客户端夯住了，可以重启一下客户端试试，或者是客户端所在机器当前压力过大，影响了客户端到服务端的网络通信。

> 005. 夜莺用的redis支持cluster版本或者sentinel版本吗？

不支持，就是用的单机版，改造成cluster版本或者sentinel应该也没啥太大问题，一般公有云会提供高可用的redis，一主一从那种，足够用了，自己搭建也可以，机器挂掉的概率其实很小，满足sla问题不大的

> 006. Telegraf上报的主机标识默认用的机器名，可以让它自动上报IP作为本机标识吗？

可以让它上报IP作为唯一标识，但是要想自动获取IP，做不到。个人建议是使用一些批量执行工具，比如ansible之类的，批量部署Telegraf，部署脚本里自动获取目标机器的IP，然后填充到telegraf.conf中。

> 007. 触发告警和触发恢复的逻辑是什么？

告警规则里有3个配置非常关键，promql、执行频率、持续时长，意思就是按照执行频率，每隔一段时间执行promql查询（即时查询，即调用Prometheus协议的`/api/v1/query`接口），如果查到数据就认为触发了阈值，触发了阈值是否会产生告警事件，不一定，还要看持续时长，如果持续时长为0，就相当于不用等待，触发了阈值就立马生成事件，如果持续时长大于0，那就要等待，要保证持续时长这段时间内，每次执行promql的查询都触发阈值，才认为应该生成事件。持续时长就相当于prometheus.yml中的alert rule中的for。

比如promql为 `cpu_usage_idle{cpu="cpu-total"} < 20`，执行频率是10s，持续时长是60s，就表示在60s内每10s执行一次promql查询，看promql查询是否返回内容，如果6次都返回了，说明应该生成告警事件。

恢复的逻辑：比如已经产生了告警事件，然后再次拿着promql去查询，发现没有返回内容，那就说明当前的监控数据已经不符合promql中指定的阈值条件了，就表示恢复了。当然， 如果数据丢点了，promql自然也查不到，这种情况也是会报恢复，因为夜莺确实无法区分到底是因为丢点了，还是因为没有满足阈值而导致没有返回内容。那你可能会问，把promql解析一下，去掉promql中的操作符和阈值，只拿着前面部分去查询，不就能区分到底是没数据还是因为没有满足阈值了吗？其实很难，上面举例的promql是一种简单情况，复杂的promql非常复杂，没法这么轻易的拿掉操作符和阈值。

> 008. 如何监控机器的CPU、内存、磁盘、IO、网络、进程？

这个问题，文档里有答案，Telegraf章节，有个调研笔记的链接，调研笔记中描述的很清楚了，除了机器层面的这些监控项，还有讲如何做PING监控，TCP探测等

> 009. 夜莺可以接入到Grafana来展示吗？

可以。但不是把夜莺作为DataSource，因为实在是没必要。夜莺后端可以接入多个时序库：Prometheus、M3DB、VictoriaMetrics等，这些时序库都可以直接作为Grafana的DataSource，所以，只要监控数据进了这些时序库了，Grafana就可以直接展示了

> 010. 夜莺可以配置InfluxDB的QL做告警规则吗？

不支持，当前只支持配置promql，夜莺接入时序库，有两个层面的接入，一是通过remote write，把夜莺收到的数据转发给时序库，所有支持remote write 接口的存储都可以通过这种方式接入夜莺，接收夜莺转发过来的数据；二是时序库如果开放兼容Prometheus的Querier接口，那夜莺还可以读取时序库的数据做告警判断，即Prometheus、M3DB、VictoriaMetrics、Thanos等这些存储，都完全兼容Prometheus的Querier接口，则夜莺可以配置告警规则，从这些存储中读取监控数据做告警判断。而OpenTSDB、InfluxDB等，因为不支持Prometheus的Querier接口，所以夜莺的告警规则，没法读取这些存储的数据做判断，这些存储只能作为remote write的写入端。

> 011. 从哪里获取微信机器人token？

首先，是企业微信，不是微信，在企业微信里建个群，点开群管理，添加机器人，完事查看这个机器人的信息，即可看到webhook地址，webhook地址中包含一个key的参数，key=后面就是token。拿着这个token，去夜莺里创建一个用户，比如就叫xx企微机器人，给这个用户设置一下Wecom Robot Token，配置为刚才从webhook中获取的token，后面把这个人加入告警接收组，相关的告警就会发给企业微信的这个群组。

> 012. 监控对象列表中如何添加监控对象？看到监控对象列表和对象视角都是空的

公众号的视频教程要整体看一遍，就理解整个设计了。简而言之，监控数据需要上报给n9e-server，然后由n9e-server转发给时序库，即监控数据要流经n9e-server，n9e-server才有可能从监控数据中提取ident标签的值，n9e-server会把ident标签的值作为监控对象注册到数据库中，才能在列表中看到。如果监控数据的整个传输过程没有流经n9e-server，或监控数据中没有ident标签，也就没法提取到了。

夜莺可以支持Telegraf作为客户端采集器，也可以支持grafana-agent作为采集器，这俩采集器实际都没法上报ident标签，但是Telegraf会上报host标签，grafana-agent会上报agent_hostname标签，这俩标签会被自动转换为ident字段。

当然，即使没有注册到对象视角中，其实也不耽误，没啥大不了的，只是未来看监控图表的时候，没法使用对象视角看图罢了。后面夜莺也会持续优化，增加自定义视图，到时候对象视角的看图可能就可有可无了。

> 013. v5版本如何通过api接口上报数据的时候顺便带上alias字段？

v5目前支持的上报接口是opentsdb的协议、remote write协议、datadog的协议，都不支持alias字段，而数据库中也干掉了这个字段，所以，v5没法通过api上报这个字段了，不过v5有备注字段，和自定义标签字段，可以考虑用这俩字段放置别名，这需要直接操作DB，或者通过API修改监控对象，总之，无法通过上报数据的接口来搞定

> 014. v5邮件告警的服务器配置（smtp）在哪里？

v5.4（含）之后的版本，在server.conf中，v5.4之前的版本，在etc/script/notify.py中，脚本的前面几行，很容易找到

> 015. 我在A业务组配置的告警规则，为何收到了B业务组的机器的告警？

告警规则的逻辑可以查看009号问题，每次就是用promql查询时序库，而时序库里是没有业务组的信息的，所以promql就生效到了全量的监控对象上了。推荐做法：a业务组的机器，统一打上bg=a的标签，b业务组的机器统一打上bg=b的标签，配置告警规则的时候，要带上这个标签做筛选，比如 `cpu_usage_idle{bg="a"}<15` 就表示只有bg=a这个标签的机器生效，即只有a业务组的机器生效。

近期夜莺新版本支持在告警规则配置的时候指定只在本业务组生效，一定程度上解决了这个问题，后续也会支持把业务组直接作为标签附到时序数据上，到时候就更方便了

> 016. Telegraf推送数据给n9e-server，如何加上认证机制保证安全？

Telegraf使用datadog的output吐数据给n9e-server（版本要在5.2.1以上）

```toml
[[outputs.datadog]]
timeout = "5s"
url = "http://localhost:19000/datadog/api/v1/series"
apikey = "datadog-api-key"
```

n9e-server中加上这个配置：

```toml
[BasicAuth]
apiKey = "datadog-api-key"
```

注意：这样配置之后，表示n9e-server的所有请求都走basic auth认证，原来Telegraf通过openTSDB的接口吐数据的链路就走不通了

> 017. 从低版本怎么向高版本升级？

首先去github的 [releases](https://github.com/didi/nightingale/releases) 页面 找到自己的版本，然后挨个查看每个比你高的版本，看看各个版本的release log中写了要更新哪些内容，重点关注sql相关的，比如你的版本是5.1.0，当前版本是5.3.0，那你要看 ( 5.1.0, 5.3.0 ]之间的所有版本，从低到高挨个查看，把每个版本的release log中的sql都执行一遍（如果release log中没有提sql语句，说明这几个版本没有DB表结构变更，那更简单了），最后替换n9e二进制和etc下的内容到最新版本，如果你之前etc下有些特殊配置，记得要基于最新的配置文件再修改下

> 018. telegraf的基本配置和中间件采集的配置都放到一个telegraf.conf中不好管理怎么办？

telegraf启动的时候支持两个参数`--config`和`--config-directory`，这俩参数可以一并使用，通过`--config`指定主配置文件，通过`--config-directory`指定配置目录，各种非通用的配置，都可以放到配置目录里，每个配置文件以.conf结尾，这样telegraf就可以都识别到了，比如：

```shell
./usr/bin/telegraf --config etc/telegraf/telegraf.conf --config-directory etc/telegraf.d
```

然后我做了一个mysql.conf放到telegraf.d目录下，专门用于监控mysql，内容如下，供大家参考：

```toml
[[inputs.mysql]]
servers = ["root:1234@tcp(localhost:3306)/?tls=false"]
metric_version = 2
gather_global_variables = true
interval_slow = "1m"
tagexclude = ["innodb_version"]
```

## Any Questions?

如果问题仍然没有解决，请加微信公众号：cloudmon 公众号底部菜单有各种答疑方式

