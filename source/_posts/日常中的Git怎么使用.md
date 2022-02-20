---
title: 日常中的Git怎么使用
categories: 开发
keywords: Git
top_img: /img/1419217.png
cover: /img/gitlogo.png
abbrlink: e996634a
date: 2022-02-20 22:03:56
---
---

## 日常中的Git怎么使用

`git`是真的香呀，主要是好管理，用着方便。

我的git学习路程就是：学了 -> desktop点点点 -> 三剑客 -> 敲击命令

说白就是不知道各种情况不知道怎么办？

然后今天就简单总结下几种情况。

##  情景再现：

1. 我创建了远程仓库，我本地已经有代码了，如何将本地的加入到仓库里面
2. 为别人仓库提交Pr，参与别人的项目。
3. 我创建了仓库，拉下来自己改了，别人在我之前上传了代码，我怎么上传
4. 创建了主线分支与支线分支，支线开发合并到主线
5. 一些基础命令，账号设置。

{% note green 'fas fa-bullhorn' flat %}

在这里我准备了2个账号来模拟下。分别用台式和笔记本设置了全局账号

至于终端怎么加速 点击右侧👉 ：https://hengy1.top/article/567e1422.html

{% endnote %}

##  情境一：

> 我创建了远程仓库，我本地已经有代码了，如何将本地的加入到仓库里面

一般创建好了会有`LICENSE`与`README`，然后本地已经有了文件。

<img src="/img/mics/image-20220220222648642.webp" alt="远程仓库" style="zoom:67%;" />

<img src="/img/mics/image-20220220222956379.webp" alt="本地存在文件了" style="zoom:67%;" />

下面步骤与注释我都写在里面了。

```bash
# 初始化init环境
$ git init
# 切换到main分支 因为现在git的默认分支是main了
$ git checkout -b main
# 添加远程地址 originName自己取
$ git remote add originName git@github.com:HengY1Sky/gitTry.git
# 查看下远程
$ git remote -v
# 把远程的拿来下然后与本地的合到一起
$ git pull originName main
# 查看效果
$ ls -l
# 添加到工作区
$ git add .
$ git commit -m "init"
# 提交 
$ git push --set-upstream originName main
```

<img src="/img/mics/image-20220220224029970.webp" alt="" style="zoom:67%;" />

##  情境二：

> 为别人仓库提交Pr，参与别人的项目。

这个是分情况的：

如果你有这个仓库权限，即你是成员：直接拉下来提交就行了

如果是别人的仓库，你得先自己fork个，拥有权限再动。

<img src="/img/mics/image-20220220224632928.webp" alt="" style="zoom:67%;" />

```bash
# 从自己fork的拿下来将文件命名为git
$ git clone https://github.com/HengY1Sky/gitTest.git git 
$ cd git
# 创建分支
# 为什么加分支？因为主分支防止别人仓库更新了
$ git checkout -b change
# 开始自己加东西
$ ....
$ git add .
$ git commit -m "add hello.txt"
# 提交到远程
$ git push --set-upstream origin change
```

<img src="/img/mics/image-20220220225547672.webp" alt="" style="zoom:67%;" />

<img src="/img/mics/image-20220220225637580.webp" alt="" style="zoom:67%;" />

`你的仓库：分支` ->`对方的仓库/分支`，然后加上你的描述，对方接受会给你发邮件，你们可以交流交流了。

```bash
# 当然别人仓库可以更新的
$ git branch
$ git checkout main
$ git remote add upstream https://github.com/HengY1Sky/gitTest.git
# 将别人的最新分支更新到你的本地main分支上
$ git pull upstream main
```

##  情境三：

> 我创建了仓库，拉下来自己改了，别人在我之前上传了代码，我怎么上传

这里我直接在Github上改一下相当于别人改了

<img src="/img/mics/image-20220220230237186.webp" alt="" style="zoom:67%;" />

然后我在本地不知道，继续更改

```bash
$ git add .
$ git commit -m "make change"
$ git push # 会被拒绝
```

<img src="/img/mics/image-20220220230441903.webp" alt="" style="zoom:67%;" />

```bash
# 拉取并合并
$ git pull
# 手动解决冲突
$ vim index.txt
```

<img src="/img/mics/image-20220220230555258.webp" alt="" style="zoom:67%;" />

<img src="/img/mics/image-20220220230613363.webp" alt="" style="zoom:67%;" />

留下你想要的，其他删除干净。

```bash
$ git add .
$ git commit -m "final"
$ git push # 搞定
```

##  情境四：

> 创建了主线分支与支线分支，支线开发合并到主线

这个可以两种: 

1. 本地两个分支，在主分支合并好了，然后更新远端主分支

```bash
# 查看本地分支
$ git branch 
# 创建副分支
$ git checkout -b other
# 删除主分支内容加上自己新的内通
$ vim index.txt
$ git add .
$ git commit -m "new"
# 切换回主分支
$ git checkout main
# 合并分支
$ git merge other
$ git push
```

2. 两个分支都上传到远端，然后在github上图形化合并到主线(通常是团队)

```bash
$ git checkout ohter
$ vim index.txt
$ git add .
$ git commit -m "other"
$ git push --set-upstream originName other
```

<img src="/img/mics/image-20220220232640656.webp" alt="" style="zoom:67%;" />

<img src="/img/mics/image-20220220232704906.webp" alt="" style="zoom:67%;" />

正常提交，手动解决冲突保存想要的

<img src="/img/mics/image-20220220232741088.webp" alt="" style="zoom:67%;" />

<img src="/img/mics/image-20220220232816828.webp" alt="手动解决冲突" style="zoom:67%;" />

然后一路点点点就好了，通常主线是技术组长，组员在分支提交，组长审核合并到主线。

##  基础命令，账号设置

```bash
$ git config --global user.name ''
$ git config --global user.email ''
$ git config --list --global  # 提交的时候会知道是你提交的以及会联系你

$ git status # 查看状况
$ git log # 查看日志
$ git tag -a v1.2 xxx -m "my tag" # 给xxx打上标签与描述

$ git branch -v # 查看分支
$ git checkout -b temp d96e27d54e4aaa79e0ccc76e1023b3d489a9aced # 创建一个叫做temp的从d96e27d54e开始的分支
$ git checkout master # 切换分支

$ git diff xxx xxx # 两者进行比较
$ git diff HEAD HEAD^1^1 # 与爷爷进行比较
$ git diff temp master -- readme.md # 当前temp分支与master进行比较

# 这个人可以根据新的分支自己在本地创建分支来编辑-》 更新 -》 上传 -》 提交
$ git fetch gitTest # 将别人提交的分支也拉下来
$ git fetch gitTest # 在编辑完后如果还有人编辑了而你即将提交了就要fetch
$ git merge gitTest/master # 没有冲突智能合并/有冲突要改
$ git push
```

现在没有账号登陆了然后要配置公钥 https://docs.github.com/cn

网上教程一大堆，随便搜搜