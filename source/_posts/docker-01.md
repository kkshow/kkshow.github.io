---
title: Docker中经常用到的命令
date: 2018-01-23 14:04:49
tags: docker
---

## 前言
Docker是一种容器平台技术，在容器技术的基础上进行了扩展优化，使容器能够拥有虚拟机的特性而又比虚拟机更快捷、轻便。

总的来说docker绝对是时下非常值得学习的技术之一。

## docker 安装
1、docker的安装非常简单，只需要执行官方提供的脚本就可以了。
```
// 官方脚本安装
wget -qO- https://get.docker.com/ | sh

// 阿里云脚本安装
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

// daocloud脚本安装
curl -sSL https://get.daocloud.io/docker | sh
```
> 当然由于墙的问题，执行这个脚本有可能会异常缓慢，具体的解决方法就是更换linux源即可。

2、当要以非root用户可以直接运行docker时，需要执行以下命令：
```
sudo usermod -aG docker 账户名
```
否则系统会提示以下报错：
```
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?
```

3、配置docker的镜像源
```
// 修改配置文件 /etc/docker/daemon.json
vim /etc/docker/daemon.json

// 加入下面的内容：
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}

// 然后重启docker服务即可
```
> 由于不可描述的原因，docker的源下载非常慢，这里使用DaoCloud的源，详见 https://www.docker-cn.com/registry-mirror

<!-- more -->

## 镜像运行命令
镜像运行的命令比较复杂，这里把常用的几个命令放这里，当作案例。

1、运行busybox
```
// -it 终端交互运行
docker run -it busybox
```

2、运行nginx
```
// -d 守护进程运行
// -p 端口映射
// -v 文件路径映射
// -- mount type=bind,source=/var/www/html,target=/usr/share/nginx/html,readonly 
docker run -d -p 80:80 -v /var/www/html:/usr/share/nginx/html nginx:v2

// 另一种方式
// -- mount 和上面一样，也是用于文件映射，但是有不同的就是如果镜像里面的文件不存在，
// 则生成容器的时候回报错。
// readonly 参数可以不传，如果传了，表示对应容器内的文件只读。
docker run -d -p 80:80 \ 
    --mount type=bind,source=/var/www/html,target=/usr/share/nginx/html,readonly \ 
    nginx:v2
    
// 如果不想指定web容器的端口，则可以使用 -P（注意这里是大写） 来随机映射
docker run -d -P nginx:v2
```

3、启用一个ubuntu终端
```
// --rm 容器退出后自动删除
// --restart=always 自动重启
// --name 容器名称 命名容器
docker run -it --rm --restart=always --name myubuntu ubuntu:latest
```

4、容器互联

> 通过创建一个docker网络，让两个容器运行在同一个网络中。

```
1) 创建一个docker网络
docker network create -d bridge my-net

2) 运行一个容器并连接到新建的 my-net 网络
docker run -it --rm --name busybox1 --network my-net busybox sh

3) 打开新的终端，再运行一个容器并加入到 my-net 网络
docker run -it --rm --name busybox2 --network my-net busybox sh

4) 在 busybox1 容器输入以下命令
/ # ping busybox2
PING busybox2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.072 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.118 ms

// 可以看到在 busybox1 中是能请求到 busybox2 的。
// 同样，在 busybox 中也能请求到 busybox1

```

## 服务命令
1、docker的启动/重启/关闭
```
service docker start/restart/stop
```

2、安装Docker管理工具
```
docker run -d -p 9001:9000 -v "/var/run/docker.sock:/var/run/docker.sock" portainer/portainer
```
> portainer 是一款易用的docker图形管理工具，地址如下：https://github.com/portainer/portainer

## 镜像命令
1、查询docker镜像
```
docker images
```

2、获取镜像
```
docker pull (镜像下载地址/镜像名称)
```

3、删除镜像
```
// 通过ID删除
docker rmi (镜像ID)

// 通过tag删除
docker rmi (tag)

// 批量删除
docker rmi $(docker images | grep "模糊匹配内容" | awk '{print $3}') 
```

4、转移镜像
```
// 保存镜像
docker save compose01_web(镜像名) > t.tar(目标文件)

// 读取镜像
docker load --input ./t.tar(读取的文件)

// 也可以一步到位，直接通过ssh发送到其他机器
docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
```
> docker可以建立自有镜像仓库，但是一般来说我们还是直接转移会方便一些。

5、安装私有仓库
```
// 获取仓库的镜像
sudo docker pull registry

// 启动镜像，将镜像的存放地址映射为非临时目录
sudo docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry

// 启动完成后，如果我们直接push镜像，会出现如下提示：
Error response from daemon: Get https://10.10.239.222:5000/v1/_ping: http: server gave HTTP response to HTTPS client

// 这是由于新版本的docker私有库要求https传输镜像，所以要进行一些设置，让我们能够正常上传镜像。

// 操作步骤如下：
// 打开配置文件 /etc/docker/daemon.json，加入json格式的内容
{
  "其他配置": "其他配置", // 这里是演示，如果已经有其他配置，共用一个json
  "insecure-registries": ["10.10.239.222:5000"] // 不要求使用ssl的库以及端口。
}

// 重启docker即可
service docker restart
```

> 此外：不同版本的linux的设置方式可能有点不同，具体可以参考这个文档 https://yeasy.gitbooks.io/docker_practice/content/repository/registry.html 
如果需要配置有认证的registry，参考这个文档：https://yeasy.gitbooks.io/docker_practice/content/repository/registry_auth.html

参考文档：
- http://blog.csdn.net/wangtaoking1/article/details/44180901
- http://www.jianshu.com/p/00ac18fce367
- https://my.oschina.net/yyflyons/blog/656278

6、上传镜像
```
// 上传镜像
docker pull
```

7、查询镜像 - 这个命令用于查询私有的镜像库
```
// 查询镜像中存在的分类
curl -X GET http://172.16.9.30:5000/v2/_catalog

// 返回
{"repositories":["busybox"]}

// 查询镜像的tag
curl -X GET http://172.16.9.30:5000/v2/busybox/tags/list

// 返回
{"name":"busybox","tags":["latest"]}
```
> 具体的API查询地址：https://docs.docker.com/registry/spec/api/

## 容器命令
1、查询当前容器状态
```
docker ps -a
```
> 这里可以不带 -a ，不带的情况下只查询当前在运行的容器。

2、停止容器运行
```
docker stop (容器ID)
```

3、删除容器
```
docker rm (容器ID)

// 批量删除
docker rm $(docker ps -qa)

// 批量删除
docker ps -qa | xargs -n 1 docker rm
```

4、进入一个已有容器
```
$ docker exec -it webserver bash
```

5、转移容器
```
// 导出容器
docker export

// 导入容器
docker import
```

6、清除所有未运行状态的容器
```
docker container prune
```

7、设置容器的dns
```
// 进入docker的配置文件
cd /etc/docker/daemon.json

// 增加下面的内容，然后重启docker服务
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}

// 后续的容器dns都会从读取以上配置的dns
docker run -it --rm ubuntu cat etc/resolv.conf

// 输出
nameserver 114.114.114.114
nameserver 8.8.8.8
```

## 参考资料
- http://www.runoob.com/docker/docker-tutorial.html
- https://yeasy.gitbooks.io/docker_practice/content/
- https://github.com/docker-library/docs
