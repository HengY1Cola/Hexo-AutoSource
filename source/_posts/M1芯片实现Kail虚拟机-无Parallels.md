---
title: M1芯片实现Kail虚拟机(无Parallels)
categories: 分享
keywords: Kail虚拟机
top_img: /img/kali-bright.webp
cover: /img/4034416.webp
abbrlink: 11d32e85
date: 2022-03-12 10:00:17
---

##  前言

我发现M1芯片想实现虚拟机是真的费劲。我开始找Windows资源(必要时平台会限制)，找不到。结果淘宝解君愁。这下好了Parallels的PD启动好像给了window系统。然后我有14天试用，我是可以直接下载虚拟机，但是我过期了我也不知道会是个什么情况。所以我就另辟蹊径，我根据docker官方的kail镜像自己打包个不好嘛。（以及Ubuntu都是一个道理）

##  为啥不想云容器

首先我的教程已经写的非常详细了，几乎只需要复制运行就好了。然后在2022年了有的地方变了，挺多坑都绕过来了。

如果你实在不想动手想直接白嫖(*￣︿￣)，可以到我的仓库看看👉[仓库地址](https://hub.docker.com/repository/docker/hengyisky/daily)

但是**我还是建议按照下面花点时间**，因为我上传上去的10G不到而我本地有20G+，肯定很多都阉割了


{% note green 'fas fa-bullhorn' flat %}

里面的坑包括但不限于：
1. 包过期了找不到了
2. CPU编译的设置与优化
3. 包安装等待
...

{% endnote %}

##  开始慢慢搞得像个系统

先去官网找到官方镜像：发现支持arm，那就好办了。

<img src="/img/mics/202203112311539.webp" alt="" style="width:80%;" />

```bash
$ docker pull kalilinux/kali-rolling:latest
$ docker images
$ docker run -id  --restart=always --name=kali --cpus=2 -p 2222:22 -p 4888:4500 $(id) # <=填写
$ docker ps -a # 查看状态
$ docker exec -it kali /bin/bash # 进入命令行
$ passwd root # 修改命令
# 开始换源
$ cp /etc/apt/sources.list /etc/apt/sources.list.bak && rm /etc/apt/sources.list
$ cd /etc/apt
$ echo 'deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib' >> /etc/apt/sources.list
$ echo 'deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib' >> ./sources.list
$ apt-get update && apt-get upgrade && apt install vim curl git wget
# 把下面的继续写入sources.list
$ echo "deb http://mirrors.ustc.edu.cn/kali kali-rolling main contrib non-free" >> /etc/apt/sources.list
$ echo "deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main contrib non-free" >> /etc/apt/sources.list
$ echo "deb http://mirrors.ustc.edu.cn/kali-security kali-current/updates main contrib non-free" >> /etc/apt/sources.list
$ echo "deb-src http://mirrors.ustc.edu.cn/kali-security kali-current/updates main contrib non-free" >> /etc/apt/sources.list
```

开始搞双Python与SSH登陆

```bash
$ apt-get -y install net-tools openssh-server python2.7
$ apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev
$ cd ~ && wget https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tgz && tar -xf Python-3.9.1.tgz && cd Python-3.9.1
$ ./configure --enable-optimizations
$ make -j 4 && make altinstall
$ python3.9 --version
$ apt install python3-pip
$ wget https://bootstrap.pypa.io/pip/2.7/get-pip.py && python2.7 get-pip.py

$ vim ~/.bashrc
# 将下面的写入
# alias pip=/usr/local/bin/pip2.7
# alias pip3=/usr/local/bin/pip3.9
# alias python=/usr/bin/python2.7
# alias python3=/usr/local/bin/python3.9
$ source ~/.bashrc

$ echo "PermitRootLogin yes" >> /etc/ssh/sshd_config && service ssh restart
```

<img src="/img/mics/202203120152396.webp" alt="" style="width: 67%;" />

##  SSH上去的效果

<img src="/img/mics/202203120954020.webp" alt="image-20220312015300950" style="width:67%;" />

把kail核心工具包装上：很大！

```bash
$ apt-get update && apt-get upgrade
$ apt-get install kali-linux-everything
```

##  保存容器为镜像与文件挂载

```bash
# 退出
$ exit
# 停掉
$ docker stop kali
# 保存为image
$ docker commit d46ea163bd90 hengyisky/daily:kail2022
# 删除container并挂载文件
$ docker rm -f kali
$ docker run -id  -v /xxx/xxx:/root/vol  --restart=always --name=kali --cpus=2 -p 2222:22 -p 4888:4500 63ddce7ec717
# 再次进入
$ docker exec -it kali /bin/bash
```

<img src="/img/mics/202203120949492.webp" alt="" style="width:67%;" />