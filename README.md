#  恒HengY1毅

这里是@HengY1博客**源码**地址，很高兴我也是Hexo的一员了。

从购买服务器到申请域名到自己捣鼓Hexo

终于在**2022-02-17**正式开启小破站之旅。

我会努力更新，日志我就丢下面啦！

## 自动部署

参照了B站Up(objtube的卢克儿)的代码实现了自动打包部署到main分支下

达到了更新源文件从而更新main分支，服务器直接拉取main分支最新文件即可

点击右侧进行查看效果👉 https://github.com/HengY1Sky/Hexo-AutoSource/tree/main

部署源码在`.github/workflows/deploy.yml`中, 分享拿走！

##  更新日志

<details>
  <summary>20220224</summary>
  <h3>添加介绍</h3>

  - 更新了服务页面
  - 添加了水费接口
  - 迁移 MAC上RabbitMQ从安装到用GO快速实现搬移
  - 添加Robots.txt
</details>

<details>
  <summary>20220222</summary>
  <h3>添加403 503</h3>

  - 添加了403 503页面
  - 添加文章 Ubuntu纯命令行走Clash终端代理-Linux同理
  - 压缩网站文件
  - 添加文章 Python浅谈多线程 Go实现基础密码加密解密
  - 迁移 树莓派文章
</details>

<details>
  <summary>20220220</summary>
  <h3>修正日期，添加文章</h3>

  - 修正了原文的日期
  - 加入文章 Docker日常究竟要怎么用
  - 更新文章 Docker日常究竟要怎么用
  - 加入打赏功能
  - 加入文章 学习Python高级编程到asyncio并发实践
  - 加入文章 日常中的Git怎么使用
</details>

<details>
  <summary>20220219</summary>
  <h3>加入URL算法</h3>

  - 加入了hexo-abbrlink简化
  - 加入文章 Js逆向练习制造Token与Id
  - 添加百度sitemap
</details>

<details>
  <summary>20220218</summary>
  <h3>加入搜索系统，更新留言板</h3>

  - 加入了本地搜索系统
  - 留言板上BB了两句
  - 添加友链规则
  - 引入[iconfont](https://www.iconfont.cn/)
  - 代码片段更换为等宽字体
  - 上CDN加速以及强制跳转HTTPS
  - 首页图片更新
  - 创建文章 数字取证究竟如何入门
</details>

<details>
  <summary>20220217</summary>
  <h3>初始化博客网站，实现自动化部署</h3>
  <p>今天域名终于审批下来了耶～</br>在原来的基础上开始部署公网系统。</p>

- 添加网站分析统计，使用[谷歌分析](https://www.google.com/analytics/)
- 使用`Twikoo`作为[网站的评价系统](https://twikoo.js.org/quick-start.html#%E4%BA%91%E5%87%BD%E6%95%B0%E9%83%A8%E7%BD%B2)
- 添加了Akismet 反垃圾服务
- 添加了即时的[微信消息推送](https://sct.ftqq.com/)
- 谷歌分析延迟改为[百度分析](https://tongji.baidu.com/sc-web)

</details>