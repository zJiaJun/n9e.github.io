---
weight: 5
title: "多种告警通知媒介的支持"
---

告警通知媒介，是说发告警的时候，发送邮件、钉钉、企微之类的，夜莺默认提供的就是这三种，在告警规则配置的时候，可以勾选想通过什么方式发送。页面上的选项，来自webapi.conf的配置文件，NotifyChannels这个配置项，所以，如果想扩展一种新的通知媒介，就要修改这个配置了。比如我们加上一个新的发送方式，叫feishu，即，把NotifyChannels设置为：`NotifyChannels = [ "email", "dingtalk", "wecom", "feishu" ]`，页面上就能看到feishu这个选项了。

虽然这里加了新的通知媒介，用户可以勾选了，但是没有具体的处理逻辑也没有用，为了便于扩展，夜莺在etc/script目录下放置了一个notify.py，告警通知最终都是通过这个脚本发送的，如果我们想加入feishu的发送逻辑，那就要修改这个脚本。这个脚本总共不超过200行，还是比较容易理解的。

原理上来讲，告警判断是n9e-server这个模块负责的，n9e-server生成告警事件之后，会把告警事件序列化为JSON，然后调用notify.py，调用脚本的时候，会把告警事件那个大JSON，传给这个notify.py的标准输入，作为脚本的一个输入信息。剩下的就交由这个脚本了。如果夜莺处理过告警发送了，会在夜莺的启动目录下找到一个.payload的隐藏文件，这个文件的内容就是n9e-server序列化的JSON内容，相当于n9e-server先生成了.payload文件，然后调用 `./etc/script/notify.py < .payload` 所以，如果大家发现脚本工作不正常，可以用这个命令做测试。

这种方式，大家可以修改notify.py，非常非常灵活，想加入一些新的通知媒介，或者想和自己的系统做一些打通工作，都非常简单。这个设计其实从夜莺5.0版本就引入了，反馈最多的几个问题如下，这里也做一一解答：

**1. n9e-server报错找不到这个文件**

通常是因为启动路径不对，要去夜莺的etc目录的同级目录启动n9e-server，如果是用systemd托管的就要把WorkingDirectory配对。

**2. python找不到requests库**

在5.0版本的时候，调用微信、钉钉的接口，都是用的requests这个库，所以需要手工提前安装一下。不过从5.1开始，就不用这个库了，用了内置的urllib2，减少外部依赖。

**3. python找不到bottle库**

在5.0版本的时候，告警事件作为JSON传给notify.py，notify.py要发送邮件、钉钉等，需要拼接发送的内容，拼接发送内容的过程中，用到了模板库，bottle是个模板库，为了减少外部依赖，5.1版本做了新的设计，发送内容的拼接不再放在notify.py里了，由n9e-server来做。etc/template目录下放置了多个模板文件（大家想扩展新的模板的话，直接在这个目录下新增即可，以.tpl作为文件后缀），每次n9e-server调用notify.py的时候，都会调用这些模板文件，把告警事件的各个字段传入，拼成具体的通知消息。这些通知消息也会放到序列化的那个大JSON中传给notify.py。我们在notify.py中可以看到这么一行代码：`mail_body = payload.get('tpls').get("mailbody.tpl", "mailbody.tpl not found")`，这就是从JSON中获取n9e-server提前拼好的邮件内容。
