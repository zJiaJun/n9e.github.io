---
weight: 300
title: "监控URL"
---

之前在写调研笔记的时候，测试了PING监控和TCP探测监控，调研笔记在 [这里](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU3ODAxNTIzMQ==&action=getalbum&album_id=2124352687600205826&scene=173&from_msgid=2247484223&from_itemidx=1&count=3&nolastread=1) 这个章节主要给大家讲解域名URL探测。直接上测试配置：

```toml
[[inputs.http_response]]
urls = ["https://www.baidu.com", "http://ulricqin.io/ping"]
response_timeout = "5s"
method = "GET"
fielddrop = ["result_type"]
tagexclude = ["result", "status_code"]
```

`https://www.baidu.com` 显然是通的，`http://ulricqin.io/ping` 这个是个假的URL，不通，我们测试一下输出的内容：

```bash
[root@10-255-0-34 telegraf-1.20.3]# ./usr/bin/telegraf --config etc/telegraf/telegraf.conf --input-filter http_response --test
2021-12-13T04:16:43Z I! Starting Telegraf 1.20.3
> http_response,host=10-255-0-34,method=GET,server=https://www.baidu.com content_length=227i,http_response_code=200i,response_time=0.028757521,result_code=0i 1639369003000000000
> http_response,host=10-255-0-34,method=GET,server=http://ulricqin.io/ping result_code=5i 1639369003000000000
```

这里有个字段是result_code，用这个字段配置告警即可，正常可以访问的URL，result_code是0，不正常就是非0，告警规则里可以配置如下promql：

```
http_response_result_code != 0
```

或者直接在夜莺的告警规则页面导入这条告警规则JSON：

```json
[
  {
    "name": "有URL探测失败，请注意",
    "note": "",
    "severity": 1,
    "disabled": 0,
    "prom_for_duration": 60,
    "prom_ql": "http_response_result_code != 0",
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

如果想对域名返回的statuscode或者response body的内容做判断，Telegraf也是支持的，使用response_status_code和response_string_match这些字段配置，配置文件里有样例，大家可以自行参考下。




