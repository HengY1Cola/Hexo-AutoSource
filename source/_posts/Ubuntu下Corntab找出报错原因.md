---
title: Ubuntu下Corntab找出报错原因
categories: 捣鼓分享
keywords: Corntab
top_img: /img/1225403.webp
cover: /img/2587493.webp
abbrlink: 2c7b2295
date: 2022-02-27 12:22:19
---

##  起因：

事情的起因是：我的自动打卡在我加入了`Firefox拓展`之后开始失效了

我手动执行完完全全Ok的，一到自动运行就GG了

然后我设置每分钟运行一次观察CPU状态发现`Firefox`会启动但是几秒后就没了

好家伙，去找日志啥都没有。经过我的捣鼓直接放答案。

##  打开Crontab日志

```bash
$ sudo vi /etc/rsyslog.d/50-default.conf
# 把cron.*前面的注释去掉
:wq

$ sudo service rsyslog restart
$ cd /var/log
$ cat cron.log
```

到这里只能查看到执行日志，也就是说看到执行状况而看不到报错

##  安装postfix

我想起了在Linux下每次执行完毕后会有个打印说的是到`/var/mail`，那里会详细说明执行情况

```bash
$ sudo apt-get install postfix
# 会进入一个设置界面
# 全部默认然后全部选择OK/SELECT啥的
$ crontab -e 
# MAILTO=example@gmail.com 写入文件
# 等待执行
$ cd /var/mail
$ cat ubuntu
```

到这里查看对应用户的文件就能详细看到执行情况了

##  结果

我的问题是在crontab下执行，路径文件找不到，我直接cp过去就成功了。

花了快一个小时捣鼓😭