---
title: "使用Telegraf采集监控数据"
weight: 3
---

>Telegraf 是 InfluxData 开源的一款采集器，可以采集操作系统、各种中间件的监控指标，采集[目标列表](https://github.com/influxdata/telegraf/tree/master/plugins/inputs)，看起来是非常丰富，Telegraf是一个大一统的设计，即一个二进制可以采集CPU、内存、mysql、mongodb、redis、snmp等，不像Prometheus的exporter，每个监控对象一个exporter，管理起来略麻烦。一个二进制分发起来确实比较方便。

这里提供快速安装的教程，Telegraf的更多知识，请参考[Telegraf官网](https://github.com/influxdata/telegraf)，笔者之前也写了一个[Telegraf调研笔记，讲解了Telegraf的基本用法，一定要看！！！](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU3ODAxNTIzMQ==&action=getalbum&album_id=2124352687600205826&scene=173&from_msgid=2247484223&from_itemidx=1&count=3&nolastread=1)，大家亦可参考。

Telegraf下载地址在[这里](https://github.com/influxdata/telegraf/releases)，根据自己的平台选择对应的二进制下载即可。笔者的环境是CentOS，下面是安装脚本，/opt/telegraf/telegraf.conf 是一个经过删减的干净的配置文件，指定了opentsdb output plugin，这个plugin的写入地址配置的是n9e-server，所以，Telegraf采集的数据会被推送给n9e-server，二者贯通：

```bash
#!/bin/sh

version=1.20.4
tarball=telegraf-${version}_linux_amd64.tar.gz
wget https://dl.influxdata.com/telegraf/releases/$tarball
tar xzvf $tarball

mkdir -p /opt/telegraf
cp -far telegraf-${version}/usr/bin/telegraf /opt/telegraf

cat <<EOF > /opt/telegraf/telegraf.conf
[global_tags]

[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  hostname = ""
  omit_hostname = false

[[outputs.opentsdb]]
  host = "http://127.0.0.1"
  port = 19000
  http_batch_size = 50
  http_path = "/opentsdb/put"
  debug = false
  separator = "_"

[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = true

[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

[[inputs.diskio]]

[[inputs.kernel]]

[[inputs.mem]]

[[inputs.processes]]

[[inputs.system]]
  fielddrop = ["uptime_format"]

[[inputs.net]]
  ignore_protocol_stats = true

EOF

cat <<EOF > /etc/systemd/system/telegraf.service
[Unit]
Description="telegraf"
After=network.target

[Service]
Type=simple

ExecStart=/opt/telegraf/telegraf --config telegraf.conf
WorkingDirectory=/opt/telegraf

SuccessExitStatus=0
LimitNOFILE=65536
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=telegraf
KillMode=process
KillSignal=SIGQUIT
TimeoutStopSec=5
Restart=always


[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable telegraf
systemctl restart telegraf
systemctl status telegraf
```

{{% notice warning %}}
`/opt/telegraf/telegraf.conf`的内容是个删减版，只是为了让大家快速跑起来，如果要采集更多监控对象，比如mysql、redis、tomcat等，还是要仔细去阅读从tarball里解压出来的那个配置文件，那里有很详细的注释，也可以参考官方提供的各个采集插件下的[README](https://github.com/influxdata/telegraf/tree/master/plugins/inputs)
{{% /notice %}}


💡 [Telegraf告警策略](https://github.com/didi/nightingale/blob/main/etc/alerts/linux_by_telegraf.json) | [Telegraf监控大盘](https://github.com/didi/nightingale/blob/main/etc/dashboards/linux_by_telegraf.json)

