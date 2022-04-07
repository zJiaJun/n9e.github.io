---
weight: 800
title: 使用Grafana-agent采集监控数据
---

> Acknowledgement: [Grafana Agent](https://github.com/grafana/agent) is a lightweight telemetry collector based on Prometheus that only performs its scraping and remote_write functions. Agent can also collect metrics, logs, and traces for storage in Grafana Cloud and Grafana Enterprise, as well as OSS deployments of Loki (logs), and Tempo (traces), Prometheus (metrics), and Cortex (metrics). Grafana Agent also contains several integrations (embedded metrics exporters) like node-exporter, a MySQL exporter, and many more.


如果您使用和管理着Kubernetes集群以及您的应用运行在Kubernetes之上，请参考 [在K8s中使用grafana-agent](./k8s_metrics)。

# 在Windows环境安装和运行grafana-agent
1. 从[Grafana github releases](https://github.com/grafana/agent/releases)下载Windows安装文件；
2. 运行安装文件后，会对grafana-agent进行配置，并注册为Windows服务；
3. 更详细的配置文档，可以参考[Windows Guide](https://grafana.com/docs/agent/latest/getting-started/install-agent-on-windows/)；


# 在Docker中运行grafana-agent
如果您的宿主机上运行有`docker`服务，那么使用`docker`运行`grafana-agent` 是最快捷的方式。在命令行终端运行以下命令，即可在容器中启动`grafana-agent`：

## 1. 生成 grafana-agent 的配置文件
```yaml
cat <<EOF > /tmp/grafana-agent-config.yaml
server:
  log_level: info
  http_listen_port: 12345

metrics:
  global:
    scrape_interval: 15s
    scrape_timeout: 10s
  configs:
    - name: flashtest
      host_filter: false
      scrape_configs:
        - job_name: local_scrape
          static_configs:
            - targets: ['127.0.0.1:12345']
              labels:
                cluster: 'mymac'
      remote_write:
        - url: https://n9e-server:19000/prometheus/v1/write
          basic_auth:
            username: <string>
            password: <string>
EOF
```

## 2. 启动 grafana-agent 容器
```bash
docker run \
  -v /tmp/agent:/etc/agent/data \
  -v /tmp/grafana-agent-config.yaml:/etc/agent/agent.yaml \
  -p 12345:12345 \
  -d \
  grafana/agent \
  --config.file=/etc/agent/agent.yaml \
  --prometheus.wal-directory=/etc/agent/data
```

**或者您也可以从 Dockerfile 在本地 build 镜像之后再运行：**
```bash
curl -sO https://raw.githubusercontent.com/grafana/agent/main/cmd/agent/Dockerfile
docker build -t grafana/agent:latest -f ./Dockerfile
```

上述步骤中，几个需要注意的点：
- `remote_write` 和 `basic_auth` ，请根据自己的实际情况填写；
- `-p` 把容器中的端口12345映射到主机，`-d` 把容器进程放到后台运行；
- `-v /tmp/agent:/etc/agent/data` 是把宿主机的目录 `/tmp/agent` 映射到容器中 `/etc/agent/data`，用于 `grafana-agent` 持久化保存其 `WAL`(Write Ahead Log) ；
- `-v /tmp/grafana-agent-config.yaml:/etc/agent/agent.yaml` 是把 `grafana-agent` 的配置文件，放置到容器指定的位置，即 `/etc/agent/agent.yaml`

## 3. 验证 grafana-agent 是否正常工作

您可以通过直接 `curl http://localhost:12345/metrics` 来验证数据的产生是否符合预期，正常情况下会显示如下：
```plain
agent_build_info{branch="HEAD",goversion="go1.17.6",revision="36b8ca75",version="v0.23.0"} 1
agent_inflight_requests{method="GET",route="metrics"} 1
agent_metrics_active_configs 1
agent_metrics_active_instances 1
agent_tcp_connections{protocol="grpc"} 0
agent_tcp_connections{protocol="http"} 2
go_gc_duration_seconds_sum 0.0040902
go_gc_duration_seconds_count 6
go_goroutines 50
log_messages_total{level="debug"} 44
log_messages_total{level="error"} 0
log_messages_total{level="info"} 13
log_messages_total{level="warn"} 0
loki_logql_querystats_duplicates_total 0
loki_logql_querystats_ingester_sent_lines_total 0
net_conntrack_dialer_conn_attempted_total{dialer_name="local_scrape"} 1
net_conntrack_dialer_conn_attempted_total{dialer_name="remote_storage_write_client"} 1
net_conntrack_dialer_conn_closed_total{dialer_name="local_scrape"} 0
net_conntrack_dialer_conn_closed_total{dialer_name="remote_storage_write_client"} 0
net_conntrack_dialer_conn_established_total{dialer_name="local_scrape"} 1
net_conntrack_dialer_conn_established_total{dialer_name="remote_storage_write_client"} 1
process_cpu_seconds_total 11.53
process_max_fds 1.048576e+06
process_open_fds 17
process_resident_memory_bytes 9.4773248e+07
process_start_time_seconds 1.64499076013e+09
process_virtual_memory_bytes 1.356931072e+09
process_virtual_memory_max_bytes 1.8446744073709552e+19
prometheus_interner_num_strings 275
prometheus_interner_string_interner_zero_reference_releases_total 0
prometheus_sd_consulagent_rpc_duration_seconds_sum{call="services",endpoint="agent"} 0
prometheus_sd_consulagent_rpc_duration_seconds_count{call="services",endpoint="agent"} 0
prometheus_sd_consulagent_rpc_failures_total 0
prometheus_sd_dns_lookup_failures_total 0
prometheus_sd_dns_lookups_total 0
prometheus_sd_file_read_errors_total 0
prometheus_sd_file_scan_duration_seconds{quantile="0.5"} NaN
...
```

您也可以通过访问 `grafana-agent` 所暴露的 `API`，获取到 `targets` 列表来确认是否符合预期：

```json
curl http://localhost:12345/agent/api/v1/targets |jq

{
  "status": "success",
  "data": [
    {
      "instance": "7f383657f506f53a739e2df61be58891",
      "target_group": "local_scrape",
      "endpoint": "http://127.0.0.1:12345/metrics",
      "state": "up",
      "labels": {
        "cluster": "mymac",
        "instance": "127.0.0.1:12345",
        "job": "local_scrape"
      },
      "discovered_labels": {
        "__address__": "127.0.0.1:12345",
        "__metrics_path__": "/metrics",
        "__scheme__": "http",
        "__scrape_interval__": "15s",
        "__scrape_timeout__": "10s",
        "cluster": "mymac",
        "job": "local_scrape"
      },
      "last_scrape": "2022-02-16T07:18:55.6221085Z",
      "scrape_duration_ms": 6,
      "scrape_error": ""
    }
  ]
}
```

# 在本机安装运行grafana-agent

如果您的主机上没有`docker`或者您希望直接把`grafana-agent`运行在宿主机上，可以依照以下步骤：

## 1. 下载预先编译好的二进制包

**[下载地址为](https://github.com/grafana/agent/releases)**:
`https://github.com/grafana/agent/releases/download/${version}/agent-${platform}-${arch}.zip`

- 其中，`version`当前为`v0.23.0`
- 其中，可下载的`platform`和`arch`列表如下：
  - linux/amd64
  - linux/arm64
  - linux/armv7
  - linux/armv6
  - darwin/amd64
  - darwin/arm64
  - windows/amd64
  - linux/mipsle
  - freebsd/amd64

比如，我们现在的操作系统为`Linux`，架构为`Amd64`， 那么`grafana-agent`的二进制包下载命令如下：

```bash
# download the binary
curl -SOL "https://github.com/grafana/agent/releases/download/v0.23.0/agent-linux-amd64.zip"

# extract the binary
gunzip ./agent-linux-amd64.zip

# make sure it is executable
chmod a+x "agent-linux-amd64"
```

## 2. 生成 grafana-agent 的配置文件
```yaml
cat <<EOF > ./agent-cfg.yaml
server:
  log_level: info
  http_listen_port: 12345

metrics:
  global:
    scrape_interval: 15s
    scrape_timeout: 10s
    remote_write:
      - url: https://n9e-server:19000/prometheus/v1/write
        basic_auth:
          username: <string>
          password: <string>

integrations:
  agent:
    enabled: true

  node_exporter:
    enabled: true
    include_exporter_metrics: true
EOF
```

## 3. 启动 grafana-agent 

```bash
nohup ./agent-linux-amd64 \
  -config.file ./agent-cfg.yaml \
  -metrics.wal-directory ./data \
  &> grafana-agent.log &
```

## 4. 验证 grafana-agent 是否正常工作
- 您可以通过直接 `curl http://localhost:12345/metrics` 来验证数据的产生是否符合预期；
- 您也可以通过访问 `grafana-agent` 所暴露的 `API` ，获取到 `targets` 列表来确认是否符合预期，操作命令为 `curl http://localhost:12345/agent/api/v1/targets`；

至此，我们已经成功的将 grafana-agent 运行起来，并且开始收集 grafana-agent 自身的 metrics 指标。下一步，我们讲述如何通过 grafana-agent 的内嵌的各种 exporter 来采集主机、进程、MySQL等监控指标。

