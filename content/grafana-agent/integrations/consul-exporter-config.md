---
title: "Consul Exporter"
---

The `consul_exporter_config` block configures the `consul_exporter`
integration, which is an embedded version of
[`consul_exporter`](https://github.com/prometheus/consul_exporter). This allows
for the collection of consul metrics and exposing them as Prometheus metrics.

Full reference of options:

```yaml
  # Enables the consul_exporter integration, allowing the Agent to automatically
  # collect system metrics from the configured consul server address
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from the hostname portion
  # of the server URL.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the consul_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/consul_exporter/metrics and can be scraped by an external
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

  # Prefix from which to expose key/value pairs.
  [kv_prefix: <string> | default = ""]

  # Regex that determines which keys to expose.
  [kv_filter: <string> | default = ".*"]

  # Generate a health summary for each service instance. Needs n+1 queries to
  # collect all information.
  [generate_health_summary: <bool> | default = true]

  # HTTP API address of a Consul server or agent. Prefix with https:// to
  # connect using HTTPS.
  [server: <string> | default = "http://localhost:8500"]

  # Disable TLS host verification.
  [insecure_skip_verify: <bool> | default = false]

  # File path to a PEM-encoded certificate authority used to validate the
  # authenticity of a server certificate.
  [ca_file: <string> | default = ""]

  # File path to a PEM-encoded certificate used with the private key to verify
  # the exporter's authenticity.
  [cert_file: <string> | default = ""]

  # File path to a PEM-encoded private key used with the certificate to verify
  # the exporter's authenticity.
  [key_file: <string> | default = ""]

  # When provided, this overrides the hostname for the TLS certificate. It can
  # be used to ensure that the certificate name matches the hostname we declare.
  [server_name: <string> | default = ""]

  # Timeout on HTTP requests to the Consul API.
  [timeout: <duration> | default = "500ms"]

  # Limit the maximum number of concurrent requests to consul. 0 means no limit.
  [concurrent_request_limit: <int> | default = 0]

  # Allows any Consul server (non-leader) to service a read.
  [allow_stale: <bool> | default = true]

  # Forces the read to be fully consistent.
  [require_consistent: <bool> | default = false]
```

# 采集的指标列表
```yaml
consul_memberlist_tcp : irate(consul_memberlist_tcp{host="$consul"}[1m])
consul_memberlist_udp : irate(consul_memberlist_udp{host="$consul"}[1m])
consul_raft_apply[30s]) : delta(consul_raft_apply[30s])
consul_raft_commitTime : consul_raft_commitTime
consul_raft_leader_dispatchLog : consul_raft_leader_dispatchLog
consul_raft_leader_lastcontact : consul_raft_leader_lastcontact
consul_raft_leader_lastcontact_count : consul_raft_leader_lastcontact_count
consul_raft_replication_appendEntries_rpc : consul_raft_replication_appendEntries_rpc
consul_raft_replication_heartbeat : consul_raft_replication_heartbeat
consul_rpc_query : delta(consul_rpc_query{host="$consul"}[30s])
consul_serf_coordinate_adjustment_ms : consul_serf_coordinate_adjustment_ms{host="$consul"}
labels) : COUNT (changes(consul_memberlist_gossep_sum[1m]) > 0) BY (labels)
node_cpu : sum(irate(node_cpu{mode="idle", host="$consul"}[1m])) * 100 / count_scalar(node_cpu{mode="user", host="$consul"})
node_load1 : node_load1{host="$consul"}
node_load15 : node_load15{host="$consul"}
node_load5 : node_load5{host="$consul"}
```