---
title: Docker日常究竟要怎么用
categories: 开发
keywords: Docker
top_img: /img/mics/3218562.webp
cover: /img/mics/docker.webp
abbrlink: 86cd1cda
date: 2022-02-19 23:46:05
---

##  Docker日常究竟要怎么用

最近在考C4认证，结果模拟卡在了Docker部署。我真的十分无语😓

我在Windows上面编辑的，踩了好多坑，耗了很多时间。

趁着下周考试前来写一下Docker在日常只能够究竟要怎么用：

{% note green 'fas fa-bullhorn' flat %}

本文首发于博客，观感更好 :) 

我的Docker自学笔记 ：[CSDN博客，求个赞～](https://blog.csdn.net/weixin_51485807/article/details/122517156)

{% endnote %}

##  你能收获什么

下面这4项，除了玩复杂网络的，基本够了。

我把从本地 = > 云上 => 别的地方 都串通了

- DockerFile使用
- Docker上传仓库
- docker-compose的编写与部署
- 常用命令

## 更新

{% note red 'fas fa-bullhorn' flat %}

重新写了下DockerFile与docker-compose的文件
找到了相比较depends_on等待Mysql打通再链接的脚本
这个很好的解决了 {% btn 'https://github.com/Eficode/wait-for',点击查看,far fa-hand-point-right %}

{% endnote %}

##  DockerFile使用

这个是干嘛的？ 简单来说就是你在本地写好了文件，然后写个DockerFile来打造你的专属Docker容器

也就是`Centos`上造个`Ubuntu`的感觉。我拿我的一个项目案例来说下。

首先呢创建一个`Dockerfile`文件，对！名字都是死的，当然你用`-f`也是行的，但是我懒得搞路径

其次呢，类似于 `.gitignore`你也要搞个`.dockerignore`这样就“加速”了，具体原因谷歌下。

好了到现在为止：开始写DockerFile 然后 命令参数到处都是 [随便放个在这里](https://jiajially.gitbooks.io/dockerguide/content/chapter_fastlearn/dockerfile_details.html)

对了你在打包前记得**开启docker服务**哈～

下面是我的目录👇 目标是打包个Flask

<img src="/img/mics/image-20220219231439092.webp" alt="这是我的目录" style="zoom:80%;" />

这里有几个天坑！

1. `COPY` 过去的话**文件**就会自己创建相同名字，**文件夹**的话必须写一遍
2. `PYTHONPATH`必须指定，不然的话就会说找不到Python模块，贼麻烦
3. `RUN`尽量写一起，少建立层数，减小内存
4. `CMD`的写法也要注意⚠️，不然只会执行最后一个

```dockerfile
FROM python:3.9.5-slim
WORKDIR /pythonApp
COPY requirements.txt /pythonApp
COPY utils /pythonApp/utils
COPY src /pythonApp/src
COPY repo /pythonApp/repo
COPY model /pythonApp/model
COPY flaskApp /pythonApp/flaskApp
COPY __init__.py /pythonApp
COPY wait-for /
ENV FLASK_APP=/pythonApp/flaskApp/app.py
ENV PYTHONPATH /pythonApp
EXPOSE 8080
RUN pip install --upgrade pip && \
    pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple && \
    apt-get -q update && \
    apt-get -qy install netcat
```

---

开始部署打造自己的。

```bash
$ docker build -t csdn:v0.2 . # <名字>:<版本> . <= 注意这个点

$ docker image ls # 就能看到了 cfdac878807f 为csdn的

$ docker container run -d -p 8080:8080 cfdac878807f # 后台开启个container

$ docker container exec -it cfdac878807f sh # 交互式进入容器
```

## Docker上传仓库

你打造出了自己的容器，自己在本地用肯定不够，放到云上，别人也能用对吧～

首先到仓库里面创建仓库 https://hub.docker.com/repositories 

<img src="/img/mics/image-20220219233253507.webp" alt="创建效果" style="zoom:50%;" />

```bash
$ docker login # 然后本地登陆下验证下你是你就好了

$ docker tag cfdac878807f <hub-user>/<repo-name>[:<tag>] # 给你将要上传的打上标签

$ docker push <hub-user>/<repo-name>:<tag> # 传到哪个仓库去 不写Tag就是最新的
```

到这里只要有网就能部署你的。

## docker-compose的编写与部署

这个`docker-compose`又是干嘛的？ 这个是你给别人的，别人有你这个

就能马上搭建出你想给他的环境。类似于sh脚本，很快就在另外机器上部署相同环境。

名字也是写死的 `docker-compose.yml`

```yaml
version: '2'
services:
  mysql:
    image: mysql:5.7.31
    networks:
      small:
        ipv4_address: 172.19.0.3
    environment:
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - "3306:3306"
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
  web:
    image: hengyisky/daily:v0.1
    networks:
      small:
        ipv4_address: 172.19.0.4
    ports:
     - "8080:8080"
    links:
     - mysql:mysql
    depends_on:
     - mysql
    command: sh -c '/wait-for mysql:3306 -- python /pythonApp/src/app.py; python /pythonApp/flaskApp/run.py'

networks:
  small:
    driver: bridge
    ipam:
     config:
       - subnet: 172.19.0.0/16
```

这里就是吧 mysql 与自己的环境部署到一个子网里面，然后暴露端口直接用就行了。

```bash
$ docker-compose pull # 拉取需要的image

$ docker-compose up -d --build # 后台启动了直接用

# 对于已经在运行的container，更改了本地的文件之后呢还是可以继续使用 docker-compose up -d --build
```

##  常用命令

除了上面的命令；

```bash
$ docker container run -d -p 80:80 nginx # 后台运行创建一个nginx容器

$ docker run -it $(name) # 执行容器中的默认脚本

$ docker container exec -it $(id) sh # 交互式进入容器

$ docker system prune -f # 删除所有已经停止的容器

$ docker image prune -a # 删除所有没有使用的镜像
```

其实吧：`Docker DeskTop`已经可以实现点点点了。  https://www.docker.com/products/docker-desktop

<img src="/img/mics/image-20220219234504845.webp" alt="image-20220219234504845" style="zoom:50%;" />