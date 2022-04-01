---
title: 用爱发电的小服务
date: 2022-01-26 21:07:33
top_img: /img/1261652.webp
aside: false
comments: false
---

## 公告
**接口路由**： https://api.hengy1.top
所有的接口都在这个下面了～

{% note red 'fas fa-bullhorn' flat %}

因为现在主要是Golang后端，在冲大厂ing
前端的话我准备大二暑假陆陆续续写出来～
所以大家想调用服务器直接怼接口就好了。
因为是用爱发电的，尽量别攻击，没啥意思。

{% endnote %}

##  谈谈安全
1. 接口走的Cloudfare，~~你随便Ping下就能发现~~
2. 你打DDOS,CC的话基本过不来，~~别浪费钱~~
3. 接口下面是有Waf的，~~别注入啦～~~ 
4. 如果你真能打进来～ 联系我QAQ
5. 相互多多交流吧，共勉

##  服务一：宿舍水费预警/充值系统
1. 每天的总请求量为10000
2. 每个IP总请求为50
3. 查询后结果缓存时效为6小时
4. 邮件激活后开始工作
5. 邮件取消/申请为同一接口
6. 激活邮件发送间隔存在一定限制
7. 充值金额不超过30元，仅为紧急充值
8. 充值服务每20秒提供一次有效服务

点击查看文档：{% btn '/service/JnuWater.html',这里啦～,far fa-hand-point-right,green outline %}

## 敬请期待