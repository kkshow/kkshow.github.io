---
title: hexo + github 搭建免费个人博客
date: 2018-01-31 15:45:06
tags: [hexo,github,blog]
---
## 什么是Hexo？
Hexo是一款基于nodejs开发的轻量级博客，不需要数据库就可以直接跑起来的博客。

## Hexo有什么优势？
除了上述的不需要数据库、webserver以外，Hexo还有以下优点：
1. 编译后直接生成静态文件，可以直接访问。
2. 可以和github配合构建免费博客。
3. 文档编写使用md格式，更加方便。
4. 有很多开源主题，可以根据喜好进行选择和替换。

<!-- more -->

## 如何起手
### 安装nodejs 
使用的环境是win10，直接上[官网](https://nodejs.org/en/download/)去下载安装包即可。

安装完后，在cmd输入命令测试是否安装成功。
```
# 执行环境变量 path
$ path
# 输出中包含nodejs 和 npm则表示安装成功
PATH=*;E:\Program Files\nodejs\;C:\Users\111\AppData\Roaming\npm

# 查看 nodejs 版本
$ node -v
# 输出 nodejs 的版本号
v6.11.1

# 查看 npm 的版本
$ npm -v
# 输出 npm 的版本号
3.10.10
```

> 我这里下载的安装包是 node-v6.11.1-x64。\
参考教程是 http://www.runoob.com/nodejs/nodejs-install-setup.html

### 配置淘宝镜像
由于不可描述的原因，国外的源通常都很慢，所以这里安装使用淘宝的源，代码如下：

```
# 安装源的命令
$ npm install -g cnpm --registry=https://registry.npm.taobao.org

# 安装完成后，后续可以使用以下命令来安装模块 
$ cnpm install [name]
```

当然也可以使用命令
```
npm --registry https://registry.npm.taobao.org info underscore 
```

### 安装Hexo
```
npm install -g hexo-cli
```

**到此，Hexo的环境已经创建完毕，可以开始建站了！**

## 开始建站
### 生成站点源码
环境构建好之后，就可以开始来建站了。
```
// 进入建站的文件夹
cd xxxxxxx

// 初始化项目
hexo init

// 初始化插件
cnpm install 

// 构建完成后，目录结构大致如下
.
├─ _config.yml  # 配置文件
├─ package.json # 依赖配置
├─ node_modules # 依赖文件夹
├─ scaffolds    # 模板
├─ public       # 生成的博客代码，构建之后才会出现
├─ source       # 源文件目录
|   ├─ _drafts  # 
|   └─ _posts   # 你编写的文章
└─ themes       # 主题包

// 运行一下，在浏览器访问 localhost:4000 即可看到搭建的博客了。
hexo server
```

### 修改配置文件
好了，接下来我们继续完善我们的博客。

打开_config.yml，在这里配置一下基本信息。
```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: 网站标题
subtitle: 网站副标题
description: 网站描述，用于SEO
author: 作者名称
language: 语言包
timezone: 时区，一般填 Asia/Shanghai

...

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: 主题，默认是 landscape，如果下载了别的主题，修改即可

// 部署配置，需要专门学习，这里简单配置一下
# Deployment 
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git # 类型，这里用git
  repo: https://github.com/kkshow/kkshow.github.io # github路径
  branch: master # 这里配置的主干
# name: github账号，这个会被 git config 里面的配置覆盖
# email: github账号所使用的邮箱，这个会被 git config 里面的配置覆盖
```

修改完成后，重启Hexo就可以开到你修改的内容了。

## 编写文章
// 首先生成一篇文章
```
hexo n "文章名"

// 然后可以在 项目/source/_posts 中看到你新建的文件
.
├─ source       # 源文件目录
|   └─ _posts   # 
|          └─ 文章名.md

```
然后就可以开始你的创作~~表演~~了。

## tips 
### 文章里面有多个 tag
```
tags: [hexo,github,blog]
```

### 部署后github上显示的账号不对
这个很有可能是你本地环境中设置了name和email两个属性，并且和你github上的不一直导致。
具体可以查询一下，如果对不上，就重新设置一下。
```
git config -l

git config --global user.email "你的邮箱"
git config --global user.name "你的账号"
```
设置完后再次提交，就会正确显示了。

## 常用命令
初始化站点
```
hexo init 
```

启动站点
```
hexo
```

生成博客文件
```
hexo g
```

清空生成的静态文件
```
hexo clean
```

部署 - 需要配合别的插件
```
hexo d
```

## 参考文档
中文文档地址 https://hexo.io/zh-cn/docs/deployment.html \
hexo-deployer-git https://github.com/hexojs/hexo-deployer-git \
高手的博客 https://zhuanlan.zhihu.com/p/26625249