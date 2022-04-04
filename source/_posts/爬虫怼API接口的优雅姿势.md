---
title: 爬虫怼API接口的优雅姿势
categories: 分享
keywords: 爬虫
top_img: /img/1313483.webp
cover: /img/2312312.webp
abbrlink: cb732448
date: 2022-04-04 17:35:06
---

##  前言：

爬虫的知识点很多，大量的网络知识编程知识。而对于业余的来说最多的是：

对于某些接口，伪造请求拿想要的数据。最近“一不小心”拿下了某个系统的部分接口

就来“水”一篇文章，分享下我感觉效率很高的姿势。

##  工具准备：

- 浏览器的F12

- 爬虫工具网站
- Burpsuit

##  开始寻找目标：

进入对应的页面，找到自己想要的数据，然后打开F12 => 网络

<img src="https://pic.hengy1.top/typora/202204041728254.png" alt="image-20220404171619551" width=66%; />

注意是搜索而不是过滤然后回车就会发现在哪个接口下

<img src="https://pic.hengy1.top/typora/202204041728606.png" alt="image-20220404171745062" width=66%; />

载荷就是Payload也就是参数 预览就是回来的数据

##  模拟请求：

<img src="https://pic.hengy1.top/typora/202204041728484.png" alt="image-20220404171850353" width=66%; />

然后到工具网站：https://spidertools.cn/#/curl2Request

<img src="https://pic.hengy1.top/typora/202204041730637.png" alt="image-20220404172015057" width=66%; />

直接把Pthon代码贴出来了，到这里可以在session/cookie没有失效的前提下进行测试。确定哪些参数是必须的

##  倒推找cookie

你会发现有的cookie是怎么来的，那么就需要Burpsuit的帮忙了

1. 清除目标网站的已经又的cookie
2. 浏览器走8080端口
3. 设置好Burp但不用拦截

<img src="https://pic.hengy1.top/typora/202204041728808.png" alt="image-20220404172404786" width=66%; />

然后你就正常操作一遍，然后到Burp里面进行分析

<img src="https://pic.hengy1.top/typora/202204041729224.png" alt="image-20220404172614689" width=66%; />

*很重要👉 注意几个点 1. 按照时间排序 2.注意cookie设置的时机

然后**右键丢给重发器可以模拟请求猜测字段**，然后根据cookie一步一步倒退回去

基本搞定～