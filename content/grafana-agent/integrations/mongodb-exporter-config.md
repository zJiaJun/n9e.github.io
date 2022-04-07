---
title: "Mongodb Exporter"
---



grafana-agent内置了[`mongodb_exporter`](https://github.com/percona/mongodb_exporter)，可以采集mongodb的metrics。 

该mongodb_exporter，不支持同时配置多个mongodb node，目前只支持配置一个mongodb node，对其进行数据采集。此外您需要通过`relabel_configs`对label做自定义处理，一个是`service_name`，用来标识mongodb node（例如ReplicaSet1-Node1）；另一个是`mongodb_cluster`，标识该mongodb cluster（比如prod-cluster）

一个relabel_configs的例子：
```yaml
relabel_configs:
    - source_labels: [__address__]
      target_label: service_name
      replacement: 'replicaset1-node1'
    - source_labels: [__address__]
      target_label: mongodb_cluster
      replacement: 'prod-cluster'
```

强烈推荐您为grafana-agent设置一个单独的账号来访问您的mongodb，以避免过度授权带来的安全隐患，具体可以参考[official documentation](https://github.com/percona/mongodb_exporter#permissions)。

# 配置并启用mongodb_exporter
```
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

integrations:
  mongodb_exporter:
    enabled: true
```

# 采集的关键指标列表
```yaml
# Whether MongoDB is up.
# 实例是否存活
# Gauge
mongodb_up

# The number of seconds that the current MongoDB process has been active
# 实例启动累计时间（秒）
# Counter
mongodb_instance_uptime_seconds

# The amount of memory, in mebibyte (MiB), currently used by the database process
# 内存占用（MiB）
# Gauge
# mongodb_memory

# The total combined latency in microseconds
# 累计操作耗时（毫秒）
mongodb_mongod_op_latencies_latency_total

# The total number of operations performed since startup
# 累计操作次数
# Counter
mongodb_mongod_op_latencies_ops_total

# The total number of operations received since the mongod instance last started 
# 累计接收的操作请求次数（即使操作不成功也会增加）
# Counter
mongodb_op_counters_total

# The number of incoming connections from clients to the database server. This number includes the current shell session
# 连接数
# Gauge
# mongodb_connections

# The number of open cursors
# 打开游标数量
# Gauge
mongodb_mongod_metrics_cursor_open

# The total number of document access and modification patterns
# 累计文档操作次数
# Counter
mongodb_mongod_metrics_document_total

# The total number of operations queued waiting for the lock
# 当前排队等待获取锁的操作个数
# Gauge
mongodb_mongod_global_lock_current_queue

# The total number of (index or document) items scanned during queries and query-plan evaluation
# 查询和查询计划评估过程扫描的（索引或文档）条目总数
# Counter 
mongodb_mongod_metrics_query_executor_total

# The number of assertions raised since the MongoDB process started
# 累计断言错误次数
# Counter
mongodb_asserts_total

# The total number of getLastError operations with a specified write concern (i.e. w) that wait for one or more members of a replica set to acknowledge the write operation (i.e. a w value greater than 1.)
# 累计getLastError操作数量
# Counter
mongodb_mongod_metrics_get_last_error_wtime_num_total

# The number of times that write concern operations have timed out as a result of the wtimeout threshold to getLastError. This number increments for both default and non-default write concern specifications.
# 累计getLastError超时操作数量
# Counter
mongodb_mongod_metrics_get_last_error_wtimeouts_total

# Size in byte of the data currently in cache
# 当前缓存数据大小（byte）
# Gauge
mongodb_mongod_wiredtiger_cache_bytes

# Size in byte of the data read into or write from cache 
# 写入或读取的缓存数据大小（byte）
# Counter
mongodb_mongod_wiredtiger_cache_bytes_total

# Number of pages currently held in the cache
# 当前缓存页数量
# Gauge
mongodb_mongod_wiredtiger_cache_pages

# The total number of pages (modified or unmodified) evicted
# 累计缓存移除页数量
# Counter
mongodb_mongod_wiredtiger_cache_evicted_total

# The total number of page faults
# 累计缺页中断次数
# Counter
mongodb_extra_info_page_faults_total

# The total number of bytes that the server has sent over network connections initiated by clients or other mongod or mongos instances.
# 累计发送网络流量（byte）
# Counter
mongodb_ss_network_bytesOut

# The total number of bytes that the server has received over network connections initiated by clients or other mongod or mongos instances
# 累计接收网络流量（byte）
# Counter
mongodb_ss_network_bytesIn

# The timestamp the node was elected as replica leader
# 副本集选主时间
# Gauge
mongodb_mongod_replset_member_election_date

# The replication lag that this member has with the primary
# 副本集成员主从延迟（秒）
# Gauge
mongodb_mongod_replset_member_replication_lag

```

# 完整地配置项说明
```yaml
  # Enables the mongodb_exporter integration
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from the hostname
  # portion of the mongodb_uri field.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the mongodb_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/mongodb_exporter/metrics and can be scraped by an external
  # process.
  [scrape_integration: <boolean> | default = <integrations_config.scrape_integrations>]

  # How often should the metrics be collected? Defaults to
  # metrics.global.scrape_interval.
  [scrape_interval: <duration> | default = <global_config.scrape_interval>]

  # The timeout before considering the scrape a failure. Defaults to
  # metrics.global.scrape_timeout.
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

  # MongoDB node connection URL, which must be in the [`Standard Connection String Format`](https://docs.mongodb.com/manual/reference/connection-string/#std-label-connections-standard-connection-string-format)
  [mongodb_uri: <string>]
```
