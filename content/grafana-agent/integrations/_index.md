---
title: "grafana-agent采集常用exporter"
weight: 1
---

- grafana-agent 的 metrics采集，完全兼容 prometheus exporter 生态，一些常见的 exporter，会在 grafana-agent 中内嵌实现（列表如下）;
- 对于未嵌入到 grafana-agent中的 exporter，则可以在 grafana-agent 中配置 scrape_configs 来完成抓取和收集，请参考[抓取第三方exporter](#通过grafana-agent抓取第三方exporter并收集);


# grafana-agent 内置实现的 exporter 列表
- [node-exporter](/fc-monitor/agent/integrations/node-exporter-config)
- [mysqld-exporter](/fc-monitor/agent/integrations/mysqld-exporter-config)
- [process-exporter](/fc-monitor/agent/integrations/process-exporter-config)
- [cadvisor](/fc-monitor/agent/integrations/cadvisor-config)
- [windows-exporter](/fc-monitor/agent/integrations/windows-exporter-config)
- [postgres-exporter](/fc-monitor/agent/integrations/postgres-exporter-config)
- [mongodb-exporter](/fc-monitor/agent/integrations/mongodb-exporter-config)
- [redis-exporter](/fc-monitor/agent/integrations/redis-exporter-config)
- [memcached-exporter](/fc-monitor/agent/integrations/memcached-exporter-config)
- [kafka-exporter](/fc-monitor/agent/integrations/kafka-exporter-config)
- [elasticsearch-exporter](/fc-monitor/agent/integrations/elasticsearch-exporter-config)
- [consul-exporter](/fc-monitor/agent/integrations/consul-exporter-config)
- [dnsmasq-exporter](/fc-monitor/agent/integrations/dnsmasq-exporter-config)

# 内置exporter的配置项说明
{{< highlight yaml "linenos=table" >}}
# grafana-agent 本身的配置
server:
  log_level: info
  http_listen_port: 12345

# grafana-agent 抓取 metrics 的相关配置（类似于prometheus的scrape_configs）
metrics:
  global:
    scrape_interval: 15s
    scrape_timeout: 10s
    remote_write:
      - url: https://flashc.at/api/v1/prom/write
        basic_auth:
          username: <string>
          password: <string>

# grafana-agent integration 相关的配置
integrations:
  ## grafana-agent self-integration
  ## grafana-agent 本身的metrics 采集，这也是一个内嵌的 integration，可以选择启用或者关闭。
  agent:
    ### 是否开启针对grafana-agent 自身的integration，允许grafana-agent自动采集和发送其自身的metrics
    [enabled: <boolean> | default = false]

    # Sets an explicit value for the instance label when the integration is
    # self-scraped. Overrides inferred values.
    #
    # The default value for this integration is inferred from the agent hostname
    # and HTTP listen port, delimited by a colon.
    [instance: <string>]

    # Automatically collect metrics from this integration. If disabled,
    # the agent integration will be run but not scraped and thus not
    # remote_written. Metrics for the integration will be exposed at
    # /integrations/agent/metrics and can be scraped by an external process.
    ### 这个配置项如果设置为false，那么 /integrations/agent/metrics 的数据并不会被自动抓取和发送
    ### 但是，该接口 /integrations/agent/metrics 的数据仍然支持被外部的抓取进程所抓取 
    [scrape_integration: <boolean> | default = <integrations_config.scrape_integrations>]

    # How often should the metrics be collected? Defaults to
    # prometheus.global.scrape_interval.
    [scrape_interval: <duration> | default = <global_config.scrape_interval>]

    # The timeout before considering the scrape a failure. Defaults to
    # prometheus.global.scrape_timeout.
    [scrape_timeout: <duration> | default = <global_config.scrape_timeout>]

    # How frequent to truncate the WAL for this integration.
    [wal_truncate_frequency: <duration> | default = "60m"]

    # Allows for relabeling labels on the target.
    relabel_configs:
      [- <relabel_config> ... ]

    # Relabel metrics coming from the integration, allowing to drop series
    # from the integration that you don't care about.
    metric_relabel_configs:
      [ - <relabel_config> ... ]

  # Client TLS Configuration
  # Client Cert/Key Values need to be defined if the server is requesting a certificate
  #  (Client Auth Type = RequireAndVerifyClientCert || RequireAnyClientCert).
  http_tls_config: <tls_config>

  ## 控制内嵌的 node_exporter 工作逻辑 
  node_exporter: <node_exporter_config>

  ## 控制内嵌的 process_exporter 工作逻辑
  process_exporter: <process_exporter_config>

  ## 控制内嵌的 mysqld_exporter 工作逻辑
  mysqld_exporter: <mysqld_exporter_config>

  ## 控制内嵌的 redis_exporter 工作逻辑
  redis_exporter: <redis_exporter_config>

  ## 控制内嵌的 dnsmasq_exporter 工作逻辑
  dnsmasq_exporter: <dnsmasq_exporter_config>

  ## 控制内嵌的 elasticsearch_exporter 工作逻辑
  elasticsearch_expoter: <elasticsearch_expoter_config>

  # Controls the memcached_exporter integration
  memcached_exporter: <memcached_exporter_config>

  ## 控制内嵌的 postgres_exporter 工作逻辑
  postgres_exporter: <postgres_exporter_config>

  ## 控制内嵌的 statsd_exporter 工作逻辑
  statsd_exporter: <statsd_exporter_config>

  ## 控制内嵌的 consul_exporter 工作逻辑
  consul_exporter: <consul_exporter_config>

  ## 控制内嵌的 windows_exporter 工作逻辑
  windows_exporter: <windows_exporter_config>

  ## 控制内嵌的 kafka_exporter 工作逻辑
  kafka_exporter: <kafka_exporter_config>

  ## 控制内嵌的 mongodb_exporter 工作逻辑 
  mongodb_exporter: <mongodb_exporter_config>
  
  ## 控制内嵌的 github_exporter 工作逻辑
  github_exporter: <github_exporter_config>

  # Automatically collect metrics from enabled integrations. If disabled,
  # integrations will be run but not scraped and thus not remote_written. Metrics
  # for integrations will be exposed at /integrations/<integration_key>/metrics
  # and can be scraped by an external process.
  ## 如果设置为false，相关的exporter metrics接口仍会被暴露出来，但是grafana-agent不会去主动抓取和发送
  [scrape_integrations: <boolean> | default = true]

  # Extra labels to add to all samples coming from integrations.
  labels:
    { <string>: <string> }

  # The period to wait before restarting an integration that exits with an
  # error.
  [integration_restart_backoff: <duration> | default = "5s"]

  # A list of remote_write targets. Defaults to global_config.remote_write.
  # If provided, overrides the global defaults.
  prometheus_remote_write:
    - [<remote_write>]
{{< /highlight >}}

# 通过grafana-agent抓取第三方exporter并收集

如文章开头所述，对于未嵌入到grafana-agent中的exporter，则可以在grafana-agent中配置`scrape_configs`来完成抓取和收集，其配置形式完全等同于 [prometheus scrape_configs](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config)。

grafana-agent中关于自定义配置scrape_configs的详细说明如下：

{{< highlight yaml "linenos=table" >}}
# scrape_configs like prometheus style
configs:
  scrape_timeout: 10s

  # 比如，我们可以配置抓取 grafana-agent 本身的 metrics ： http://127.0.0.1:12345/metrics
  - name: grafana-agent
    host_filter: false
    scrape_configs:
      - job_name: grafana-agent
        static_configs:
          - targets: ['127.0.0.1:12345']
    remote_write:
      - url: http://localhost:9090/api/v1/write

  # 再比如,我们也可以配置抓取您的应用程序暴露的metrics接口： http://helloworld.app:8088/metrics
  - name: outside-exporters
    host_filter: false
    scrape_configs:
      - job_name: prometheus
        static_configs:
          - targets: ['127.0.0.1:9090']
            labels:
              cluster: 'fc-monitoring'
    remote_write:
      - url: https://flashc.at/api/v1/prom/write
        basic_auth:
          username: <string>
          password: <string>
{{< /highlight >}}
