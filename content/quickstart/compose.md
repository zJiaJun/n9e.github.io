---
title: "使用Docker Compose快速部署"
weight: 1
---

> 采用Docker Compose做编排，用于单机快速启动环境做测试，包含了MySQL、Redis、Prometheus、Ibex、Nightingale、Telegraf

从[Github](https://github.com/didi/nightingale)下载夜莺的源码，进入docker目录，执行`docker-compose up -d`即可，docker-compose会自动拉取镜像并启动，查看各个容器启动状态，使用命令`docker-compose ps`，都是Up状态则表示启动成功。

如果ibex、nserver、nwebapi等模块一直在Restarting，可能是数据库容器启动太慢了没有准备好，可以执行`docker-compose down`停掉再重新尝试启动测试，这个问题受限于个人知识水平一直不知道如何解决，如果有对 docker-compose 机制熟悉的小伙伴可以赐教下，怎么保证各个容器的启动顺序，要求 MySQL 进程启动之后，其他的进程才启动。

```bash
$ git clone https://github.com/didi/nightingale.git
$ cd nightingale/docker
$ docker-compose up -d
Creating network "docker_nightingale" with driver "bridge"
Creating mysql      ... done
Creating redis      ... done
Creating prometheus ... done
Creating ibex       ... done
Creating agentd     ... done
Creating nwebapi    ... done
Creating nserver    ... done
Creating telegraf   ... done
$ docker-compose ps
   Name                 Command               State                                   Ports
----------------------------------------------------------------------------------------------------------------------------
agentd       /app/ibex agentd                 Up      10090/tcp, 20090/tcp
ibex         /app/ibex server                 Up      0.0.0.0:10090->10090/tcp, 0.0.0.0:20090->20090/tcp
mysql        docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp, 33060/tcp
nserver      /app/n9e server                  Up      18000/tcp, 0.0.0.0:19000->19000/tcp
nwebapi      /app/n9e webapi                  Up      0.0.0.0:18000->18000/tcp, 19000/tcp
prometheus   /bin/prometheus --config.f ...   Up      0.0.0.0:9090->9090/tcp
redis        docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp
telegraf     /entrypoint.sh telegraf          Up      0.0.0.0:8092->8092/udp, 0.0.0.0:8094->8094/tcp, 0.0.0.0:8125->8125/udp
```

更多docker-compose相关知识请参考[官网](https://docs.docker.com/compose/)

{{% notice warning %}}
启动成功之后，建议把initsql目录下的内容挪走，这样下次重启的时候，DB就不会重新初始化了。否则下次启动mysql还是会自动执行initsql下面的sql文件导致DB重新初始化，页面上创建的规则、用户等都会丢失。

docker-compose 这种部署方式，只是用于简单测试，不推荐在生产环境使用，当然了，如果您是 docker-compose 专家，另当别论。
{{% /notice %}}

服务启动之后，浏览器访问nwebapi的端口，即18000，默认用户是`root`，密码是`root.2020`
