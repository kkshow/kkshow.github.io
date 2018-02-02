---
title: Docker-compose使用详解
date: 2018-01-30 12:16:42
tags: docker
---

## Docker Compose
Docker Compose是官方开源的一个Docker容器编排软件，定位于将多个容器编排在起义行程一组服务。

> 官方手册地址：https://docs.docker.com/compose/overview/

## 关键概念
### 项目
通过compose生成的一组服务我们称之为项目，项目一般由多个容器组成，共享生命周期。

## 安装
```
// 直接通过下载compose的执行文件即可
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

// 下载完成后，进行授权即可使用
sudo chmod +x /usr/local/bin/docker-compose

// 查看一下版本，输出以下内容则说明安装成功
docker-compose version 1.18.0, build 8dd22a9
docker-py version: 2.6.1
CPython version: 2.7.13
OpenSSL version: OpenSSL 1.0.1t  3 May 2016
```

## 使用
### 首先来一个简单的例子
```
// 创建一个新目录
cd /var/local

// 创建compose的目录
mkdir tomcat_compose

// 创建compose的生成文件
touch docker-compose.yml

// 进入docker-compose.yml，写入以下内容
version: '3'
services:
  web:
    image: "tomcat"
    ports:
      - "33338:8080"

// 保存文件后，执行docker-compose up运行案例
docker-compose up

// 然后docker-compose就会开始构建你的项目了
// 构建输出...

// 构建完成后，访问33338端口可以看到输出信息
```
> 可以看的出来，其实docker-compose是另一种方式的docker容器构建方式。

<!-- more -->

### 别人给的一个例子
1、创建一个app.py文件
```
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

2、创建一个Dockerfile文件

```
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]

```

3、创建一个docker-compose.yml文件
```
version: '3' // 注意，这里的3是指docker-compose引擎识别的yml格式，一般用3就可以了
services:

  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - redis

  redis:
    image: "redis:alpine"
```
> 注意,yml文件里面的缩进为2个空格

4、执行docker-compose up构建
```
// 构建输出
```

5、构建完成后，访问 ip:5000，返回 Hello World! 该页面已被访问 N 次。

> 从docker-compose.yml的结构上看出，这里用了两个镜像来实现这组服务，APP中调用了redis的自增命令来实现计数。


** 另外有一个没有说到的点是，compose会把这两个容器放置到一个网络中，所以web端是可以请求到redis的。**
```
// 查询一下生成的两个容器
docker ps | grep compose01

// 输出
ID              ...    NAMES
82731293d9de    ...    compose01_web_1
8d9d8b80de9f    ...    compose01_redis_1

// 查询一下先有docker网络
docker network ls | grep compose01 // 这个compose01是我构建时的目录

// 输出
1f38316baee8    compose01_default    bridge    local

// 查询web的网络
docker inspect compose01_web_1

// 输出
...
    "Networks": {
        "compose01_default": {
...

// 查询redis的网络
docker inspect compose01_redis_1

// 输出
...
    "Networks": {
        "compose01_default": {
...
```
> 由此看出两容器使用的网络相同，验证上面的结论。

### wordpress案例
创建wordpress的目录，然后创建docker-compose.yml文件，运行docker-compose up命令，然后访问就可以看到wordpress的安装页面了。
```
version: "3"
services:

   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data:
```

## 删除项目
在compose的构建目录下直接运行 **docker-compose rm**即可生成的项目

## compose命令详解
docker-compose的支持了docker的大部分命令，其使用方式也大同小异，具体可以看这里。https://yeasy.gitbooks.io/docker_practice/content/compose/commands.html#rm

## compose模板命令详解
这里指的是写在yml里面的命令，详见：https://yeasy.gitbooks.io/docker_practice/content/compose/compose_file.html

> 以上内容后面补完
