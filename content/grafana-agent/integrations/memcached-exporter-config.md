---
title: "Memcached Exporter"
---



grafana-agent内置了[`memcached_exporter`](https://github.com/prometheus/memcached_exporter)，来采集memcached的运行指标。

当前grafana-agent只支持配置一个memcached的地址来采集其metrics数据。如果您需要采集多个memcached的metrics指标，那么需要启动多个grafana-agent实例，并通过`relabel_configs`来区分来自不同memcached server的metrics。

# 配置并启用memcached_exporter
```yaml
memcached_exporter:
  enabled: true
  memcached_address: memcached-a:53
  relabel_configs:
  - source_labels: [__address__]
    target_label: instance
    replacement: memcached-a
```

# 采集的关键指标列表
```yaml
memcached_commands_total : sum (memcached_commands_total{instance=~"$node", command="set"}) / sum (memcached_commands_total{instance=~"$node", command="get"})
memcached_commands_total : sum (memcached_commands_total{instance=~"$node", status="miss"})  / sum (memcached_commands_total{instance=~"$node"}) 
memcached_commands_total : sum (memcached_commands_total{instance=~"$node"}) by (command)
memcached_current_bytes : sum(memcached_current_bytes{instance=~"$node"}) / sum(memcached_limit_bytes{instance=~"$node"})
memcached_current_connections : sum (memcached_current_connections{instance=~"$node"}) by (instance)
memcached_current_items : sum (memcached_current_items{instance=~"$node"})
memcached_items_evicted_total : sum(memcached_items_evicted_total{instance=~"$node"})
memcached_items_reclaimed_total : sum(memcached_items_reclaimed_total{instance=~"$node"})
memcached_read_bytes_total : sum(irate(memcached_read_bytes_total{instance=~"$node"}[5m]))
memcached_written_bytes_total : irate(memcached_written_bytes_total{instance=~"$node"}[10m])
```

# 完整地配置项说明
```yaml
  # Enables the memcached_exporter integration, allowing the Agent to automatically
  # collect system metrics from the configured memcached server address
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from
  # memcached_address.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the memcached_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/memcached_exporter/metrics and can be scraped by an external
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

  # Monitor the exporter itself and include those metrics in the results.
  [include_exporter_metrics: <bool> | default = false]

  #
  # Exporter-specific configuration options
  #

  # Address of the memcached server in host:port form.
  [memcached_address: <string> | default = "localhost:53"]

  # Timeout for connecting to memcached.
  [timeout: <duration> | default = "1s"]
```