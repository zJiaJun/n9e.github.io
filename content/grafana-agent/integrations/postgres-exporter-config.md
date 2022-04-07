---
title: "Postgres Exporter"
---



grafana-agent内置了[`postgres_exporter`](https://github.com/prometheus-community/postgres_exporter)，来采集Postgres Server的metrics采集。


我们强烈推荐您分配独立的账号，供grafana-agent来连接到Postgres server，以避免过度授权带来的安全性问题，具体可以餐你考[postgres exporter官方文档](https://github.com/prometheus-community/postgres_exporter#running-as-non-superuser).

# 配置并启用cadvisor_exporter
```yaml
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
  postgres_exporter:
    enabled: true
EOF
```

# 采集的关键指标列表
```yaml
pg_locks_count : pg_locks_count{datname=~"$datname", instance=~"$instance", mode=~"$mode"} != 0
pg_postmaster_start_time_seconds : pg_postmaster_start_time_seconds{release="$release", instance="$instance"} * 1000
pg_settings_effective_cache_size_bytes : pg_settings_effective_cache_size_bytes{instance="$instance"}
pg_settings_maintenance_work_mem_bytes : pg_settings_maintenance_work_mem_bytes{instance="$instance"}
pg_settings_max_connections : pg_settings_max_connections{release="$release", instance="$instance"}
pg_settings_max_parallel_workers : pg_settings_max_parallel_workers{instance="$instance"}
pg_settings_max_wal_size_bytes : pg_settings_max_wal_size_bytes{instance="$instance"}
pg_settings_max_worker_processes : pg_settings_max_worker_processes{instance="$instance"}
pg_settings_random_page_cost : pg_settings_random_page_cost{instance="$instance"}
pg_settings_seq_page_cost : pg_settings_seq_page_cost
pg_settings_shared_buffers_bytes : pg_settings_shared_buffers_bytes{instance="$instance"}
pg_settings_work_mem_bytes : pg_settings_work_mem_bytes{instance="$instance"}
pg_stat_activity_count : pg_stat_activity_count{datname=~"$datname", instance=~"$instance", state="active"} !=0
pg_stat_activity_count : pg_stat_activity_count{datname=~"$datname", instance=~"$instance", state=~"idle|idle in transaction|idle in transaction (aborted)"}
pg_stat_bgwriter_buffers_alloc : irate(pg_stat_bgwriter_buffers_alloc{instance="$instance"}[5m])
pg_stat_bgwriter_buffers_backend : irate(pg_stat_bgwriter_buffers_backend{instance="$instance"}[5m])
pg_stat_bgwriter_buffers_backend_fsync : irate(pg_stat_bgwriter_buffers_backend_fsync{instance="$instance"}[5m])
pg_stat_bgwriter_buffers_checkpoint : irate(pg_stat_bgwriter_buffers_checkpoint{instance="$instance"}[5m])
pg_stat_bgwriter_buffers_clean : irate(pg_stat_bgwriter_buffers_clean{instance="$instance"}[5m])
pg_stat_bgwriter_checkpoint_sync_time : irate(pg_stat_bgwriter_checkpoint_sync_time{instance="$instance"}[5m])
pg_stat_bgwriter_checkpoint_write_time : irate(pg_stat_bgwriter_checkpoint_write_time{instance="$instance"}[5m])
pg_stat_database_blks_hit : pg_stat_database_blks_hit{instance="$instance", datname=~"$datname"} / (pg_stat_database_blks_read{instance="$instance", datname=~"$datname"} + pg_stat_database_blks_hit{instance="$instance", datname=~"$datname"})
pg_stat_database_conflicts : irate(pg_stat_database_conflicts{instance="$instance", datname=~"$datname"}[5m])
pg_stat_database_deadlocks : irate(pg_stat_database_deadlocks{instance="$instance", datname=~"$datname"}[5m])
pg_stat_database_temp_bytes : irate(pg_stat_database_temp_bytes{instance="$instance", datname=~"$datname"}[5m])
pg_stat_database_tup_deleted : pg_stat_database_tup_deleted{datname=~"$datname", instance=~"$instance"} != 0
pg_stat_database_tup_fetched : SUM(pg_stat_database_tup_fetched{datname=~"$datname", instance=~"$instance"})
pg_stat_database_tup_fetched : pg_stat_database_tup_fetched{datname=~"$datname", instance=~"$instance"} != 0
pg_stat_database_tup_inserted : SUM(pg_stat_database_tup_inserted{release="$release", datname=~"$datname", instance=~"$instance"})
pg_stat_database_tup_inserted : pg_stat_database_tup_inserted{datname=~"$datname", instance=~"$instance"} != 0
pg_stat_database_tup_returned : pg_stat_database_tup_returned{datname=~"$datname", instance=~"$instance"} != 0
pg_stat_database_tup_updated : SUM(pg_stat_database_tup_updated{datname=~"$datname", instance=~"$instance"})
pg_stat_database_tup_updated : pg_stat_database_tup_updated{datname=~"$datname", instance=~"$instance"} != 0
pg_stat_database_xact_commit : irate(pg_stat_database_xact_commit{instance="$instance", datname=~"$datname"}[5m])
pg_stat_database_xact_rollback : irate(pg_stat_database_xact_rollback{instance="$instance", datname=~"$datname"}[5m])
pg_static : pg_static{release="$release", instance="$instance"}
process_cpu_seconds_total : avg(rate(process_cpu_seconds_total{release="$release", instance="$instance"}[5m]) * 1000)
process_open_fds : process_open_fds{release="$release", instance="$instance"}
process_resident_memory_bytes : avg(rate(process_resident_memory_bytes{release="$release", instance="$instance"}[5m]))
process_virtual_memory_bytes : avg(rate(process_virtual_memory_bytes{release="$release", instance="$instance"}[5m]))
```
# 完整地配置项说明
```yaml
  # Enables the postgres_exporter integration, allowing the Agent to automatically
  # collect system metrics from the configured postgres server address
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from a truncated version of
  # the first DSN in data_source_names. The truncated DSN includes the hostname
  # and database name (if used) of the server, but does not include any user
  # information.
  #
  # If data_source_names contains more than one entry, the integration will fail to
  # load and a value for instance must be manually provided.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the postgres_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/postgres_exporter/metrics and can be scraped by an external
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

  # Data Source Names specifies the Postgres server(s) to connect to. This is
  # REQUIRED but may also be specified by the POSTGRES_EXPORTER_DATA_SOURCE_NAME
  # environment variable, where DSNs the environment variable are separated by
  # commas. If neither are set, the integration will fail to start.
  #
  # The format of this is specified here: https://pkg.go.dev/github.com/lib/pq#ParseURL
  #
  # A working example value for a server with a password is:
  # "postgresql://username:passwword@localhost:5432/database?sslmode=disable"
  #
  # Multiple DSNs may be provided here, allowing for scraping from multiple
  # servers.
  data_source_names:
  - <string>

  # Disables collection of metrics from pg_settings.
  [disable_settings_metrics: <boolean> | default = false]

  # Autodiscover databases to collect metrics from. If false, only collects
  # metrics from databases collected from data_source_names.
  [autodiscover_databases: <boolean> | default = false]

  # Excludes specific databases from being collected when autodiscover_databases
  # is true.
  exclude_databases:
  [ - <string> ]

  # Includes only specific databases (excluding all others) when autodiscover_databases
  # is true.
  include_databases:
  [ - <string> ]

  # Path to a YAML file containing custom queries to run. Check out
  # postgres_exporter's queries.yaml for examples of the format:
  # https://github.com/prometheus-community/postgres_exporter/blob/master/queries.yaml
  [query_path: <string> | default = ""]

  # When true, only exposes metrics supplied from query_path.
  [disable_default_metrics: <boolean> | default = false]
```
