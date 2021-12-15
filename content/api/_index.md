---
weight: 500
title: "API"
---

对于监控系统的API，通常有如下几个使用场景：1、读写监控数据 2、以个人用户身份调用API做一些操作，不想去WEB上操作 3、第三方系统调用监控的API做包装，用户在第三方系统操作，实际这个第三方系统底层是调用的监控的接口。

## 1.读写监控数据

对于这个场景，大家可以直接绕过夜莺，调用后端时序库的读写接口。对于写监控数据而言，如果想要走夜莺的接口，请调用n9e-server的`/opentsdb/put`接口，POST方法，该接口实现了OpenTSDB的数据协议，监控数据做成JSON放到HTTP Request Body中，举例：

```json
[
	{
		"metric": "cpu_usage_idle",
		"timestamp": 1637732157,
		"tags": {
			"cpu": "cpu-total",
			"ident": "c3-ceph01.bj"
		},
		"value": 30.5
	},
	{
		"metric": "cpu_usage_util",
		"timestamp": 1637732157,
		"tags": {
			"cpu": "cpu-total",
			"ident": "c3-ceph01.bj"
		},
		"value": 69.5
	}
]
```

显然，JSON最外层是个数组，如果只上报一条监控数据，也可以不要外面的中括号，直接把对象结构上报：

```json
{
	"metric": "cpu_usage_idle",
	"timestamp": 1637732157,
	"tags": {
		"cpu": "cpu-total",
		"ident": "c3-ceph01.bj"
	},
	"value": 30.5
}
```

服务端会看第一个字符是否是`[`，来判断上报的是数组，还是单个对象，自动做相应的Decode。如果觉得上报的内容太过占用带宽，也可以做gzip压缩，此时上报的数据，要带有`Content-Encoding: gzip`的Header。

{{% notice info %}}
注意ident这个标签，ident是identity的缩写，表示设备的唯一标识，如果标签中有ident标签，n9e-server就认为这个监控数据是来自某个机器的，会自动获取ident的value，注册到监控对象的列表里，这样后续就可以在对象看图视角页面根据监控对象筛选指标了。

如果没有ident这个标签，就没法在对象视角的看图页面筛选看图了，只能去即时查询页面通过promql查询。
{{% /notice %}}

OK，上面是推送监控数据的接口，至于查询监控数据，请大家直接调用后端时序库的接口，即Prometheus那些`/api/v1/query` `/api/v1/query_range`之类的接口。相关接口文档请参考：[Prometheus官网](https://prometheus.io/docs/prometheus/latest/querying/api/)

## 2.以个人身份模仿WEB操作

这种方式，页面上JavaScript可以调用的所有接口，你都可以用程序调用，打开chrome的开发者工具，扒拉这些接口，还是非常容易的。当然，要先登录，登录调用webapi模块的`/api/n9e/auth/login`接口，系统使用jwt认证，如果登录成功，会返回access_token和refresh_token，每次调用的时候都要把access_token放到Header里，access_token差不多15分钟过期，之后可以重新调用登录接口换token，也可以调用`/api/n9e/auth/refresh`接口用refresh_token换一个新的access_token，当然，也会顺道返回一个新的refresh_token，举例：

```bash
# 调用登录接口拿到access_token和refresh_token记录下来，后面调用其他接口的时候会用到
[root@10-255-0-34 ~]# curl -X POST 'http://localhost:18000/api/n9e/auth/login' -d '{"username": "root", "password": "root.2020"}'
{"dat":{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdXVpZCI6ImIxNTcyMjgwLWZlNzAtNDhjZi1hNDQ3LWVlMjVhZmYwMjRhZCIsImF1dGhvcml6ZWQiOnRydWUsImV4cCI6MTYzNzgyMzA1OSwidXNlcl9pZGVudGl0eSI6IjEtcm9vdCJ9.nJ56Pc7qS5Ik_UaVmlNWu_QlABaBc4pZ_WkU45u2wWk","refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MzgzMzc4NTksInJlZnJlc2hfdXVpZCI6ImIxNTcyMjgwLWZlNzAtNDhjZi1hNDQ3LWVlMjVhZmYwMjRhZCsrMS1yb290IiwidXNlcl9pZGVudGl0eSI6IjEtcm9vdCJ9.JKsbfTYBCOOfR_oPsf496N9ml9yXbP7BHb4E8Yfnzbo","user":{"id":1,"username":"root","nickname":"超管","phone":"","email":"","portrait":"","roles":["Admin"],"contacts":{},"create_at":1637545881,"create_by":"system","update_at":1637546351,"update_by":"root","admin":true}},"err":""}

# access_token放到Authorization这个Header里，Bearer的验证方式
[root@10-255-0-34 ~]# curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdXVpZCI6ImIxNTcyMjgwLWZlNzAtNDhjZi1hNDQ3LWVlMjVhZmYwMjRhZCIsImF1dGhvcml6ZWQiOnRydWUsImV4cCI6MTYzNzgyMzA1OSwidXNlcl9pZGVudGl0eSI6IjEtcm9vdCJ9.nJ56Pc7qS5Ik_UaVmlNWu_QlABaBc4pZ_WkU45u2wWk" 'http://localhost:18000/api/n9e/self/profile'
{"dat":{"id":1,"username":"root","nickname":"超管","phone":"","email":"","portrait":"","roles":["Admin"],"contacts":{},"create_at":1637545881,"create_by":"system","update_at":1637546351,"update_by":"root","admin":true},"err":""}

# 如果token过期了，后端会返回异常HTTP状态码，此时要调用refresh接口换取新的token
[root@10-255-0-34 ~]# curl -X POST 'http://localhost:18000/api/n9e/auth/refresh' -d '{"refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MzgzMzc4NTksInJlZnJlc2hfdXVpZCI6ImIxNTcyMjgwLWZlNzAtNDhjZi1hNDQ3LWVlMjVhZmYwMjRhZCsrMS1yb290IiwidXNlcl9pZGVudGl0eSI6IjEtcm9vdCJ9.JKsbfTYBCOOfR_oPsf496N9ml9yXbP7BHb4E8Yfnzbo"}'
{"dat":{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdXVpZCI6IjAxMzkzYzkxLTk5MWItNGE0Yi04ODk2LTJhZGRjMDUwYjcxMCIsImF1dGhvcml6ZWQiOnRydWUsImV4cCI6MTYzNzgyMzMxOCwidXNlcl9pZGVudGl0eSI6IjEtcm9vdCJ9.2BeWyYfcnRi3qw69zecaaeFnPFUNAGsiPIZBBnd5lug","refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MzgzMzgxMTgsInJlZnJlc2hfdXVpZCI6IjAxMzkzYzkxLTk5MWItNGE0Yi04ODk2LTJhZGRjMDUwYjcxMCsrMS1yb290IiwidXNlcl9pZGVudGl0eSI6IjEtcm9vdCJ9.zFZaRYcJI6G5maSgDVF-jZzxQ3Tb5dybIqufJhBy034"},"err":""}
```

## 3.第三方系统调用夜莺

比如第三方系统想获取夜莺中的所有未恢复告警，或者获取夜莺中的全量用户列表，这些需求，建议走`/v1/n9e`打头的接口，这些接口走BasicAuth认证，BasicAuth的用户名和密码在webapi.conf中可以找到，就是BasicAuth那个section的配置。当前这个阶段，还没有哪个系统会依赖夜莺的接口，所以，这个`/v1/n9e`前缀的接口目前一个都还没有提供，不过代码框架已经搭起来了，代码在`src/webapi/router/router.go`文件中，service那个路由Group，如果贵司要封装夜莺的接口，可能要在这个路由分组下加一些路由配置了。作为开源软件，说清楚原理就好了，如果贵司仍然搞不明白可以联系我们，我们提供商业技术支持服务 :-)

