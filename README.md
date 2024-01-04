#  恒HengY1毅

这里是@HengY1博客**源码**地址，很高兴我也是Hexo的一员了。

从购买服务器到申请域名到自己捣鼓Hexo

终于在**2022-02-17**正式开启小破站之旅。

我会努力更新，日志我就丢下面啦！

## 自动部署

参照了B站Up(objtube的卢克儿)的代码实现了自动打包部署到main分支下

达到了更新源文件从而更新main分支，服务器直接拉取main分支最新文件即可

点击右侧进行查看效果👉 https://github.com/HengY1Cola/Hexo-AutoSource/tree/main

部署源码在`.github/workflows/deploy.yml`中, 分享拿走！

## Hexo常用命令

https://hexo.io/zh-cn/docs/commands

```bash
# 生成静态文件
$ hexo g

# 跑服务
$ hexo s

# 清除缓存
$ hexo clean
```