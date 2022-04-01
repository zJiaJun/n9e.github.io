---
weight: 34
title: "告警通知媒介"
---

告警引擎模块是n9e-server，发送也是这个模块，查看server.conf，可以看到Alerting相关的一些配置，都是和告警发送相关的：

```toml
[SMTP]
Host = "smtp.163.com"
Port = 994
User = "username"
Pass = "password"
From = "username@163.com"
InsecureSkipVerify = true
Batch = 5

[Alerting]
TemplatesDir = "./etc/template"
NotifyConcurrency = 10
# use builtin go code notify by default
NotifyBuiltinEnable = true

[Alerting.CallScript]
# built in sending capability in go code
# so, no need enable script sender
Enable = false
ScriptPath = "./etc/script/notify.py"

[Alerting.CallPlugin]
Enable = false
# use a plugin via `go build -buildmode=plugin -o notify.so`
PluginPath = "./etc/script/notify.so"
Caller = "n9eCaller"

[Alerting.RedisPub]
Enable = false
# complete redis key: ${ChannelPrefix} + ${Cluster}
ChannelPrefix = "/alerts/"

[Alerting.Webhook]
Enable = false
Url = "http://a.com/n9e/callback"
BasicAuthUser = ""
BasicAuthPass = ""
Timeout = "5s"
Headers = ["Content-Type", "application/json", "X-From", "N9E"]
```

SMTP相关的是邮件发送的配置；剩下的Alerting相关的配置，都可以不用动，默认夜莺支持了邮件、钉钉、企微、飞书四种通知机制；对于发送的消息内容，有个通知模板，在Alerting.TemplatesDir指定的配置目录里可以看到一堆模板文件，采用Go的模板语法，如果想修改消息内容就是修改这些模板文件即可。

如果想二次开发，自行对告警做处理，可以启用Alerting.CallScript、Alerting.CallPlugin、Alerting.RedisPub、Alerting.Webhook等配置段，分别表示，通过脚本的方式自行处理发送告警、通过Plugin的方式来发送告警、让系统把告警消息统一推送给Redis，启用全局的Webhook，告警之后夜莺回调一个全局的Url，您可以在这个回调地址里写自己的定制逻辑。
