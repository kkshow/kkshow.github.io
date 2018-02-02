---
title: Hexo 加入流量统计插件
date: 2018-02-01 14:06:29
tags: hexo
---

## 前言
在本章中，我们来讲解如何在你的博客中加入流量统计插件。

## 加入流量统计插件
Hexo生成的博客本身是一个静态网站，所以它是没有流量统计能力的，为了增加这一特性，我们需要结合第三方统计平台来实现。

<!-- more -->

常用的有统计平台有下面的几家：
1. Google Analytics（分析） https://www.google.com/intl/zh-CN/analytics/
2. 腾讯分析 http://ta.qq.com/
3. 百度统计 https://tongji.baidu.com/web/welcome/login

鉴于谷歌一直是被墙的状态，我们这里使用腾讯分析来讲解。

<!-- more -->

### 注册你的网站
进入统计平台后，要注册你的站点，用于统计这个站点的流量。

1. 进入你的站点列表
![image](http://p3gt0myn4.bkt.clouddn.com/hexo-01-01.png)

2. 创建一个网站
![image](http://p3gt0myn4.bkt.clouddn.com/hexo-01-02.png)

3. 填入你的域名地址
![image](http://p3gt0myn4.bkt.clouddn.com/hexo-01-03.png)

4. 添加完成后，获取统计代码
![image](http://p3gt0myn4.bkt.clouddn.com/hexo-01-04.png)

5. 拿到这串代码后，就可以改造你的网站了
![image](http://p3gt0myn4.bkt.clouddn.com/hexo-01-05.png)

### 在Hexo中添加js代码
由于Hexo是通过模板文件生成静态页面的，所以我们在添加的时候直接去查找生成的模板文件修改即可。

**不同的主题修改的文件不尽一致，但是基本上大同小异，学会一个之后其他主题也就很容易修改了。这里是用 NexT 这个主题来讲解。**

```
// 打开主题下的 siderbar.swig文件
vim 项目路径/themes/next/layout/_custom/siderbar.swig

// 添加分析网站给你的监控代码，例如：
<script type="text/javascript" src="http://tajs.qq.com/stats?sId=你的sid" charset="UTF-8"></script>

// 重新部署Hexo
hexo clean
hexo g
hexo d

```
然后去你的网站访问就可以看到流量信息了

### 加入流量统计插件 For NexT
不得不说 NexT 这个主题做的很齐全，该有的东西基本上都可以通过配置做出来。

上面一段所说的统计分析插件，在NexT中已经集成了，我们进入主题的配置文件直接写入自己的sId即可。

操作方式如下：
```
// 打开站点的主题配置文件
vim 项目路径/themes/next/_config.yml

// 找到 tencent_analytics 一项，设置你的 sId
tencent_analytics: 你的sid

// 如果使用 百度统计，则用这一项
# Baidu Analytics ID
baidu_analytics: 你的sid

// 如果使用 google，则用这一项
# Google Analytics
google_analytics: 你的sid
```
好了，刷新之后就同样可以看到流量信息了，就是这么简单。

## 参考资料
NexT主题官方文档 http://theme-next.iissnan.com/getting-started.html \
NexT github https://github.com/iissnan/hexo-theme-next

