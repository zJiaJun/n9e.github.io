---
title: "使用VictoriaMetrics作为时序库"
weight: 4
---
>[VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics) 架构简单，可靠性高，在性能，成本，可扩展性方面表现出色，社区活跃，且和 [Prometheus](https://prometheus.io) 生态绑定紧密。夜莺推荐您在生产环境中使用 VictoriaMetrics 作为时序数据库。

VictoriaMetrics 提供[单机版](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html)和[集群版](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html)。如果您的每秒写入数据点数小于100万，VictoriaMetrics 官方默认推荐您使用单机版，单机版可以通过增加服务器的CPU核心数，增加内存，增加IOPS来获得线性的性能提升。且单机版易于配置和运维。

接下来的文章，介绍在夜莺中，以 VictoriaMetrics 集群版本作为时序数据库为例，完整的安装和配置过程。

![使用 VictoriaMetrics 作为夜莺的时序数据库](/fc-monitoring-vm.png )

vmstorage、vminsert、vmselect 三者组合构成 VictoriaMetrics 的集群功能，三者都可以通过启动多个实例来分担承载流量。

>**vmstorage 是数据存储模块**：
- 其数据保存在`-storageDataPath`指定的目录中，默认为`./vmstorage-data/`，vmstorage 是有状态模块，删除 storage node 会丢失约 `1/N`的历史数据（N 为集群中 vmstorage node 的节点数量）。增加 storage node，则需要同步修改 vminsert 和  vmselect 的启动参数，将新加入的storage node节点地址通过命令行参数 `-storageNode`传入给vminsert和vmselect。
- vmstorage 启动后，会监听三个端口，分别是 `-httpListenAddr :8482`、`-vminsertAddr :8400`、`-vmselectAddr :8401`。端口`8400`负责接收来自 vminsert 的写入请求，端口`8401`负责接收来自 vmselect 的数据查询请求，端口`8482`则是 vmstorage 自身提供的 http api 接口。

>**vminsert 接收来自客户端的数据写入请求，并负责转发到选定的vmstorage**：
- vminsert 接收到数据写入请求后，按照  [jump consistent hash](https://github.com/lithammer/go-jump-consistent-hash)  算法，将数据转发到选定的某个vmstorage node 上。vminsert 本身是无状态模块，可以增加或者删除一个或多个实例，而不会造成数据的损失。vminsert 模块通过启动时的参数 `-storageNode xxx,yyy,zzz` 来感知到整个 vmstorage 集群的完整 node 地址列表。
- vminsert 启动后，会监听一个端口`-httpListenAddr :8480`。该端口实现了 `prometheus remote_write`协议，因此可以接收和解析通过 `remote_write` 协议写入的数据。不过要注意，VictoriaMetrics 集群版本具有`多租户`功能，因此租户ID会以如下形式出现在 API URL 中: `http://vminsert:8480/insert/<account_id>/prometheus/api/v1/write`。
- 更多 URL Format 可以参考 [VictoriaMetrics官网](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#url-format)。

>**vmselect 接收来自客户端的数据查询请求，并负责转发到所有的 vmstorage 查询结果并合并**：
- vmselect 启动后，会监听一个端口`-httpListenAddr :8481`。该端口实现了 `prometheus remote_query`等协议，因此可以接收和解析 `remote_query` 协议的查询。不过要注意，VictoriaMetrics 集群版本具有`多租户`功能，因此租户ID会以如下形式出现在 API URL 中: `http://vminsert:8481/select/<account_id>/prometheus/api/v1/query`。
- 更多 URL Format 可以参考 [VictoriaMetrics官网](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#url-format)。

## 下载和安装 VictoriaMetrics 集群版
- 去 [vm release](https://github.com/VictoriaMetrics/VictoriaMetrics/releases) 下载编译好的二进制版本，比如我们选择下载 [v1.69.0 amd64](https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.69.0/victoria-metrics-amd64-v1.69.0-cluster.tar.gz)。
- 解压缩后得到：
```bash
$ls -l vm*-prod
-rwxr-xr-x 1 work work 10946416 Nov  8 22:03 vminsert-prod*
-rwxr-xr-x 1 work work 13000624 Nov  8 22:03 vmselect-prod*
-rwxr-xr-x 1 work work 11476736 Nov  8 22:03 vmstorage-prod*
```
- 启动三个 vmstorage 实例(可以用下面的脚本快速生成不同实例的启动命令)：
```bash
#!/bin/bash    
   
for i in `seq 0 2`; do  
    if [ $i -eq 0 ]; then
        i=""   
    fi         
   
    pp=$i      
   
    httpListenAddr=${pp}8482
    vminsertAddr=${pp}8400
    vmselectAddr=${pp}8401
    storageDataPath=./${pp}vmstorage-data
   
    prog="nohup ./vmstorage-prod -loggerTimezone Asia/Shanghai \
        -storageDataPath $storageDataPath \
        -httpListenAddr :$httpListenAddr \
        -vminsertAddr :$vminsertAddr \
        -vmselectAddr :$vmselectAddr \
        &> ${pp}vmstor.log &"
   
    echo $prog 
    (exec "$prog")                                                                                                                   
done
```

也可以输入以下命令行启动三个实例：

```bash
nohup ./vmstorage-prod -loggerTimezone Asia/Shanghai -storageDataPath ./vmstorage-data -httpListenAddr :8482 -vminsertAddr :8400 -vmselectAddr :8401 &> vmstor.log &
nohup ./vmstorage-prod -loggerTimezone Asia/Shanghai -storageDataPath ./1vmstorage-data -httpListenAddr :18482 -vminsertAddr :18400 -vmselectAddr :18401 &> 1vmstor.log &
nohup ./vmstorage-prod -loggerTimezone Asia/Shanghai -storageDataPath ./2vmstorage-data -httpListenAddr :28482 -vminsertAddr :28400 -vmselectAddr :28401 &> 2vmstor.log &
```

- 启动一个 vminsert 实例：
```bash
nohup ./vminsert-prod -httpListenAddr :8480 -storageNode=127.0.0.1:8400,127.0.0.1:18400,127.0.0.1:28400 &>vminsert.log  &
```

- 启动一个 vmselect 实例：
```bash
nohup ./vmselect-prod -httpListenAddr :8481 -storageNode=127.0.0.1:8401,127.0.0.1:18401,127.0.0.1:28401 &>vmselect.log &
```

- 查看 vmstorage，vminsert，vmselect 的 /metrics 接口:
```bash
curl http://127.0.0.1:8482/metrics 
curl http://127.0.0.1:18482/metrics 
curl http://127.0.0.1:28482/metrics 
curl http://127.0.0.1:8481/metrics 
curl http://127.0.0.1:8480/metrics 
```

- n9e-server通过remote write接口写入时序库，vm作为时序库的一个选择，其remote write接口地址为：`http://127.0.0.1:8480/insert/0/prometheus/api/v1/write` 把这个地址配置到server.conf当中即可，配置完了重启n9e-server
```toml
[[Writers]]
Name = "vm"
Url = "http://172.21.0.8:8480/insert/0/prometheus/api/v1/write"
```


- 修改您的 n9e-webapi 的配置文件 ./etc/webapi.conf 如下：
```toml
[[Clusters]]
# Prometheus cluster name
Name = "Default"
# Prometheus APIs base url
Prom = "http://127.0.0.1:8481/select/0/prometheus"
```

然后，重新启动n9e-webapi，这样夜莺就可以通过 remote query 查询到 victoriametrics 集群的数据了。

{{% notice info %}}
n9e-webapi 的安装、配置和启动，请参考 [这里]({{%relref "standalone" %}})。
{{% /notice %}}

## FAQ
- VictoriaMetrics 单机版本如何保障数据的可靠性？
	>vm 针对磁盘IO有针对性的优化，单机版可以考虑将数据的可靠性保障交给 EBS 等云盘来保证。
- VictoriaMetrics 如何评估容量？
	>参考vm的[官方文档](https://docs.victoriametrics.com/#capacity-planning)。
- VictoriaMetrics 集群版本增加或者删除vmstorage Node的时候，数据如何再平衡？
	>vm 不支持扩缩容节点时，对数据进行自动的再平衡。
- VictoriaMetrics 的数据大小如何查看？
	>可以通过 vmstorage 实例暴露的 /metrics 接口来获取到相应的统计数据，譬如：
	```bash
	$ curl http://127.0.0.1:8482/metrics |grep -i data_size
	vm_data_size_bytes{type="indexdb"} 609291
  vm_data_size_bytes{type="storage/big"} 0
  vm_data_size_bytes{type="storage/small"} 8749893
  ```
- vminsert 在将数据写入多个 vmstorage Node的时候，是按照什么规则将数据写入到不同的 node 上的？
	>采用jump consistent hash 对数据进行分片，写入到相应的storage node上。
- vmselect 在接到查询请求的时候，如何定位到请求的数据是在哪个 storage node上的？
	>vmselect 并不知道每个metrics对应的数据分布的storage node，vmselect会对所有的storage node发起查询请求，最后进行数据合并，并返回。
- VictoriaMetrics 和 M3db 的对比和选择？
	>m3db架构设计上更高级，实现难度高，m3db在时序数据功能之后，重点解决了自动扩缩容，数据自动平衡等运维难题。但是因此也更复杂，可靠性目前也更难保证。VictoriaMetrics架构设计上的tradeoff 更倾向于简单可靠，重点优化了单机版的性能，强调垂直扩展，同时和prometheus 生态做到兼容，甚至于在很多的点上做到了加强。但是 VictoriaMetrics 对于时序数据downsample，节点的自动扩缩容，数据自动再平衡等高级功能和分布式能力，是有缺失的。


## 相关资料
- [使用 Docker Compose 快速部署 VictoriaMetrics](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#start-with-docker-compose)。
- [使用 Helm Chart 快速在 Kubernetes中部署 VictoriaMetrics](https://github.com/VictoriaMetrics/helm-charts)。
- [使用 VictoriaMetrics Operator 在 Kubernetes中部署 VictoriaMetrics](https://github.com/VictoriaMetrics/operator)。
- VictoriaMetrics 集群版架构：
![VictoriaMetrics 集群版架构 ](/vm-cluster-arch.png)
