---
title: ClashX测速失败以及Mac终端走代理
categories: 捣鼓分享
keywords: ClashX
top_img: /img/24672.webp
cover: /img/251378418.webp
abbrlink: '567e1422'
date: 2021-03-09 21:46:03
---

##  ClashX测速失败以及Mac终端走代理

用机场也快一年半了，ClashX的话在Mac上很明显在终端上没有走代理

然后我写这篇在2021年份了，但是在**CSDN上没法审核通过**。

放到我博客上来科学上网，**不然百度拿来看广告的**？

{% note green 'fas fa-bullhorn' flat %}

注意：这是我的邀请链接（我可以收到 **10%** 的返利）

我一直在用，暂时没翻过车，然后推荐半年半年买，价格合理

我的链接： https://fastlink-aff.com/auth/register?code=HengY1Sky

{% endnote %}


## 下载Clashx 

官方下载链接：https://github.com/yichengchen/clashX/releases

<img src="/img/mics/20210309210610923.webp" alt="选择Clashx.dmg下载" style="zoom:80%;" />

## 测速失败

**我的原因是：**
系统时间慢啦，**是整整慢了一天**！
说来搞笑，我测速不管怎样都是失败。
时分秒都显示正确，花了几个晚上去翻看帖子。
Github上的issue在小火箭和Clashx看到了凌晨
都没解决，突然看到时间慢了一天，当时真想哭又想笑

**还有个原因：**
ClashX内置dns不通引起的问题：
这功能是让网络更干净，但特殊时期dns不通了倒直接卡住了，此时请关闭该功能。
**方法如下：**
点击**小猫咪〉配置〉打开本地配置文件夹**
用文本编辑器打开编辑 yaml配置文件
将 dns下的 **enable** 值改为 **false**
修改保存后默认系统会弹出重载配置提示，点击重载

## 终端走代理

我是看了这位大佬的帖子[Clashx走终端](https://blog.csdn.net/DSZhappy/article/details/108393159?ops_request_misc=&request_id=&biz_id=102&utm_term=clashxmac%E7%BB%88%E7%AB%AF%E4%BB%A3%E7%90%86&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-108393159.pc_search_result_before_js)
但他只说了结果，具体方法得自己摸索
其实现在弄多了之后，也很快找到了具体的解决方法。

  **1.第一步**

`vim .bash_profile`

 **2. 第二步：**

`输入i->进入insert-> 复制下面内容 ->：wq!(强制写入退出)`

```
function proxy_on() {
    export no_proxy="localhost,127.0.0.1,localaddress,.localdomain.com"
    export http_proxy="http://127.0.0.1:7890"
    export https_proxy=$http_proxy
    #export all_proxy=socks5://127.0.0.1:7890 # or this line
    echo -e "已开启代理"
}

function proxy_off(){
    unset http_proxy
    unset https_proxy
    echo -e "已关闭代理"
}

```

<img src="/img/mics/20210309212421139.webp" alt="效果图" style="zoom:80%;" />


**4. 第四步：**

```bash
$ source ~/.bash_profile
```

**5.第五步：**

```bash
$ open -e .zshrc
```

<img src="/img/mics/20210309213339480.webp" alt="效果图" style="zoom:80%;" />

**6.第六步**

```bash
$ source .zshrc
```

恭喜你！到这步就可以啦！

重新打开终端 `proxy_on`就打开了 不管是Git啥的都快很多。