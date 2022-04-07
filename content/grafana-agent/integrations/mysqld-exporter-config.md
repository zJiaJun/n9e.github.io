---
title: "MySQLd Exporter"
---



grafana-agent 内置集成了 [`mysqld_exporter`](https://github.com/prometheus/mysqld_exporter)， 来收集MySQL Server的metrics指标。



目前一个grafana-agent实例，只能配置和采集一个MySQL server的metrics，因此如果您想要配置采集多个MySQL server的指标，那么需要启动多个grafana-agent实例，并使用 `relabel_configs` 机制来给不同的MySQL server的metrics数据做区分。


# 配置并启用mysqld_exporter
下面是开启了mysqld_exporter的配置文件示例:
```yaml
mysqld_exporter:
  enabled: true
  data_source_name: root@(server-a:3306)/
  relabel_configs:
  - source_labels: [__address__]
    target_label: instance
    replacement: server-a
```

为了安全起见，推荐您为grafana-agent mysqld_exporter 配置一个单独的数据库账号，并授予合适的权限，需要的权限配置详情可以参考 [MySQL Expoter 官方文档](https://github.com/prometheus/mysqld_exporter#required-grants).

# 采集的关键指标列表
```yaml
mysql_global_status_uptime: The number of seconds that the server has been up.(Gauge)
mysql_global_status_uptime_since_flush_status: The number of seconds since the most recent FLUSH STATUS statement.(Gauge)
mysql_global_status_queries: The number of statements executed by the server. This variable includes statements executed within stored programs, unlike the Questions variable. It does not count COM_PING or COM_STATISTICS commands.(Counter)
mysql_global_status_threads_connected: The number of currently open connections.(Counter)
mysql_global_status_connections: The number of connection attempts (successful or not) to the MySQL server.(Gauge)
mysql_global_status_max_used_connections: The maximum number of connections that have been in use simultaneously since the server started.(Gauge)
mysql_global_status_threads_running: The number of threads that are not sleeping.(Gauge)
mysql_global_status_questions: The number of statements executed by the server. This includes only statements sent to the server by clients and not statements executed within stored programs, unlike the Queries variable. This variable does not count COM_PING, COM_STATISTICS, COM_STMT_PREPARE, COM_STMT_CLOSE, or COM_STMT_RESET commands.(Counter)
mysql_global_status_threads_cached: The number of threads in the thread cache.(Counter)
mysql_global_status_threads_created: The number of threads created to handle connections. If Threads_created is big, you may want to increase the thread_cache_size value. The cache miss rate can be calculated as Threads_created/Connections.(Counter)
mysql_global_status_created_tmp_tables: The number of internal temporary tables created by the server while executing statements.(Counter)
mysql_global_status_created_tmp_disk_tables: The number of internal on-disk temporary tables created by the server while executing statements. You can compare the number of internal on-disk temporary tables created to the total number of internal temporary tables created by comparing Created_tmp_disk_tables and Created_tmp_tables values.(Counter)
mysql_global_status_created_tmp_files: How many temporary files mysqld has created.(Counter)
mysql_global_status_select_full_join: The number of joins that perform table scans because they do not use indexes. If this value is not 0, you should carefully check the indexes of your tables.(Counter)
mysql_global_status_select_full_range_join: The number of joins that used a range search on a reference table.(Counter)
mysql_global_status_select_range: The number of joins that used ranges on the first table. This is normally not a critical issue even if the value is quite large.(Counter)
mysql_global_status_select_range_check: The number of joins without keys that check for key usage after each row. If this is not 0, you should carefully check the indexes of your tables.(Counter)
mysql_global_status_select_scan: The number of joins that did a full scan of the first table.(Counter)
mysql_global_status_sort_rows: The number of sorted rows.(Counter)
mysql_global_status_sort_range: The number of sorts that were done using ranges.(Counter)
mysql_global_status_sort_merge_passes: The number of merge passes that the sort algorithm has had to do. If this value is large, you should consider increasing the value of the sort_buffer_size system variable.(Counter)
mysql_global_status_sort_scan: The number of sorts that were done by scanning the table.(Counter)
mysql_global_status_slow_queries: The number of queries that have taken more than long_query_time seconds. This counter increments regardless of whether the slow query log is enabled.(Counter)
mysql_global_status_aborted_connects: The number of failed attempts to connect to the MySQL server.(Counter)
mysql_global_status_aborted_clients: The number of connections that were aborted because the client died without closing the connection properly.(Counter)
mysql_global_status_table_locks_immediate: The number of times that a request for a table lock could be granted immediately. Locks Immediate rising and falling is normal activity.(Counter)
mysql_global_status_table_locks_waited: The number of times that a request for a table lock could not be granted immediately and a wait was needed. If this is high and you have performance problems, you should first optimize your queries, and then either split your table or tables or use replication.(Counter)
mysql_global_status_bytes_received: The number of bytes received from all clients.(Counter)
mysql_global_status_bytes_sent: The number of bytes sent to all clients.(Counter)
mysql_global_status_innodb_page_size: InnoDB page size (default 16KB). Many values are counted in pages; the page size enables them to be easily converted to bytes.(Gauge)
mysql_global_status_buffer_pool_pages: The number of pages in the InnoDB buffer pool.(Gauge)
mysql_global_status_commands_total: The number of times each xxx statement has been executed.(Counter)
mysql_global_status_handlers_total: Handler statistics are internal statistics on how MySQL is selecting, updating, inserting, and modifying rows, tables, and indexes. This is in fact the layer between the Storage Engine and MySQL.(Counter)
mysql_global_status_opened_files: The number of files that have been opened with my_open() (a mysys library function). Parts of the server that open files without using this function do not increment the count.(Counter)
mysql_global_status_open_tables: The number of tables that are open.(Gauge)
mysql_global_status_opened_tables: The number of tables that have been opened. If Opened_tables is big, your table_open_cache value is probably too small.(Counter)
mysql_global_status_table_open_cache_hits: The number of hits for open tables cache lookups.(Counter)
mysql_global_status_table_open_cache_misses: The number of misses for open tables cache lookups.(Counter)
mysql_global_status_table_open_cache_overflows: The number of overflows for the open tables cache.(Counter)
mysql_global_status_innodb_num_open_files: The number of files InnoDB currently holds open.(Gauge)

mysql_global_variables_thread_cache_size: How many threads the server should cache for reuse.(Gauge)
mysql_global_variables_max_connections: The maximum permitted number of simultaneous client connections.(Gauge)
mysql_global_variables_innodb_buffer_pool_size: The size in bytes of the buffer pool, the memory area where InnoDB caches table and index data. The default value is 134217728 bytes (128MB).(Gauge)
mysql_global_variables_innodb_log_buffer_size: The size in bytes of the buffer that InnoDB uses to write to the log files on disk.(Gauge)
mysql_global_variables_key_buffer_size: Index blocks for MyISAM tables are buffered and are shared by all threads.(Gauge)
mysql_global_variables_query_cache_size: The amount of memory allocated for caching query results.(Gauge)
mysql_global_variables_table_open_cache: The number of open tables for all threads.(Gauge)
mysql_global_variables_open_files_limit: The number of file descriptors available to mysqld from the operating system.(Gauge)

```

![Dashboard demo](/fc-monitor/integrations/mysqld-exporter/dashboard-n9e.png)

# mysqld-exporter-config详细配置项说明
```yaml
  # Enables the mysqld_exporter integration, allowing the Agent to collect
  # metrics from a MySQL server.
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is a truncated version of the
  # connection DSN, containing only the server and db name. (Credentials
  # are not included.)
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the mysqld_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/mysqld_exporter/metrics and can be scraped by an external
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

  # Data Source Name specifies the MySQL server to connect to. This is REQUIRED
  # but may also be specified by the MYSQLD_EXPORTER_DATA_SOURCE_NAME
  # environment variable. If neither are set, the integration will fail to
  # start.
  #
  # The format of this is specified here: https://github.com/go-sql-driver/mysql#dsn-data-source-name
  #
  # A working example value for a server with no required password
  # authentication is: "root@(localhost:3306)/"
  data_source_name: <string>

  # A list of collector names to enable on top of the default set.
  enable_collectors:
    [ - <string> ]
  # A list of collector names to disable from the default set.
  disable_collectors:
    [ - <string> ]
  # A list of collectors to run. Fully overrides the default set.
  set_collectors:
    [ - <string> ]

  # Set a lock_wait_timeout on the connection to avoid long metadata locking.
  [lock_wait_timeout: <int> | default = 2]
  # Add a low_slow_filter to avoid slow query logging of scrapes. NOT supported
  # by Oracle MySQL.
  [log_slow_filter: <bool> | default = false]

  ## Collector-specific options

  # Minimum time a thread must be in each state to be counted.
  [info_schema_processlist_min_time: <int> | default = 0]
  # Enable collecting the number of processes by user.
  [info_schema_processlist_processes_by_user: <bool> | default = true]
  # Enable collecting the number of processes by host.
  [info_schema_processlist_processes_by_host: <bool> | default = true]
  # The list of databases to collect table stats for. * for all
  [info_schema_tables_databases: <string> | default = "*"]
  # Limit the number of events statements digests by response time.
  [perf_schema_eventsstatements_limit: <int> | default = 250]
  # Limit how old the 'last_seen' events statements can be, in seconds.
  [perf_schema_eventsstatements_time_limit: <int> | default = 86400]
  # Maximum length of the normalized statement text.
  [perf_schema_eventsstatements_digtext_text_limit: <int> | default = 120]
  # Regex file_name filter for performance_schema.file_summary_by_instance
  [perf_schema_file_instances_filter: <string> | default = ".*"]
  # Remove path prefix in performance_schema.file_summary_by_instance
  [perf_schema_file_instances_remove_prefix: <string> | default = "/var/lib/mysql"]
  # Database from where to collect heartbeat data.
  [heartbeat_database: <string> | default = "heartbeat"]
  # Table from where to collect heartbeat data.
  [heartbeat_table: <string> | default = "heartbeat"]
  # Use UTC for timestamps of the current server (`pt-heartbeat` is called with `--utc`)
  [heartbeat_utc: <bool> | default = false]
  # Enable collecting user privileges from mysql.user
  [mysql_user_privileges: <bool> | default = false]
```
