---
weight: 310
title: "监控MySQL"
---

使用Telegraf监控MySQL也较为简单，会采集很多指标，但是具体应该关注哪些指标，是个难点，本章会提供一些例子，欢迎大家补充最佳实践。

Telegraf的配置为了便于管理，可以拆成多个文件，放到统一目录中，使用 `--config-directory` 参数指定具体的目录，举例：

```bash
# telegraf 启动命令：
./usr/bin/telegraf --config ./etc/telegraf/telegraf.conf --config-directory ./etc/telegraf.d

# 后面如果修改了telegraf.conf或者修改了telegraf.d下的配置，可以通过如下方式让Telegraf重新加载配置
kill -HUP `pidof telegraf`
```

我们创建一个mysql.conf的配置放到telegraf.d下，内容如下：

```toml
[[inputs.mysql]]
servers = ["root:1234@tcp(localhost:3306)/?tls=false"]
metric_version = 2
gather_global_variables = true
interval_slow = "1m"
tagexclude = ["innodb_version"]
```

mysql采集插件的具体使用方式，参考 [这里](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/mysql) 的文档。下面笔者整理了监控大盘和告警规则，供大家参考：

大盘JSON：

```json
[
  {
    "name": "MySQL关键指标 - by Telegraf",
    "tags": "MySQL",
    "configs": "{\"var\":[{\"name\":\"ident\",\"selected\":\"10-255-0-34\",\"definition\":\"label_values(mysql_uptime, ident)\"},{\"name\":\"server\",\"definition\":\"label_values(mysql_uptime{ident=\\\"$ident\\\"}, server)\",\"selected\":\"localhost_3306\"}]}",
    "chart_groups": [
      {
        "name": "MySQL关键指标",
        "weight": 0,
        "charts": [
          {
            "configs": "{\"name\":\"Connections\",\"QL\":[{\"PromQL\":\"rate(mysql_aborted_connects{ident=\\\"$ident\\\", server=\\\"$server\\\"}[5m])\",\"Legend\":\"aborted_connections - {{ident}} - {{server}}\"},{\"PromQL\":\"mysql_variables_max_connections{ident=\\\"$ident\\\", server=\\\"$server\\\"}\",\"Legend\":\"max_connections - {{ident}} - {{server}}\"},{\"PromQL\":\"mysql_threads_connected{ident=\\\"$ident\\\", server=\\\"$server\\\"}\",\"Legend\":\"threads_connected - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":0,\"y\":0,\"i\":\"0\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Slow Queries\",\"QL\":[{\"PromQL\":\"rate(mysql_slow_queries{ident=\\\"$ident\\\", server=\\\"$server\\\"}[5m])\",\"Legend\":\"slow_queries - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":8,\"y\":0,\"i\":\"1\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Open Files\",\"QL\":[{\"PromQL\":\"mysql_variables_open_files_limit{ident=\\\"$ident\\\", server=\\\"$server\\\"}\",\"Legend\":\"open_files_limit - {{ident}} - {{server}}\"},{\"PromQL\":\"mysql_variables_innodb_open_files{ident=\\\"$ident\\\", server=\\\"$server\\\"}\",\"Legend\":\"open_files_used - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":16,\"y\":0,\"i\":\"2\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Queries per second\",\"QL\":[{\"PromQL\":\"rate(mysql_queries{ident=\\\"$ident\\\", server=\\\"$server\\\"}[1m])\",\"Legend\":\"mysql_queries - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":0,\"y\":2,\"i\":\"3\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Writes per second\",\"QL\":[{\"PromQL\":\"rate(mysql_com_insert{ident=\\\"$ident\\\", server=\\\"$server\\\"}[1m])\",\"Legend\":\"command_insert - {{ident}} - {{server}}\"},{\"PromQL\":\"rate(mysql_com_update{ident=\\\"$ident\\\", server=\\\"$server\\\"}[1m])\",\"Legend\":\"command_update - {{ident}} - {{server}}\"},{\"PromQL\":\"rate(mysql_com_delete{ident=\\\"$ident\\\", server=\\\"$server\\\"}[1m])\",\"Legend\":\"command_delete - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":16,\"y\":2,\"i\":\"4\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Threads\",\"QL\":[{\"PromQL\":\"mysql_threads_running{ident=\\\"$ident\\\", server=\\\"$server\\\"}\",\"Legend\":\"threads_running - {{ident}} - {{server}}\"},{\"PromQL\":\"mysql_threads_connected{ident=\\\"$ident\\\", server=\\\"$server\\\"}\",\"Legend\":\"threads_connected - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":16,\"y\":4,\"i\":\"5\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"Data Reads/Writes per second\",\"QL\":[{\"PromQL\":\"rate(mysql_innodb_data_reads{ident=\\\"$ident\\\", server=\\\"$server\\\"}[5m])\",\"Legend\":\"mysql_innodb_data_reads - {{ident}} - {{server}}\"},{\"PromQL\":\"rate(mysql_innodb_data_writes{ident=\\\"$ident\\\", server=\\\"$server\\\"}[5m])\",\"Legend\":\"mysql_innodb_data_writes - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":0,\"y\":4,\"i\":\"6\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"InnoDB Buffer Pool Size\",\"QL\":[{\"PromQL\":\"mysql_variables_innodb_buffer_pool_size{ident=\\\"$ident\\\", server=\\\"$server\\\"}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":8,\"y\":4,\"i\":\"7\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"InnoDB Buffer Pool Pages\",\"QL\":[{\"PromQL\":\"mysql_innodb_buffer_pool_pages_free{ident=\\\"$ident\\\", server=\\\"$server\\\"}\"},{\"PromQL\":\"mysql_innodb_buffer_pool_pages_data{ident=\\\"$ident\\\", server=\\\"$server\\\"}\"},{\"PromQL\":\"mysql_innodb_buffer_pool_pages_total{ident=\\\"$ident\\\", server=\\\"$server\\\"}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":11,\"x\":0,\"y\":6,\"i\":\"8\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"TPS\",\"QL\":[{\"PromQL\":\"rate(mysql_com_commit{ident=\\\"$ident\\\", server=\\\"$server\\\"}[1m]) + rate(mysql_com_rollback{ident=\\\"$ident\\\", server=\\\"$server\\\"}[1m])\",\"Legend\":\"tps - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":8,\"x\":8,\"y\":2,\"i\":\"9\"}}",
            "weight": 0
          },
          {
            "configs": "{\"name\":\"InnoDB Buffer Pool Hit Rate\",\"QL\":[{\"PromQL\":\"(increase(mysql_innodb_buffer_pool_read_requests[1m])-increase(mysql_innodb_buffer_pool_reads[1m]))/increase(mysql_innodb_buffer_pool_read_requests[1m])*100\",\"Legend\":\"rate - {{ident}} - {{server}}\"}],\"legend\":false,\"highLevelConfig\":{\"shared\":true,\"sharedSortDirection\":\"desc\",\"precision\":\"short\",\"formatUnit\":1000},\"version\":1,\"layout\":{\"h\":2,\"w\":13,\"x\":11,\"y\":6,\"i\":\"10\"}}",
            "weight": 0
          }
        ]
      }
    ]
  }
]
```

告警规则JSON：

```json
[
  {
    "name": "MySQL InnoDB buffer pool 命中率太低",
    "note": "",
    "severity": 2,
    "disabled": 0,
    "prom_for_duration": 0,
    "prom_ql": "(increase(mysql_innodb_buffer_pool_read_requests[1m])-increase(mysql_innodb_buffer_pool_reads[1m]))/increase(mysql_innodb_buffer_pool_read_requests[1m])*100 < 99",
    "prom_eval_interval": 15,
    "enable_stime": "00:00",
    "enable_etime": "23:59",
    "enable_days_of_week": [
      "1",
      "2",
      "3",
      "4",
      "5",
      "6",
      "0"
    ],
    "notify_recovered": 1,
    "notify_channels": [
      "email",
      "dingtalk",
      "wecom"
    ],
    "notify_repeat_step": 60,
    "callbacks": [],
    "runbook_url": "",
    "append_tags": []
  },
  {
    "name": "MySQL出现慢查询",
    "note": "",
    "severity": 3,
    "disabled": 0,
    "prom_for_duration": 0,
    "prom_ql": "rate(mysql_slow_queries[5m]) > 0",
    "prom_eval_interval": 15,
    "enable_stime": "00:00",
    "enable_etime": "23:59",
    "enable_days_of_week": [
      "1",
      "2",
      "3",
      "4",
      "5",
      "6",
      "0"
    ],
    "notify_recovered": 1,
    "notify_channels": [
      "email",
      "dingtalk",
      "wecom"
    ],
    "notify_repeat_step": 60,
    "callbacks": [],
    "runbook_url": "",
    "append_tags": []
  },
  {
    "name": "MySQL出现连接失败的情况",
    "note": "或许需要调整max_connections",
    "severity": 1,
    "disabled": 0,
    "prom_for_duration": 0,
    "prom_ql": "rate(mysql_aborted_connects[5m]) > 0",
    "prom_eval_interval": 15,
    "enable_stime": "00:00",
    "enable_etime": "23:59",
    "enable_days_of_week": [
      "1",
      "2",
      "3",
      "4",
      "5",
      "6",
      "0"
    ],
    "notify_recovered": 1,
    "notify_channels": [
      "email",
      "dingtalk",
      "wecom"
    ],
    "notify_repeat_step": 60,
    "callbacks": [],
    "runbook_url": "",
    "append_tags": []
  },
  {
    "name": "MySQL句柄快用完了",
    "note": "",
    "severity": 2,
    "disabled": 0,
    "prom_for_duration": 0,
    "prom_ql": "100 * mysql_variables_innodb_open_files / mysql_variables_open_files_limit > 90",
    "prom_eval_interval": 15,
    "enable_stime": "00:00",
    "enable_etime": "23:59",
    "enable_days_of_week": [
      "1",
      "2",
      "3",
      "4",
      "5",
      "6",
      "0"
    ],
    "notify_recovered": 1,
    "notify_channels": [
      "email",
      "dingtalk",
      "wecom"
    ],
    "notify_repeat_step": 60,
    "callbacks": [],
    "runbook_url": "",
    "append_tags": []
  },
  {
    "name": "MySQL近期有重启",
    "note": "",
    "severity": 3,
    "disabled": 0,
    "prom_for_duration": 0,
    "prom_ql": "mysql_uptime < 1800",
    "prom_eval_interval": 15,
    "enable_stime": "00:00",
    "enable_etime": "23:59",
    "enable_days_of_week": [
      "1",
      "2",
      "3",
      "4",
      "5",
      "6",
      "0"
    ],
    "notify_recovered": 1,
    "notify_channels": [
      "email",
      "dingtalk",
      "wecom"
    ],
    "notify_repeat_step": 60,
    "callbacks": [],
    "runbook_url": "",
    "append_tags": []
  },
  {
    "name": "MySQL连接数快用完了",
    "note": "或许需要调整max_connections",
    "severity": 2,
    "disabled": 0,
    "prom_for_duration": 0,
    "prom_ql": "100 * mysql_threads_connected / mysql_variables_max_connections > 90",
    "prom_eval_interval": 15,
    "enable_stime": "00:00",
    "enable_etime": "23:59",
    "enable_days_of_week": [
      "1",
      "2",
      "3",
      "4",
      "5",
      "6",
      "0"
    ],
    "notify_recovered": 1,
    "notify_channels": [
      "email",
      "dingtalk",
      "wecom"
    ],
    "notify_repeat_step": 60,
    "callbacks": [],
    "runbook_url": "",
    "append_tags": []
  }
]
```


