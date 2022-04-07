---
title: "Redis Exporter"
---



grafana-agent内置了[`redis_exporter`](https://github.com/oliver006/redis_exporter)，可以采集Redis server的运行指标。

目前grafana-agent，只支持配置一个Redis server地址，对其进行数据采集。如果您希望采集多个redis实例的metrics数据，那么需要启动多个grafana-agent实例，并通过`relabel_configs`来区分来自不同redis实例的数据。

# 配置并启用redis_exporter
```yaml
redis_exporter:
  enabled: true
  redis_addr: "redis-2:6379"
  relabel_configs:
  - source_labels: [__address__]
    target_label: instance
    replacement: redis-2
```

我们强烈推荐您使用独立的账号运行grafana-agent，并做好访问redis实例的最小化授权，避免过度授权带来的安全隐患，更多可以参考[official documentation](https://github.com/oliver006/redis_exporter#authenticating-with-redis)。

# 采集的关键指标列表
```yaml
redis_active_defrag_running: When activedefrag is enabled, this indicates whether defragmentation is currently active, and the CPU percentage it intends to utilize.
redis_allocator_active_bytes: Total bytes in the allocator active pages, this includes external-fragmentation.
redis_allocator_allocated_bytes: Total bytes allocated form the allocator, including internal-fragmentation. Normally the same as used_memory.
redis_allocator_frag_bytes: Delta between allocator_active and allocator_allocated. See note about mem_fragmentation_bytes.
redis_allocator_frag_ratio: Ratio between allocator_active and allocator_allocated. This is the true (external) fragmentation metric (not mem_fragmentation_ratio).
redis_allocator_resident_bytes: Total bytes resident (RSS) in the allocator, this includes pages that can be released to the OS (by MEMORY PURGE, or just waiting).
redis_allocator_rss_bytes: Delta between allocator_resident and allocator_active.
redis_allocator_rss_ratio: Ratio between allocator_resident and allocator_active. This usually indicates pages that the allocator can and probably will soon release back to the OS.
redis_aof_current_rewrite_duration_sec: Duration of the on-going AOF rewrite operation if any.
redis_aof_enabled: Flag indicating AOF logging is activated.
redis_aof_last_bgrewrite_status: Status of the last AOF rewrite operation.
redis_aof_last_cow_size_bytes: The size in bytes of copy-on-write memory during the last AOF rewrite operation.
redis_aof_last_rewrite_duration_sec: Duration of the last AOF rewrite operation in seconds.
redis_aof_last_write_status: Status of the last write operation to the AOF.
redis_aof_rewrite_in_progress: Flag indicating a AOF rewrite operation is on-going.
redis_aof_rewrite_scheduled: Flag indicating an AOF rewrite operation will be scheduled once the on-going RDB save is complete.
redis_blocked_clients: Number of clients pending on a blocking call (BLPOP, BRPOP, BRPOPLPUSH, BLMOVE, BZPOPMIN, BZPOPMAX).
redis_client_recent_max_input_buffer_bytes: Biggest input buffer among current client connections.
redis_client_recent_max_output_buffer_bytes: Biggest output buffer among current client connections.
redis_cluster_enabled: Indicate Redis cluster is enabled.
redis_commands_duration_seconds_total: The total CPU time consumed by these commands.(Counter)
redis_commands_processed_total: Total number of commands processed by the server.(Counter)
redis_commands_total: The number of calls that reached command execution (not rejected).(Counter)
redis_config_maxclients: The value of the maxclients configuration directive. This is the upper limit for the sum of connected_clients, connected_slaves and cluster_connections.
redis_config_maxmemory: The value of the maxmemory configuration directive.
redis_connected_clients: Number of client connections (excluding connections from replicas).
redis_connected_slaves: Number of connected replicas.
redis_connections_received_total: Total number of connections accepted by the server.(Counter)
redis_cpu_sys_children_seconds_total: System CPU consumed by the background processes.(Counter)
redis_cpu_sys_seconds_total: System CPU consumed by the Redis server, which is the sum of system CPU consumed by all threads of the server process (main thread and background threads).(Counter)
redis_cpu_user_children_seconds_total: User CPU consumed by the background processes.(Counter)
redis_cpu_user_seconds_total: User CPU consumed by the Redis server, which is the sum of user CPU consumed by all threads of the server process (main thread and background threads).(Counter)
redis_db_keys: Total number of keys by DB.
redis_db_keys_expiring: Total number of expiring keys by DB
redis_defrag_hits: Number of value reallocations performed by active the defragmentation process.
redis_defrag_misses: Number of aborted value reallocations started by the active defragmentation process.
redis_defrag_key_hits: Number of keys that were actively defragmented.
redis_defrag_key_misses: Number of keys that were skipped by the active defragmentation process.
redis_evicted_keys_total: Number of evicted keys due to maxmemory limit.(Counter)
redis_expired_keys_total: Total number of key expiration events.(Counter)
redis_expired_stale_percentage: The percentage of keys probably expired.
redis_expired_time_cap_reached_total: The count of times that active expiry cycles have stopped early.
redis_exporter_last_scrape_connect_time_seconds: The duration(in seconds) to connect when scrape.
redis_exporter_last_scrape_duration_seconds: The last scrape duration.
redis_exporter_last_scrape_error: The last scrape error status.
redis_exporter_scrape_duration_seconds_count: Durations of scrapes by the exporter
redis_exporter_scrape_duration_seconds_sum: Durations of scrapes by the exporter
redis_exporter_scrapes_total: Current total redis scrapes.(Counter)
redis_instance_info: Information about the Redis instance.
redis_keyspace_hits_total: Hits total.(Counter)
redis_keyspace_misses_total: Misses total.(Counter)
redis_last_key_groups_scrape_duration_milliseconds: Duration of the last key group metrics scrape in milliseconds.
redis_last_slow_execution_duration_seconds: The amount of time needed for last slow execution, in seconds.
redis_latest_fork_seconds: The amount of time needed for last fork, in seconds.
redis_lazyfree_pending_objects: The number of objects waiting to be freed (as a result of calling UNLINK, or FLUSHDB and FLUSHALL with the ASYNC option).
redis_master_repl_offset: The server's current replication offset. 
redis_mem_clients_normal: Memory used by normal clients.(Gauge)
redis_mem_clients_slaves: Memory used by replica clients - Starting Redis 7.0, replica buffers share memory with the replication backlog, so this field can show 0 when replicas don't trigger an increase of memory usage.
redis_mem_fragmentation_bytes: Delta between used_memory_rss and used_memory. Note that when the total fragmentation bytes is low (few megabytes), a high ratio (e.g. 1.5 and above) is not an indication of an issue.
redis_mem_fragmentation_ratio: Ratio between used_memory_rss and used_memory. Note that this doesn't only includes fragmentation, but also other process overheads (see the allocator_* metrics), and also overheads like code, shared libraries, stack, etc.
redis_mem_not_counted_for_eviction_bytes: (Gauge)
redis_memory_max_bytes: Max memory limit in bytes.
redis_memory_used_bytes: Total number of bytes allocated by Redis using its allocator (either standard libc, jemalloc, or an alternative allocator such as tcmalloc)
redis_memory_used_dataset_bytes: The size in bytes of the dataset (used_memory_overhead subtracted from used_memory)
redis_memory_used_lua_bytes: Number of bytes used by the Lua engine.
redis_memory_used_overhead_bytes: The sum in bytes of all overheads that the server allocated for managing its internal data structures.
redis_memory_used_peak_bytes: Peak memory consumed by Redis (in bytes)
redis_memory_used_rss_bytes: Number of bytes that Redis allocated as seen by the operating system (a.k.a resident set size). This is the number reported by tools such as top(1) and ps(1)
redis_memory_used_scripts_bytes: Number of bytes used by cached Lua scripts
redis_memory_used_startup_bytes: Initial amount of memory consumed by Redis at startup in bytes
redis_migrate_cached_sockets_total: The number of sockets open for MIGRATE purposes
redis_net_input_bytes_total: Total input bytes(Counter)
redis_net_output_bytes_total: Total output bytes(Counter)
redis_process_id: Process ID
redis_pubsub_channels: Global number of pub/sub channels with client subscriptions
redis_pubsub_patterns: Global number of pub/sub pattern with client subscriptions
redis_rdb_bgsave_in_progress: Flag indicating a RDB save is on-going
redis_rdb_changes_since_last_save: Number of changes since the last dump
redis_rdb_current_bgsave_duration_sec: Duration of the on-going RDB save operation if any
redis_rdb_last_bgsave_duration_sec: Duration of the last RDB save operation in seconds
redis_rdb_last_bgsave_status: Status of the last RDB save operation
redis_rdb_last_cow_size_bytes: The size in bytes of copy-on-write memory during the last RDB save operation
redis_rdb_last_save_timestamp_seconds: Epoch-based timestamp of last successful RDB save
redis_rejected_connections_total: Number of connections rejected because of maxclients limit(Counter)
redis_repl_backlog_first_byte_offset: The master offset of the replication backlog buffer
redis_repl_backlog_history_bytes: Size in bytes of the data in the replication backlog buffer
redis_repl_backlog_is_active: Flag indicating replication backlog is active
redis_replica_partial_resync_accepted: The number of accepted partial resync requests(Gauge)
redis_replica_partial_resync_denied: The number of denied partial resync requests(Gauge)
redis_replica_resyncs_full: The number of full resyncs with replicas
redis_replication_backlog_bytes: Memory used by replication backlog
redis_second_repl_offset: The offset up to which replication IDs are accepted.
redis_slave_expires_tracked_keys: The number of keys tracked for expiry purposes (applicable only to writable replicas)(Gauge)
redis_slowlog_last_id: Last id of slowlog
redis_slowlog_length: Total slowlog
redis_start_time_seconds: Start time of the Redis instance since unix epoch in seconds.
redis_target_scrape_request_errors_total: Errors in requests to the exporter
redis_up: Flag indicating redis instance is up
redis_uptime_in_seconds: Number of seconds since Redis server start
```


# 完整地配置项说明
```yaml
  # Enables the redis_exporter integration, allowing the Agent to automatically
  # collect system metrics from the configured redis address
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from the hostname
  # portion of redis_addr.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the redis_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/redis_exporter/metrics and can be scraped by an external
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

  # exporter-specific configuration options

  # Address of the redis instance.
  redis_addr: <string>

  # User name to use for authentication (Redis ACL for Redis 6.0 and newer).
  [redis_user: <string>]

  # Password of the redis instance.
  [redis_password: <string>]

  # Path of a file containing a passord. If this is defined, it takes precedece
  # over redis_password.
  [redis_password_file: <string>]

  # Namespace for the metrics.
  [namespace: <string> | default = "redis"]

  # What to use for the CONFIG command.
  [config_command: <string> | default = "CONFIG"]

  # Comma separated list of key-patterns to export value and length/size, searched for with SCAN.
  [check_keys: <string>]

  # Comma separated list of LUA regex for grouping keys. When unset, no key
  # groups will be made.
  [check_key_groups: <string>]

  # Check key or key groups batch size hint for the underlying SCAN. Keeping the same name for backwards compatibility, but this applies to both key and key groups batch size configuration.
  [check_key_groups_batch_size: <int> | default = 10000]

  # The maximum number of distinct key groups with the most memory utilization
  # to present as distinct metrics per database. The leftover key groups will be
  # aggregated in the 'overflow' bucket.
  [max_distinct_key_groups: <int> | default = 100]

  # Comma separated list of single keys to export value and length/size.
  [check_single_keys: <string>]

  # Comma separated list of stream-patterns to export info about streams, groups and consumers, searched for with SCAN.
  [check_streams: <string>]

  # Comma separated list of single streams to export info about streams, groups and consumers.
  [check_single_streams: <string>]

  # Comma separated list of individual keys to export counts for.
  [count_keys: <string>]

  # Path to Lua Redis script for collecting extra metrics.
  [script_path: <string>]

  # Timeout for connection to Redis instance (in Golang duration format).
  [connection_timeout: <time.Duration> | default = "15s"]

  # Name of the client key file (including full path) if the server requires TLS client authentication.
  [tls_client_key_file: <string>]

  # Name of the client certificate file (including full path) if the server requires TLS client authentication.
  [tls_client_cert_file: <string>]

  # Name of the CA certificate file (including full path) if the server requires TLS client authentication.
  [tls_ca_cert_file: <string>]

  # Whether to set client name to redis_exporter.
  [set_client_name: <bool>]

  # Whether to scrape Tile38 specific metrics.
  [is_tile38: <bool>]

  # Whether to scrape Client List specific metrics.
  [export_client_list: <bool>]

  # Whether to include the client's port when exporting the client list. Note
  # that including this will increase the cardinality of all redis metrics.
  [export_client_port: <bool>]

  # Whether to also export go runtime metrics.
  [redis_metrics_only: <bool>]

  # Whether to ping the redis instance after connecting.
  [ping_on_connect: <bool>]

  # Whether to include system metrics like e.g. redis_total_system_memory_bytes.
  [incl_system_metrics: <bool>]

  # Whether to to skip TLS verification.
  [skip_tls_verification: <bool>]
```
