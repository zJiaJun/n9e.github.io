---
title: "cAdvisor Exporter"
---



grafana-agent 内置了 [`cadvisor`](https://github.com/google/cadvisor), 可以支持采集容器的各项指标。不过 cadvisor 针对宿主机需要设置相关的权限，具体可以参考 [cAdvisor docs](https://github.com/google/cadvisor#quick-start-running-cadvisor-in-a-docker-container).

# 配置并启用cadvisor_exporter

生成grafana-agent-cfg.yaml 配置文件，其中开启cadvisor integration，配置文件具体举例如下：
```yaml
cat <<EOF > /tmp/grafana-agent-cfg.yaml
server:
  log_level: info
  http_listen_port: 12345

metrics:
  global:
    scrape_interval: 15s
    remote_write:
      - url: 'https://n9e-server:19000/prometheus/v1/write'
        basic_auth:
          username: ${FC_USERNAME}
          password: ${FC_PASSWORD}

integrations:
  cadvisor:
    enabled: true
EOF
```

# 在docker中启动 grafana-agent，同时映射相关目录
```bash
docker run \
  -v /tmp/agent:/etc/agent/data \
  -v /tmp/grafana-agent-cfg.yaml:/etc/agent/agent.yaml \
  -p 12345:12345 \
  -d \
  --privileged \
  grafana/agent:v0.23.0 \
  --config.file=/etc/agent/agent.yaml \
  --metrics.wal-directory=/etc/agent/data
```

执行 `curl http://localhost:12345/agent/api/v1/targets |jq`,输出结果中预期应该包含 integrations/cadvisor 字段，如下：
```json
{
  "status": "success",
  "data": [
    {
      "instance": "7f383657f506f53a739e2df61be58891",
      "target_group": "integrations/cadvisor",
      "endpoint": "http://127.0.0.1:12345/integrations/cadvisor/metrics",
      "state": "up",
      "labels": {
        "agent_hostname": "509c1284c59c",
        "instance": "509c1284c59c:12345",
        "job": "integrations/cadvisor"
      },
      "discovered_labels": {
        "__address__": "127.0.0.1:12345",
        "__metrics_path__": "/integrations/cadvisor/metrics",
        "__scheme__": "http",
        "__scrape_interval__": "15s",
        "__scrape_timeout__": "10s",
        "agent_hostname": "509c1284c59c",
        "job": "integrations/cadvisor"
      },
      "last_scrape": "2022-02-17T14:54:50.652267586Z",
      "scrape_duration_ms": 30,
      "scrape_error": ""
    }
  ]
}
```

执行 `curl http://localhost:12345/integrations/cadvisor/metrics`,预期输出结果下：
```plain
cadvisor_version_info{cadvisorRevision="",cadvisorVersion="",dockerVersion="",kernelVersion="5.10.76-linuxkit",osVersion="Debian GNU/Linux 10 (buster)"} 1
container_blkio_device_usage_total{device="/dev/vda",id="/",major="254",minor="0",operation="Read"} 4.6509056e+07 1645109878135
container_blkio_device_usage_total{device="/dev/vda",id="/",major="254",minor="0",operation="Write"} 3.13243648e+09 1645109878135
container_cpu_load_average_10s{id="/"} 0 1645109878135
container_cpu_system_seconds_total{id="/"} 57.789 1645109878135
container_cpu_usage_seconds_total{cpu="total",id="/"} 91.57 1645109878135
container_cpu_user_seconds_total{id="/"} 33.781 1645109878135
container_fs_inodes_free{device="/dev",id="/"} 254415 1645109878135
container_fs_inodes_free{device="/dev/shm",id="/"} 254551 1645109878135
container_fs_inodes_free{device="/dev/vda1",id="/"} 3.890602e+06 1645109878135
container_fs_inodes_free{device="/rootfs/dev/shm",id="/"} 254551 1645109878135
...
```

# 采集的指标列表
```yaml
# CPU
# 容器运行经过的cfs周期总数
container_cpu_cfs_periods_total: Number of elapsed enforcement period intervals
# 容器运行时发生节流的cfs周期总数
container_cpu_cfs_throttled_periods_total: Number of throttled period intervals
# 容器发生cpu节流的总时间
container_cpu_cfs_throttled_seconds_total: Total time duration the container has been throttled
container_cpu_load_average_10s: Value of container cpu load average over the last 10 seconds
container_cpu_system_seconds_total: Cumulative system cpu time consumed
container_cpu_usage_seconds_total: Cumulative cpu time consumed
container_cpu_user_seconds_total: Cumulative user cpu time consumed
# 容器描述中的CPU周期配置
container_spec_cpu_period: CPU period of the container
# 容器描述中的CPU quota配置
container_spec_cpu_quota: CPU quota of the container
# 容器描述中的CPU权重配置
container_spec_cpu_shares: CPU share of the container


# MEM
container_memory_cache: Total page cache memory
container_memory_failcnt: Number of memory usage hits limits
container_memory_failures_total: Cumulative count of memory allocation failures
container_memory_mapped_file: Size of memory mapped files
container_memory_max_usage_bytes: Maximum memory usage recorded
container_memory_rss: Size of RSS
container_memory_swap: Container swap usage
container_memory_usage_bytes: Current memory usage, including all memory regardless of when it was accessed
container_oom_events_total: Count of out of memory events observed for the container
container_spec_memory_limit_bytes: Memory limit for the container
container_spec_memory_reservation_limit_bytes: Memory reservation limit for the container
container_spec_memory_swap_limit_bytes: Memory swap limit for the container


# Disk
# 设备IO使用总量
container_blkio_device_usage_total: Blkio device bytes usage
container_fs_inodes_free: Number of available Inodes
container_fs_inodes_total: Total number of Inodes
container_fs_io_current: Number of I/Os currently in progress
# 容器IO总耗时
container_fs_io_time_seconds_total: Cumulative count of seconds spent doing I/Os
container_fs_io_time_weighted_seconds_total: Cumulative weighted I/O time
container_fs_limit_bytes: Number of bytes that can be consumed by the container on this filesystem
container_fs_reads_bytes_total: Cumulative count of bytes read
container_fs_read_seconds_total: Cumulative count of seconds spent reading
container_fs_reads_merged_total: Cumulative count of reads merged
container_fs_reads_total: Cumulative count of reads completed
container_fs_sector_reads_total: Cumulative count of sector reads completed
container_fs_sector_writes_total: Cumulative count of sector writes completed
container_fs_usage_bytes: Number of bytes that are consumed by the container on this filesystem
container_fs_writes_bytes_total: Cumulative count of bytes written
container_fs_write_seconds_total: Cumulative count of seconds spent writing
container_fs_writes_merged_total: Cumulative count of writes merged
container_fs_writes_total: Cumulative count of writes completed


# Network
container_network_receive_bytes_total: Cumulative count of bytes received
container_network_receive_errors_total: Cumulative count of errors encountered while receiving
container_network_receive_packets_dropped_total: Cumulative count of packets dropped while receiving
container_network_receive_packets_total: Cumulative count of packets received
container_network_transmit_bytes_total: Cumulative count of bytes transmitted
container_network_transmit_errors_total: Cumulative count of errors encountered while transmitting
container_network_transmit_packets_dropped_total: Cumulative count of packets dropped while transmitting
container_network_transmit_packets_total: Cumulative count of packets transmitted


# System
container_tasks_state: Number of tasks in given state (sleeping, running, stopped, uninterruptible, or ioawaiting)


# Others
container_last_seen: Last time a container was seen by the exporter
container_start_time_seconds: Start time of the container since unix epoch

```

# 完整地配置项说明
```yaml
  # Enables the cadvisor integration, allowing the Agent to automatically
  # collect metrics for the specified github objects.
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  [instance: <string> | default = <integrations_config.instance>]

  # Automatically collect metrics from this integration. If disabled,
  # the cadvisor integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/cadvisor/metrics and can be scraped by an external
  # process.
  [scrape_integration: <boolean> | default = <integrations_config.scrape_integrations>]

  # How often should the metrics be collected? Defaults to
  # prometheus.global.scrape_interval.
  [scrape_interval: <duration> | default = <global_config.scrape_interval>]

  # The timeout before considering the scrape a failure. Defaults to
  # prometheus.global.scrape_timeout.
  [scrape_timeout: <duration> | default = <global_config.scrape_timeout>]

  # Allows for relabeling labels on the target.
  relabel_configs:
    [- <relabel_config> ... ]

  # Relabel metrics coming from the integration, allowing to drop series
  # from the integration that you don't care about.
  metric_relabel_configs:
    [ - <relabel_config> ... ]

  # How frequent to truncate the WAL for this integration.
  [wal_truncate_frequency: <duration> | default = "60m"]

  #
  # cAdvisor-specific configuration options
  #

  # Convert container labels and environment variables into labels on prometheus metrics for each container. If false, then only metrics exported are container name, first alias, and image name.
  [store_container_labels: <boolean> | default = true]

  # List of container labels to be converted to labels on prometheus metrics for each container. store_container_labels must be set to false for this to take effect.
  allowlisted_container_labels:
    [ - <string> ]

  # List of environment variable keys matched with specified prefix that needs to be collected for containers, only support containerd and docker runtime for now.
  env_metadata_allowlist:
    [ - <string> ]

  # List of cgroup path prefix that needs to be collected even when docker_only is specified.
  raw_cgroup_prefix_allowlist:
    [ - <string> ]

  # Path to a JSON file containing configuration of perf events to measure. Empty value disabled perf events measuring.
  [perf_events_config: <boolean>]

  # resctrl mon groups updating interval. Zero value disables updating mon groups.
  [resctrl_interval: <int> | default = 0]

  # List of `metrics` to be disabled. If set, overrides the default disabled metrics.
  disabled_metrics:
    [ - <string> ]

  # List of `metrics` to be enabled. If set, overrides disabled_metrics
  enabled_metrics:
    [ - <string> ]

  # Length of time to keep data stored in memory
  [storage_duration: <duration> | default = "2m"]

  # Containerd endpoint
  [containerd: <string> | default = "/run/containerd/containerd.sock"]

  # Containerd namespace
  [containerd_namespace: <string> | default = "k8s.io"]

  # Docker endpoint
  [docker: <string> | default = "unix:///var/run/docker.sock"]

  # Use TLS to connect to docker
  [docker_tls: <boolean> | default = false]

  # Path to client certificate for TLS connection to docker
  [docker_tls_cert: <string> | default = "cert.pem"]

  # Path to private key for TLS connection to docker
  [docker_tls_key: <string> | default = "key.pem"]

  # Path to a trusted CA for TLS connection to docker
  [docker_tls_ca: <string> | default = "ca.pem"]

  # Only report docker containers in addition to root stats
  [docker_only: <boolean> | default = false]
```
