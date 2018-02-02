---
title: vue开发环境配置
date: 2017-07-30 22:22:00
tags: vue
---

## 0、环境安装

vue虽然是独立的js框架，但是部署的时候需要用到npm，而npm又需要用到nodejs，所以需要进行环境的搭建和配置。

### 1)nodejs 安装
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

### 2)淘宝镜像配置
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

<!-- more -->

## 1、安装vue
需要注意的是，npm install 命令会把你的项目安装在当前目录，所以一般安装的时候要进入到nodejs路径下。

例如我的路径是：E:\Program Files\nodejs

另外这里有一份设置npm安装路径的说明，但是好像没有什么效果。
http://www.imooc.com/article/12487?block_id=tuijian_wz

### 1)npm 安装 vue
```
# 最新稳定版
$ cnpm install vue
```

### 2)Vue.js 提供一个官方命令行工具，可用于快速搭建大型单页应用。
```
# 全局安装 vue-cli
$ cnpm install --global vue-cli
# 创建一个基于 webpack 模板的新项目
$ vue init webpack my-project
# 这里需要进行一些配置，默认回车即可
This will install Vue 2.x version of the template.

For Vue 1.x use: vue init webpack#1.0 my-project

? Project name my-project
? Project description A Vue.js project
? Author runoob <test@runoob.com>
? Vue build standalone
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? Yes
? Setup e2e tests with Nightwatch? Yes

   vue-cli · Generated "my-project".

   To get started:
   
     cd my-project
     npm install
     npm run dev
   
   Documentation can be found at https://vuejs-templates.github.io/webpack
```

### 3)进入项目，安装并运行
```
$ cd my-project
$ cnpm install
$ cnpm run dev
    DONE Compiled successfully in 4388ms

> Listening at http://localhost:8080
```
{% note warning %} 
如果系统提示缺少模块，可以通过 cnpm install [模块名] 来直接安装相关模块。 
{% endnote %}
