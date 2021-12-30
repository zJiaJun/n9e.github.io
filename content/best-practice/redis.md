---
weight: 311
title: "监控Redis"
---

Telegraf可以监控Redis，相关配置如下：

```toml
[[inputs.redis]]
servers = ["tcp://localhost:6379"]
```

这是最简单的一个配置了，更多的参数可以参考telegraf.conf中的inputs.redis部分的注释，也可以把这个配置独立成一个单独的文件，参考 [监控MySQL]({{%relref "mysql"%}}) 章节。

关键是下面的监控大盘配置，json如下，可以直接导入监控大盘使用。


```json
[
  {
    "name": "Redis关键指标 - by Telegraf",
    "tags": "Redis",
    "configs": "{\"var\":[{\"name\":\"ident\",\"definition\":\"label_values(redis_used_memory, ident)\",\"selected\":\"10-255-0-34\",\"multi\":true,\"allOption\":true},{\"name\":\"server\",\"definition\":\"label_values(redis_used_memory{ident=\\\"$ident\\\"}, server)\",\"multi\":true,\"selected\":\"localhost\",\"allOption\":true},{\"name\":\"port\",\"definition\":\"label_values(redis_used_memory{ident=\\\"$ident\\\", server=\\\"$server\\\"}, port)\",\"multi\":true,\"allOption\":true,\"selected\":[\"6379\"],\"options\":[\"6379\"]}]}",
    "chart_groups": [
      {
        "name": "Default chart group",
        "weight": 0,
        "charts": [
          {
            "configs": "{\"name\":\"Redis memory\",\"QL\":[{\"PromQL\":\"redis_used_memory{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}\"},{\"PromQL\":\"redis_used_memory_lua{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}\"},{\"PromQL\":\"redis_used_memory_peak{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}\"},{\"PromQL\":\"redis_used_memory_rss{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}\"},{\"PromQL\":\"redis_maxmemory{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":12,\"x\":0,\"y\":0,\"i\":\"0\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Redis clients\",\"QL\":[{\"PromQL\":\"redis_clients{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}\"},{\"PromQL\":\"redis_blocked_clients{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":12,\"x\":12,\"y\":0,\"i\":\"1\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"commands_processed / sec\",\"QL\":[{\"PromQL\":\"rate(redis_total_commands_processed{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}[1m])\",\"Legend\":\"commands_processed_per_sec - {{ident}} - {{server}}:{{port}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":12,\"x\":0,\"y\":2,\"i\":\"2\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"keyspace hits and misses\",\"QL\":[{\"PromQL\":\"irate(redis_keyspace_hits{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}[5m])\",\"Legend\":\"hits - {{ident}} - {{server}}:{{port}}\"},{\"PromQL\":\"irate(redis_keyspace_misses{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}[5m])\",\"Legend\":\"misses - {{ident}} - {{server}}:{{port}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":12,\"x\":12,\"y\":2,\"i\":\"3\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Network IO\",\"QL\":[{\"PromQL\":\"rate(redis_total_net_input_bytes{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}[5m])\",\"Legend\":\"in_bytes - {{ident}} - {{server}}:{{port}}\"},{\"PromQL\":\"rate(redis_total_net_output_bytes{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}[5m])\",\"Legend\":\"out_bytes - {{ident}} - {{server}}:{{port}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1024},\"version\":1,\"layout\":{\"h\":2,\"w\":12,\"x\":0,\"y\":4,\"i\":\"4\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Expired Evicted keys\",\"QL\":[{\"PromQL\":\"rate(redis_expired_keys{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}[5m])\",\"Legend\":\"expired_keys - {{ident}} - {{server}}:{{port}}\"},{\"PromQL\":\"rate(redis_evicted_keys{ident=~\\\"$ident\\\", server=~\\\"$server\\\", port=~\\\"$port\\\"}[5m])\",\"Legend\":\"evicted_keys - {{ident}} - {{server}}:{{port}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":12,\"x\":12,\"y\":4,\"i\":\"5\"}}",
            "weight": 0
          }
        ]
      }
    ]
  }
]
```