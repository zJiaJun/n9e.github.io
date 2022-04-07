---
title: "Kafka Exporter"
---



grafana-agent内置了[`kafka_exporter`](https://github.com/davidmparrott/kafka_exporter)，来采集kafka的metrics指标。

我们强烈推荐您使用独立的账号运行grafana-agent，并做好访问kafka实例的最小化授权，避免过度授权带来的安全隐患，更多可以参考[documentation](https://github.com/lightbend/kafka-lag-exporter#required-permissions-for-kafka-acl)。

# 配置并启用kafka_exporter
```yaml
kafka_exporter:
  enabled: true
  # Address array (host:port) of Kafka server
  kafka_uris: ['xxx','yyy']
```

# 采集的关键指标列表
```yaml
kafka_brokers: count of kafka_brokers (gauge)
kafka_topic_partitions: Number of partitions for this Topic (gauge)
kafka_topic_partition_current_offset: Current Offset of a Broker at Topic/Partition (gauge)
kafka_consumergroup_current_offset: Current Offset of a ConsumerGroup at Topic/Partition (gauge)
kafka_consumer_lag_millis: Current approximation of consumer lag for a ConsumerGroup at Topic/Partition (gauge)
kafka_topic_partition_under_replicated_partition: 1 if Topic/Partition is under Replicated
```

# 完整地配置项说明
```yaml
  # Enables the kafka_exporter integration, allowing the Agent to automatically
  # collect system metrics from the configured dnsmasq server address
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from the hostname
  # portion of the first kafka_uri value. If there is more than one string
  # in kafka_uri, the integration will fail to load and an instance value
  # must be manually provided.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the dnsmasq_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/dnsmasq_exporter/metrics and can be scraped by an external
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

  # Address array (host:port) of Kafka server
  [kafka_uris: <[]string>]

  # Connect using SASL/PLAIN
  [use_sasl: <bool>]

  # Only set this to false if using a non-Kafka SASL proxy
  [use_sasl_handshake: <bool> | default = true]

  # SASL user name
  [sasl_username: <string>]

  # SASL user password
  [sasl_password: <string>]

  # The SASL SCRAM SHA algorithm sha256 or sha512 as mechanism
  [sasl_mechanism: <string>]

  # Connect using TLS
  [use_tls: <bool>]

  # The optional certificate authority file for TLS client authentication
  [ca_file: <string>]

  # The optional certificate file for TLS client authentication
  [cert_file: <string>]

  # The optional key file for TLS client authentication
  [key_file: <string>]

  # If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
  [insecure_skip_verify: <bool>]

  # Kafka broker version
  [kafka_version: <string> | default = "2.0.0"]

  # if you need to use a group from zookeeper
  [use_zookeeper_lag: <bool>]

  # Address array (hosts) of zookeeper server.
  [zookeeper_uris: <[]string>]

  # Kafka cluster name
  [kafka_cluster_name: <string>]

  # Metadata refresh interval
  [metadata_refresh_interval: <duration> | default = "1m"]

  # If true, all scrapes will trigger kafka operations otherwise, they will share results. WARN: This should be disabled on large clusters
  [allow_concurrency: <bool> | default = true]

  # Maximum number of offsets to store in the interpolation table for a partition
  [max_offsets: <int> | default = 1000]

  # How frequently should the interpolation table be pruned, in seconds
  [prune_interval_seconds: <int> | default = 30]

  # Regex filter for topics to be monitored
  [topics_filter_regex: <string> | default = ".*"]

  # Regex filter for consumer groups to be monitored
  [groups_filter_regex: <string> | default = ".*"]

```
