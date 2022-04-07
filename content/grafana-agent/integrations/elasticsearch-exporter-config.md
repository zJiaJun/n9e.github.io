---
title: "Elasticsearch Exporter"
---



grafana-agent内置了[`elasticsearch_exporter`](https://github.com/prometheus-community/elasticsearch_exporter)，可以采集Elasticsearch的运行指标。

目前grafana-agent不支持配置多个elasticsearch的地址，只能配置一个ElasticSearch地址对其进行metrics的采集。

我们强烈推荐您使用独立的账号运行grafana-agent，并做好访问elasticsearch实例的最小化授权，避免过度授权带来的安全隐患，更多可以参考[official documentation](https://github.com/prometheus-community/elasticsearch_exporter#elasticsearch-7x-security-privileges)。

# 配置并启用elasticsearch_exporter
```yaml
elasticsearch_exporter:
  enabled: true
  address: "http://localhost:9200"
```

# 采集的关键指标列表
```yaml
# Estimated size in bytes of breaker
# 断路器预估内存大小
# Gauge
elasticsearch_breakers_estimated_size_bytes

# Limit size in bytes for breaker
# 断路器设置内存限制
# Gauge
elasticsearch_breakers_limit_size_bytes

# tripped for breaker
# 断路器累计阻断此时
# Counter 
elasticsearch_breakers_tripped

# The number of primary shards in your cluster. This is an aggregate total across all indices
# 集群主分片数量
# Gauge
elasticsearch_cluster_health_active_primary_shards

# Aggregate total of all shards across all indices, which includes replica shards.
# 集群分片总数
# Gauge
elasticsearch_cluster_health_active_shards

# Shards delayed to reduce reallocation overhead
# 暂缓重分配的分片数
# Gauge
elasticsearch_cluster_health_delayed_unassigned_shards

# Count of shards that are being freshly created
# 创建中的分片数
# Gauge
elasticsearch_cluster_health_initializing_shards

# Number of data nodes in the cluster
# 数据节点数
# Gauge
elasticsearch_cluster_health_number_of_data_nodes

# Number of nodes in the cluster
# 节点总数
# Gauge
elasticsearch_cluster_health_number_of_nodes

# Cluster level changes which have not yet been executed
# 等待执行的集群变更总数
# Gauge
elasticsearch_cluster_health_number_of_pending_tasks

# The number of shards that are currently moving from one node to another node
# 迁移中的分片数
# Gauge
elasticsearch_cluster_health_relocating_shards

# Whether all primary and replica shards are allocated
# 集群健康度
# Gauge
elasticsearch_cluster_health_status

# The number of shards that exist in the cluster state, but cannot be found in the cluster itself
# 集群未分配的分片数
# Gauge
elasticsearch_cluster_health_unassigned_shards

# Available space on block device in bytes
# 可用磁盘容量（byte）
# Gauge
elasticsearch_filesystem_data_available_bytes

# Size of block device in bytes
# 磁盘容量（byte）
# Gauge
elasticsearch_filesystem_data_size_bytes

# Count of documents on this node
# 节点文档总数
# Gauge
elasticsearch_indices_docs

# Count of deleted documents on this node
# 节点删除文档数
# Gauge
elasticsearch_indices_docs_deleted

# Count of documents with only primary shards on all nodes
# 所有节点主分片文档总数
# Gauge
elasticsearch_indices_docs_primary

# Evictions from field data
# field data cache 内存剔除次数
# Counter
elasticsearch_indices_fielddata_evictions

# Field data cache memory usage in bytes
# field data cache 内存占用（byte）
# Gauge
elasticsearch_indices_fielddata_memory_size_bytes

# Evictions from filter cache
# filter cache 内存剔除次数
# Counter
elasticsearch_indices_filter_cache_evictions

# Filter cache memory usage in bytes
# filter cache 内存占用（byte）
# Gauge
elasticsearch_indices_flush_time_seconds

# Total flushes
# flush操作次数累计
# Counter
elasticsearch_indices_flush_total

# Total time get exists in seconds
# get成功操作次数累计
# Counter
elasticsearch_indices_get_exists_time_seconds

# Total get exists operations
# get操作次数累计
# Counter
elasticsearch_indices_get_exists_total

# Total time of get missing in seconds
# get失败操作耗时累计（秒）
# Counter
elasticsearch_indices_get_missing_time_seconds

# Total get missing
# get失败操作次数累计
# Counter
elasticsearch_indices_get_missing_total

# Total get time in seconds
# get操作耗时累计（秒）
# Counter
elasticsearch_indices_get_time_seconds

# Total get
# get操作次数累计
# Counter
elasticsearch_indices_get_tota

# Total time indexing delete in seconds
# 索引删除累计耗时（秒）
# Counter
elasticsearch_indices_indexing_delete_time_seconds_total

# Total indexing deletes
# 索引删除操作次数累计
# Counter
elasticsearch_indices_indexing_delete_total

# Cumulative index time in seconds
# index操作累计耗时（秒）
# Counter
elasticsearch_indices_indexing_index_time_seconds_total

# Total index calls
# index操作数量累计
# Counter
elasticsearch_indices_indexing_index_total

# Cumulative docs merged
# merge文档数量累计
# Counter
elasticsearch_indices_merges_docs_total

# Total merges
# merge操作数量累计
# Counter
elasticsearch_indices_merges_total

# Total merge size in bytes
# merge操作数据大小累计（byte）
# Counter
elasticsearch_indices_merges_total_size_bytes_total

# Total time spent merging in seconds
# merge操作累计耗时（秒）
# Counter
elasticsearch_indices_merges_total_time_seconds_total

# Evictions from query cache
# query cache 内存剔除次数
# Counter
elasticsearch_indices_query_cache_evictions

# Query cache memory usage in bytes
# query cache 内存占用（byte）
# Gauge
elasticsearch_indices_query_cache_memory_size_bytes

# Total time spent refreshing in seconds
# refresh操作耗时累计（秒）
# Counter
elasticsearch_indices_refresh_time_seconds_total

# Total refreshes
# refresh操作次数累计
# Counter
elasticsearch_indices_refresh_total

# Total search fetch time in seconds
# fetch操作耗时累计（秒）
# Counter
elasticsearch_indices_search_fetch_time_seconds

# Total number of fetches
# fetch操作次数累计
# Counter
elasticsearch_indices_search_fetch_total

# Total search query time in seconds
# query操作耗时累计（秒）
# Counter
elasticsearch_indices_search_query_time_seconds

# Total number of queries
# query操作次数累计
# Counter
elasticsearch_indices_search_query_total

# Segments with only primary shards on all nodes
# 所有节点主分片segment总数
# Gauge
elasticsearch_indices_segment_count_primary

# Segments with all shards on all nodes
# 所有节点所有分片segment总数
# Gauge
elasticsearch_indices_segment_count_total

# Doc values with only primary shards on all nodes in bytes
# 主分片doc value内存占用（byte）
# Gauge
elasticsearch_indices_segment_doc_values_memory_bytes_primary

# Doc values with all shards on all nodes in bytes
# 所有分片doc value内存占用（byte）
# Gauge
elasticsearch_indices_segment_doc_values_memory_bytes_total

# Size of fields with only primary shards on all nodes in bytes
# 分片field内存占用（byte）
# Gauge
elasticsearch_indices_segment_fields_memory_bytes_primary

# Size of fields with all shards on all nodes in bytes
# 所有分片field内存占用（byte）
# Gauge
elasticsearch_indices_segment_fields_memory_bytes_total

# Size of fixed bit with only primary shards on all nodes in bytes
# 主分片fixed bit set内存占用（byte）
# Gauge
elasticsearch_indices_segment_fixed_bit_set_memory_bytes_primary

# Size of fixed bit with all shards on all nodes in bytes
# 所有分片fixed bit set内存占用（byte）
# Gauge
elasticsearch_indices_segment_fixed_bit_set_memory_bytes_total

# Index writer with only primary shards on all nodes in bytes
# 主分片索引写入数据量（byte）
# Gauge
elasticsearch_indices_segment_index_writer_memory_bytes_primary

# Index writer with all shards on all nodes in bytes
# 所有分片索引写入数据量（byte）
# Gauge
elasticsearch_indices_segment_index_writer_memory_bytes_total

# Size of segments with only primary shards on all nodes in bytes
# 主分片segment数
# Gauge
elasticsearch_indices_segment_memory_bytes_primary

# Size of segments with all shards on all nodes in bytes
# 所有分片segment总数
# Gauge
elasticsearch_indices_segment_memory_bytes_total

# Size of norms with only primary shards on all nodes in bytes
# 主分片normalization factor内存占用（byte）
# Gauge
elasticsearch_indices_segment_norms_memory_bytes_primary

# Size of norms with all shards on all nodes in bytes
# 所有分片normalization factor内存占用（byte）
# Gauge
elasticsearch_indices_segment_norms_memory_bytes_total

# Size of points with only primary shards on all nodes in bytes
# 主分片point内存占用（byte）
# Gauge
elasticsearch_indices_segment_points_memory_bytes_primary

# Size of points with all shards on all nodes in bytes
# 所有分片point内存占用（byte）
# Gauge
elasticsearch_indices_segment_points_memory_bytes_total

# Size of terms with only primary shards on all nodes in bytes
# 主分片term内存占用（byte）
# Gauge
elasticsearch_indices_segment_terms_memory_primary

# Number of terms with all shards on all nodes in bytes
# 所有分片term内存占用（byte）
# Gauge
elasticsearch_indices_segment_terms_memory_total

# Size of version map with only primary shards on all nodes in bytes
# 所有分片version map内存占用（byte）
# Gauge
elasticsearch_indices_segment_version_map_memory_bytes_primary

# Size of version map with all shards on all nodes in bytes
# 所有分片version map内存占用（byte）
# Gauge
elasticsearch_indices_segment_version_map_memory_bytes_total

# Count of index segments
# segment个数
# Gauge
elasticsearch_indices_segments_count

# Current memory size of segments in bytes
# segment内存占用（byte）
# Gauge
elasticsearch_indices_segments_memory_bytes

# Current size of stored index data in bytes with only primary shards on all nodes
# 主分片索引容量（byte）
# Gauge
elasticsearch_indices_store_size_bytes_primary

# Current size of stored index data in bytes with all shards on all nodes
# 所有分片索引容量（byte）
# Gauge
elasticsearch_indices_store_size_bytes_total

# Throttle time for index store in seconds
# 索引存储限制耗时（秒）
# Counter
elasticsearch_indices_store_throttle_time_seconds_total

# Total translog operations
# tranlog操作数累计
# Counter
elasticsearch_indices_translog_operations

# Total translog size in bytes
# tranlog大小累计（byte）
# Counter
elasticsearch_indices_translog_size_in_bytes

# Count of JVM GC runs
# GC运行次数累计
# Counter
elasticsearch_jvm_gc_collection_seconds_count

# GC run time in seconds
# GC运行耗时累计（秒）
# C欧BT而
elasticsearch_jvm_gc_collection_seconds_sum

# JVM memory currently committed by area
# JVM申请内存大小（byte）
# Gauge
elasticsearch_jvm_memory_committed_bytes

# JVM memory max
# JVM内存限制大小（byte）
# Gauge
elasticsearch_jvm_memory_max_bytes

# JVM memory peak used by pool
# JVM内存峰值大小（byte）
# Counter
elasticsearch_jvm_memory_pool_peak_used_bytes

# JVM memory currently used by area
# JVM内存占用大小（byte）
# Gauge
elasticsearch_jvm_memory_used_bytes

# Shortterm load average
# 系统负载（1分钟）
# Gauge
elasticsearch_os_load1

# Midterm load average
# 系统负载（5分钟）
# Gauge
elasticsearch_os_load15

# Longterm load average
# 系统负载（15分钟）
# Gauge
elasticsearch_os_load5

# Percent CPU used by process
# 进程CPU占用率
# Gauge
elasticsearch_process_cpu_percent

# Open file descriptors
# 进程打开文件数
# Gauge
elasticsearch_process_open_files_count

# Thread Pool threads active
# 活跃线程总数
# Gauge
elasticsearch_thread_pool_active_count

# Thread Pool operations completed
# 线程池complete次数
# Counter
elasticsearch_thread_pool_completed_count

# Thread Pool operations rejected
# 线程池reject次数
# Counter
elasticsearch_thread_pool_rejected_count

# Total number of bytes received
# 网络收流量（byte）
# Counter
elasticsearch_transport_rx_size_bytes_total

# Total number of bytes received
# 网络发流量（byte）
# Counter
elasticsearch_transport_tx_size_bytes_total

```

# 完整地配置项说明
```yaml
  # Enables the elasticsearch_exporter integration, allowing the Agent to automatically
  # collect system metrics from the configured ElasticSearch server address
  [enabled: <boolean> | default = false]

  # Sets an explicit value for the instance label when the integration is
  # self-scraped. Overrides inferred values.
  #
  # The default value for this integration is inferred from the hostname portion
  # of address.
  [instance: <string>]

  # Automatically collect metrics from this integration. If disabled,
  # the elasticsearch_exporter integration will be run but not scraped and thus not
  # remote-written. Metrics for the integration will be exposed at
  # /integrations/elasticsearch_exporter/metrics and can be scraped by an external
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

  # HTTP API address of an Elasticsearch node.
  [ address: <string> | default = "http://localhost:9200" ]

  # Timeout for trying to get stats from Elasticsearch.
  [ timeout: <duration> | default = "5s" ]

  # Export stats for all nodes in the cluster. If used, this flag will override the flag `node`.
  [ all: <boolean> ]

  # Node's name of which metrics should be exposed.
  [ node: <boolean> ]

  # Export stats for indices in the cluster.
  [ indices: <boolean> ]

  # Export stats for settings of all indices of the cluster.
  [ indices_settings: <boolean> ]

  # Export stats for cluster settings.
  [ cluster_settings: <boolean> ]

  # Export stats for shards in the cluster (implies indices).
  [ shards: <boolean> ]

  # Export stats for the cluster snapshots.
  [ snapshots: <boolean> ]

  # Cluster info update interval for the cluster label.
  [ clusterinfo_interval: <duration> | default = "5m" ]

  # Path to PEM file that contains trusted Certificate Authorities for the Elasticsearch connection.
  [ ca: <string> ]

  # Path to PEM file that contains the private key for client auth when connecting to Elasticsearch.
  [ client_private_key: <string> ]

  # Path to PEM file that contains the corresponding cert for the private key to connect to Elasticsearch.
  [ client_cert: <string> ]

  # Skip SSL verification when connecting to Elasticsearch.
  [ ssl_skip_verify: <boolean> ]
```
