---
title: nginx动态配置方案：confd + redis
date: 2017-07-24 18:54:00
tags: [nginx,confd]
---
## 前言
confd 是一个使用GO语言编写的开源项目，用来自动构建文件。

通过简单的配置，可以自动生成配置文件以及重启服务，解决某些需要第三方介入重启的场景。

项目源码：https://github.com/kelseyhightower/confd

## 实操

### 业务场景描述

为了实现多租户模式，需要在创建了租户之后修改nginx的配置文件，同时让nginx重新读取配置（nginx reload）。

<!-- more -->

- 错误方案一

{% note warning %} 
原来想通过 php 直接进行 nginx reload，但是由于 php 所在的进程实际上是 worker进程，用户是 www-data，不够权限执行 nginx reload 命令，导致方案失败（当然如果强行给php赋权也有可能达到重载效果，但是风险实在太大）。
{% endnote %}

- 错误方案二

{% note warning %} 
由php生成一个临时文件，然后写一个定时任务，通过识别临时文件，然后执行 nginx reload（后来了解到confd可以直接识别配置变动，所以放弃了这个方案）。
{% endnote %}

- 选用方案

{% note success %} 
使用confd + redis来实现动态配置。confd设置成常驻模式，10秒读取一次redis，匹配键为 /nginx/* 的内容，然后生成配置文件。
{% endnote %}

### 安装 go

由于confd是基于go语言开发的，所以需要先安装go环境
```
// 安装go的环境（linux版本是ubuntu16.04）
apt-get install go

// 配置go的目录
export GOROOT=/usr/lib/go 
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
export GOPATH=$GOROOT/mygo (可选设置)
```

### 安装confd
```
$ mkdir -p $GOPATH/src/github.com/kelseyhightower
$ git clone https://github.com/kelseyhightower/confd.git $GOPATH/src/github.com/kelseyhightower/confd
$ cd $GOPATH/src/github.com/kelseyhightower/confd
$ ./build
```

### 安装完成后，即可进入 bin/目录下运行confd
```
ls bin/
./confd
```

### redis + confd 的对接案例

- 在redis中设置两个变量

```
redis-cli set /myapp/database/url db.example.com
redis-cli set /myapp/database/user rob
```

- 为confd创建模板和配置文件

```
// 创建文件 myconfig.toml
/etc/confd/conf.d/myconfig.toml

[template]
src = "myconfig.conf.tmpl"
dest = "/tmp/myconfig.conf"
keys = [
    "/myapp/database/url",
    "/myapp/database/user",
]

// 创建文件
/etc/confd/templates/myconfig.conf.tmpl

[myconfig]
database_url = {{getv "/myapp/database/url"}}
database_user = {{getv "/myapp/database/user"}}

```

- 生成配置文件

```
// 执行生成操作
./confd -onetime -backend redis -node 127.0.0.1:6379

// 全路径地址
/usr/lib/go/mygo/src/github.com/kelseyhightower/confd/bin/confd -onetime -backend redis -node 127.0.0.1:6379

// 执行后提示一下信息
INFO Backend set to redis
INFO Starting confd
INFO Backend nodes set to 127.0.0.1:6379
INFO Target config /tmp/myconfig.conf out of sync
INFO Target config /tmp/myconfig.conf has been updated
```

- 生成完后，去/tmp目录下打开生成的文件myconfig.conf，代码如下：

```
# This a comment
[myconfig]
database_url = db.example.com
database_user = rob
```


### 监听redis变化，自动生成配置

在对接SAAS服务时，需要用到自动部署，即生成租户实例后，系统向redis写入租户的信息，然后confd自动检测，生成配置以及重启nginx。

#### 实现流程如下：

- 增加confd的配置文件，内容如下

```
// 创建confd的配置文件
touch /etc/confd/confd.conf

// 打开文本，写入以下内容
backend = "redis"
node = "127.0.0.1:6379"
```

- 使用常驻模式启动confd

```
nohup /usr/lib/go/mygo/src/github.com/kelseyhightower/confd/bin/confd -interval 10 -config-file /etc/confd/confd.conf > /var/log/confd.log &
```
> -interval 10 # 10秒自动检测，如果nginx发生了变化，则触发配置更新 \
> -config-file /etc/confd/confd.conf # confd读取的配置文件 \
> -nohup 和结尾的 & 后台运行confd \
> -使用代码 \> /var/log/confd.log 将confd输出的内容写入到日志文件中 \

- 往redis中写入租户配置

```
// 项目往redis中写入租户的配置
redis-cli set /nginx/租户编码 '{"id":"租户编码", "ip":"127.0.0.1:8888"}'
```

- confd识别后自动生成配置以及重启nginx

```
// 识别后的更新信息
...
INFO /etc/nginx/sites-enabled/p04 has md5sum cb25ec66f8ea8804b7ecc38f75587b86 should be bc2917f844417fa2eb288e503e00fe7a
INFO Target config /etc/nginx/sites-enabled/p04 out of sync
INFO Target config /etc/nginx/sites-enabled/p04 has been updated
...
```

## 参考资料
dalao的博客：http://www.jianshu.com/p/bc85a54f98ff
