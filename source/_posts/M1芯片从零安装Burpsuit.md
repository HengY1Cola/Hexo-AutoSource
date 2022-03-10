---
title: M1芯片从零安装Burpsuit
categories: 分享
keywords: Burpsuit
top_img: /img/3278208.webp
cover: /img/6367119.webp
abbrlink: 385adb75
date: 2022-03-10 14:28:25
---

# M1芯片从零安装Burpsuit

##  前提事要

就在几天前，我正在写面试总结的时候我的电脑突然黑屏了。当时去搜了一下，可能是ipad上的保护套上的磁铁导致休眠。结果怎么按都没有反应，然后我回到寝室接上充电器，除了发热任然黑屏。然后到维修店寄希望于是电池出现了问题，然后还是不行。拆开之后接上充电器摸哪里发热，好家伙直接CPU，直接没了。导致我大部分的**md文件**全部丢失。第二天被迫搞了台m1pro芯片回来。暂时用着还行。

{% note red 'fas fa-bullhorn' flat %}

这是我最不情愿的换电脑的一次，现在已经买了AC了！！

{% endnote %}


##  再次光度国光的网站

我记得上次也是在国光的帖子上找到了破解的思路，现在再次看看，发现新的姿势还是不错的。

注意⚠️：以下的操作是在brew上建立的。如果你没有🪜或者不知道如何终端走代理。请访问[这个文章](https://hengy1.top/article/567e1422.html)

##  首先是安装对应的Java版本

我最开始以为是下载的burp里面自己带对应的版本。但是我根据操作发现根本不行。

在终端打开代理的情况下。

```bash
$ brew tap AdoptOpenJDK/openjdk # 添加仓库
$ brew search openjdk # 查询可用的 JDK 版本
$ brew install --cask adoptopenjdk8
$ brew install --cask adoptopenjdk13
```

然后加入对应版本的Java环境变量，这里要设置为13版本

```bash
$ vim ~/.bash_profile
```

```
# add Java
export JAVA_HOME_8=$(/usr/libexec/java_home -v1.8)
export JAVA_HOME_13=$(/usr/libexec/java_home -v13)
# export JAVA_HOME=$JAVA_HOME_8
export JAVA_HOME=$JAVA_HOME_13
```

```bash
$ source ~/.bash_profile
$ java -version
```

##  下载对应版本

国光最新的姿势就是正常下载，然后注册机正常绕过就好了。

官方下载地址：https://portswigger.net/burp/releases

<img src="/img/mics/202203101443781.webp" alt="" style="zoom:80%;" />

下载好了直接放到应用程序里面，免得后面路径不对。

然后下载注册机器与汉化包：https://sqlsec.lanzoux.com/im9eUj08a1i

然后下载2-x的通用版绕过 https://github.com/TrojanAZhen/BurpSuitePro-2.1

然后放到指定位置，当然跟着国光的2022年版本命令直接做就好了。给个效果图。

<img src="/img/mics/202203101443331.webp" alt="" style="zoom:80%;" />

就命令都不用变的，注册机使用网上到处是，我就不贴了～

最后实现的效果就是：你能正常打开了但是是英文版本的，如果你不常用完全没有使用的欲望。

##  汉化使用

如果再多往下看看帖子会发现汉化的办法。即修改`vmoptions.txt`

<img src="/img/mics/202203101446022.webp" alt="" style="zoom:80%;" />

```
# Enter one VM parameter per line
# For example, to adjust the maximum memory usage to 512 MB, uncomment the following line:
# -Xmx512m
# To include another file, uncomment the following line:
# -include-options [path to other .vmoption file]

# 注释
# -XX:MaxRAMPercentage=50
# --illegal-access=permit
# -include-options user.vmoptions
# -Dfile.encoding=utf-8
# -noverify
# -javaagent:burp-loader-x-Ai.jar
#  -Xmx2048m

# 新的启动方式
-noverify
-javaagent:BurpSuiteLoader.jar
-javaagent:BurpSuiteCn.jar
-Dfile.encoding=utf-8
-XX:MaxRAMPercentage=50
-include-options user.vmoptions
```

##  加入CA证书

上面打开就是汉化版本了，但是你抓https是不行的，得加入证书。

<img src="/img/mics/202203101448870.webp" alt="" style="zoom:50%;" />

手动保存为`burp.crt`文件，然后到任意浏览器里面设置=>搜索证书=>默认点点点=>把证书拖入进来

<img src="/img/mics/202203101449851.webp" alt="" style="zoom:80%;" />

然后右键=> 显示简介=> 信任 => 使用此证书时 => 始终信任 => OK