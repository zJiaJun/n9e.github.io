---
weight: 10
title: "监控大盘-导入大盘JSON"
---

监控大盘有个导入导出功能，这里提供一些常见的监控大盘模板，大家可以直接导入使用

## Linux基本监控指标-Telegraf采集

```json
[
  {
    "name": "Linux基本监控指标-Telegraf采集",
    "tags": "HOST",
    "configs": "{\"var\":[{\"name\":\"host\",\"definition\":\"label_values(mem_free, ident)\"}]}",
    "chart_groups": [
      {
        "name": "Default chart group",
        "weight": 0,
        "charts": [
          {
            "configs": "{\"name\":\"整机CPU空闲率(%)\",\"QL\":[{\"PromQL\":\"cpu_usage_idle{cpu=\\\"cpu-total\\\", ident=\\\"$host\\\"}\"}],\"yplotline1\":35,\"yplotline2\":15,\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"asc\",\"precision\":\"origin\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":0,\"y\":0,\"i\":\"0\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"内存可用率(%)\",\"QL\":[{\"PromQL\":\"mem_available_percent{ident=\\\"$host\\\"}\"}],\"yplotline1\":30,\"yplotline2\":15,\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"asc\",\"precision\":\"origin\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":8,\"y\":0,\"i\":\"1\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"硬盘利用率(%)\",\"QL\":[{\"PromQL\":\"disk_used_percent{ident=\\\"$host\\\"}\"}],\"yplotline1\":87,\"yplotline2\":92,\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"origin\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":16,\"y\":0,\"i\":\"2\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"IO.UTIL(%)\",\"QL\":[{\"PromQL\":\"rate(diskio_io_time{ident=\\\"$host\\\"}[1m])/10\"}],\"yplotline1\":90,\"yplotline2\":null,\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"origin\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":0,\"y\":2,\"i\":\"3\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"网卡每分钟丢包数（个）\",\"QL\":[{\"PromQL\":\"increase(net_drop_in{ident=\\\"$host\\\"}[1m])\",\"Legend\":\"net_drop_in ident:{{ident}} interface:{{interface}}\"},{\"PromQL\":\"increase(net_drop_out{ident=\\\"$host\\\"}[1m])\",\"Legend\":\"net_drop_out ident:{{ident}} interface:{{interface}}\"}],\"yplotline1\":5,\"yplotline2\":20,\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":8,\"y\":2,\"i\":\"4\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"TCP_TIME_WAIT数量\",\"QL\":[{\"PromQL\":\"netstat_tcp_time_wait{ident=\\\"$host\\\"}\"}],\"yplotline1\":null,\"yplotline2\":20000,\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":16,\"y\":2,\"i\":\"5\"}}",
            "weight": 0
          }
        ]
      }
    ]
  }
]
```

