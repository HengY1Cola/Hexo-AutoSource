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

因为现在主要是Golang后端。
前端的话我准备大二暑假陆陆续续写出来～
所以大家想调用服务器直接怼接口就好了。
因为是用爱发电的，尽量别攻击，没啥意思。

{% endnote %}

##  服务一：宿舍水费预警/充值系统
1. 每天的总请求量为1000
2. 每个IP总请求为50
3. QPS设置在50（学校的爱惜点）
4. 查询后余额结果缓存时效为1小时
5. 查询后消费记录缓存时效为1天
6. 邮件激活后开始工作
7. 邮件取消/申请为同一接口
8. 激活邮件发送间隔存在一定限制

点击查看文档：{% btn '/service/JnuWater.html',这里啦～,far fa-hand-point-right,green outline %}

##  服务二：学校教务服务
1. 每天的总请求量为1000
2. 每个IP总请求为50
3. QPS设置在50（学校的爱惜点）
4. 本系统不会保存您的账号密码，若建议勿使用
5. 请务必保管好自己的账号密码，作者不负责
6. 账号密码不匹配3次后会拒绝提供服务防止账号锁死
7. 为保护下游服务器防止解析失败，限每5秒调用一次

点击查看文档：{% btn '/service/school.html',这里啦～,far fa-hand-point-right,green outline %}

## 敬请期待