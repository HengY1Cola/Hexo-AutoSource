---
title: Mac优化 zsh 的启动速度(M1)
categories: 分享
keywords: MacOs
top_img: /img/123653.webp
cover: /img/123640.webp
abbrlink: d155260a
date: 2022-05-23 18:50:12
---

##  起因：

不知道何时何地我更新了个啥；我的终端开启就是十分的卡

卡到什么程度？卡到新开一个快20秒了；已经够离谱了吧，我就想来解决下这个问题

下面几乎没有图图了，因为是复盘，当时没截图，请耐心看完应该能解决

##  经过：

我直接谷歌：Mac终端开启过慢，然后开始看内容；发现就几个点。

1. 首先是以[Mac终端启动很慢解决方案](https://blog.csdn.net/Bobby_world/article/details/79673790)各种转发出现的删除日志文件就行了。

好嘛，我删除，发现并没有什么niao用，其实一般看这种帖子就不行，还是忍不住试了试

2. 发现[知乎好文---优化 zsh 的启动速度](https://zhuanlan.zhihu.com/p/464117825)

这个写的还是挺全的，并且里面外链内容都挺高质量的；但是我试了这个命令：

```bash
$ /usr/bin/time /bin/zsh -i -c exit
$ /usr/bin/time /bin/bash -i -c exit
```

好的结果也就`real 0.26`当然也就是不符合这类情况；然后我就发现了[知乎解决zsh启动速度慢的优化方法](https://zhuanlan.zhihu.com/p/68303393)

其实吧2篇文章都是一个意思；但是还是跟着做了；有2个坑：

`brew install qcachegrind --with-graphviz` 改为 `brew install qcachegrind`

`opam install ocamlfind` arm结构不对；到这里其实就气的想重置电脑了，但是换了个思路

3. 发现[Zsh 加载速度优化](https://wxnacy.com/2019/04/04/zsh-speed-optimization/)

发现了我应该直接根据zsh启动生命周期开看看会加载什么；也就是说完整的文件加载过程

前面只是涉及了某些部分文件；简单看了看那么就是：

**当打开新的 Terminal 时，配置加载顺序为 ~/.zshenv ~/.zprofile ~/.zshrc ~/.zlogin**

**执行 zsh 时，配置加载顺序为 ~/.zshenv ~/.zshrc**

成功，我在`~/.zprofile`发现了一条brew命令重复了1000多条！删除重新加载，1-2秒；完美！

##  总结：

可以从 [Zsh 配置文件加载顺序](https://wxnacy.com/2018/10/08/zsh-startup-files/) 从Zsh 配置文件的生命周期下手，每个文件夹看看。