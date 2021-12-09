---
weight: 80
title: "监控Linux操作系统"
---

部署了Telegraf即可采集到常见的监控指标了，Telegraf具体使用前面有章节介绍了，这里不再赘述，这里主要提供常见大盘配置和告警规则配置的JSON，便于大家快速上手。

## Linux操作系统监控大盘

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

## Linux操作系统常用告警规则

```json
[
  {
    "name": "有地址PING不通，请注意",
    "note": "",
    "severity": 1,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "ping_result_code != 0",
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
    "name": "有监控对象失联",
    "note": "",
    "severity": 1,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "target_up != 1",
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
    "name": "有端口探测失败，请注意",
    "note": "",
    "severity": 1,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "net_response_result_code != 0",
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
    "name": "机器负载-CPU较高，请关注",
    "note": "",
    "severity": 3,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "cpu_usage_idle{cpu=\"cpu-total\"} < 25",
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
    "name": "机器负载-内存较高，请关注",
    "note": "",
    "severity": 2,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "mem_available_percent < 25",
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
    "name": "硬盘-IO非常繁忙",
    "note": "",
    "severity": 2,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "rate(diskio_io_time[1m])/10 > 99",
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
    "name": "硬盘-预计再有4小时写满",
    "note": "",
    "severity": 1,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "predict_linear(disk_free[1h], 4*3600) < 0",
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
    "name": "网卡-入向有丢包",
    "note": "",
    "severity": 3,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "increase(net_drop_in[1m]) > 0",
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
    "name": "网卡-出向有丢包",
    "note": "",
    "severity": 3,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "increase(net_drop_out[1m]) > 0",
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
    "name": "网络连接-TME_WAIT数量超过2万",
    "note": "",
    "severity": 2,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "netstat_tcp_time_wait > 20000",
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
    "name": "进程监控-有进程数为0，某进程可能挂了",
    "note": "",
    "severity": 1,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "procstat_lookup_running == 0",
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
    "name": "进程监控-进程句柄限制过小",
    "note": "",
    "severity": 3,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "procstat_rlimit_num_fds_soft < 2048",
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
    "name": "进程监控-采集失败",
    "note": "",
    "severity": 1,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "procstat_lookup_result_code != 0",
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

## Grafana大盘

笔者做了一个Grafana大盘：[https://grafana.com/grafana/dashboards/15365](https://grafana.com/grafana/dashboards/15365 )使用Telegraf做采集、Prometheus做数据源、Nightingale生成的target_up指标来标识机器是否up，欢迎试用



