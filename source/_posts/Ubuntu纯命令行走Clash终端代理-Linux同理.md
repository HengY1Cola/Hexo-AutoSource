---
title: Ubuntu纯命令行走Clash终端代理(Linux同理)
categories: 开发
keywords: Js逆向
top_img: /img/7978943.webp
cover: /img/clashLogo.png
abbrlink: 3dadfa74
date: 2022-02-22 19:48:30
---

## Ubuntu纯命令行走Clash代理(Linux同理)

{% note green 'fas fa-bullhorn' flat %}

广告 ：https://hengy1.top/article/567e1422.html

{% endnote %}

## 布置环境 

到 https://github.com/Dreamacro/clash/releases 找到最新版本

> 这里吐槽下：可能一直拉不下来，手动下载然后ftp传上来

```bash
# 创建环境
$ mkdir clash
$ cd clash
# 拉取下来
$ wget https://github.com/Dreamacro/clash/releases/download/v1.9.0/clash-linux-amd64-v1.9.0.gz
$ gzip -d clash-linux-amd64-v1.9.0.gz
$ mv clash-linux-amd64-v1.9.0.gz clash
# 这里clash为二进制可以运行文件了
$ chomd 755 clash
```

##  传入配置

```bash
# 生成默认配置
$ ./clash -d .  # 默认配置
# 拉取机场配置
$ wget xxxx # 要么拉下来 要么自己拷贝上来
# 注意： 名称为：config.yaml
$ ./clash -d . # 就开始加载你的配置了
```

<img src="/img/mics/image-20220222194004626.webp" alt="image-20220222194004626" style="zoom:85%;" />

##  日常使用

你会发现这个已取消就失效了，下面我们添加到后台(建议为root用户)

1. `vim ~/..bashrc  `开始准备命令并把下面的写进入

```bash
alias clash_start="screen -S clash /home/xxx/clash/clash -d /home/xxx/clash/"
alias clash_stop="pkill clash"

alias proxy_on="export https_proxy=127.0.0.1:7890 && export http_proxy=127.0.0.1:7890"
alias proxy_off="unset http_proxy https_proxy"
```

2. `source ~/..bashrc ` 重新载入
3. `clash_start`会进入screen 按 `ctrl + a + d`退出后台
4. `proxy_on`开启代理
5. `curl https://www.youtube.com/`会发现马上返回结果

##  github加速

```bash
$ git config --global http.proxy 'http://127.0.0.1:7890'
$ git config --global https.proxy 'https://127.0.0.1:7890'
```

{% note pink 'fas fa-car-crash' flat %}
恭喜：速度起来了！
科学上网，规范行使。
{% endnote %}