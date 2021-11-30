---
title: "源码编译夜莺前后端及告警自愈模块"
weight: 20
---

> 本节讲述Nightingale的源码编译方式，分前后端两部分。另外，如果用到告警自愈模块，会用到ibex这个模块，本节也会一并讲解ibex模块的编译方法

## 后端

Nightingale后端采用Go语言编写，编译的前置条件就是配置Go的开发环境。

### 配置Go环境

到[Go官网](https://golang.google.cn/dl/)选择对应的版本下载，我的环境是Linux，选择的[go1.17.3.linux-amd64.tar.gz](https://s3-gz01.didistatic.com/n9e-pub/tgz/go1.17.3.linux-amd64.tar.gz)，直接下载到/root目录下了，然后解压缩，即Go的内容都放到了/root/go目录下了。同时准备gopath目录，如下：

```bash
cd /root && mkdir -p gopath/src
echo "GOROOT=/root/go" >> .bash_profile
echo "GOPATH=/root/gopath" >> .bash_profile
echo 'export PATH=$GOROOT/bin:$GOPATH/bin:$PATH' >> .bash_profile
source .bash_profile
```

### 编译n9e

```bash
git clone https://gitee.com/n9e/n9e.git

# 国内配置一下代理，可以加速编译
export GOPROXY=https://goproxy.cn

# 执行编译
cd n9e && make
```

编译完成之后如果生成二进制：n9e，就表示编译成功！想要快速入门Go语言？可以参考[GOCN](https://gocn.vip/wiki)的资料！

### 编译ibex

如果需要告警自愈能力，夜莺依赖ibex做命令下发执行，ibex的编译和n9e几乎一模一样，如下：

```bash
git clone https://gitee.com/cnperl/ibex.git

# 国内配置一下代理，可以加速编译
export GOPROXY=https://goproxy.cn

# 执行编译
cd ibex && make
```

编译完成之后如果生成二进制：ibex，就表示编译成功！


## 前端

```bash
git clone https://github.com/n9e/fe-v5.git
cd fe-v5
npm run build
```

