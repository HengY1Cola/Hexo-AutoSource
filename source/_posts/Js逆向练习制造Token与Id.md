---
title: Js逆向练习制造Token与Id
categories: 捣鼓分享
keywords: Js逆向
top_img: /img/7970547.webp
cover: /img/143R44B1.webp
abbrlink: 6d98b2b4
date: 2022-02-11 00:30:52
---

## 前言：

闲来无聊，把《Pyhton3网络爬虫开发实战(第二版)》看完了Js逆向部分。

最后的实战部分感觉挺有挑战性的，正好崔佬也有详细的教程。

平时的逆向都是野路子，刚好快回学校了有时间。

那为什么不自己动手下呢？下面记录下过程，只会更加详细。

##  观察页面：

废话不多说，直接上靶场：https://spa6.scrape.center

我们的目标是：1.拿到列表页的请求Ajax的Token加密 2. 详情页的Id加密与Token

![](/img/JsReverse/58413655a0d3b23bf06dc701444e3435.webp)

<img src="/img/JsReverse/image-20220210220457469.webp" alt="" width=90% />

查看网页源码可以看到：很强烈的Vue打包出来的样子，即使用的使用`SPA`页面

<img src="/img/JsReverse/image-20220210220720322.webp" alt="" width=90% />

观察Js也会发现，代码压缩，变量名字十六进制转换。

<img src="/img/JsReverse/image-20220210220948065.webp" alt="" width=90% />

好了我们将任务进行拆分，先拿到列表页面的加密规则，即去请求列表的token怎么搞到的

##  获取列表页面：

使用Ajax断点，直接拿到即将提交请求那个地方，然后使用堆栈，一点点往回找，这是基本思路

根据上面可以看到请求路径是：`/api/movie/?limit=10&offset=0&token=...` 打上断点刷新页面

<img src="/img/JsReverse/image-20220210221604905.webp" alt="" width=90% />

<img src="/img/JsReverse/image-20220210221714743.webp" alt="" width=90% />

开始针对堆栈往回找：发现axios的get方法：参数都是跟上面请求的十分符合。

到这里我们可以看到`_0x263439 = Object(_0x2fa7bd['a'])(this['$store']['state']['url']['index']);`这个应该就是token

<img src="/img/JsReverse/image-20220210222010258.webp" alt="" width=90% />

因为找到具体的了，取消xhr断电在169行打上断点。把鼠标移在上面可以发现

`/api/movie`作为参数传入进去，前面是个函数，同样把鼠标移上去也可以看到

<img src="/img/JsReverse/image-20220210222357464.webp" alt="" width=90% />

然后我们使用Watch监视看看对应的`_0x2fa7bd`：去找下面的函数

<img src="/img/JsReverse/image-20220210222633965.webp" alt="" width=90% />

<img src="/img/JsReverse/image-20220210222902673.webp" alt="" width=90% />

格式化后的对应的函数就是把参数传入进来生成Token的逻辑函数了：

<img src="/img/JsReverse/image-20220210223045604.webp" alt="" width=90% />

单步调试：去看逻辑

<img src="/img/JsReverse/image-20220210223400006.webp" alt="" width=90% />

<img src="/img/JsReverse/image-20220210224351817.webp" alt="" width=90% />

*最后的逻辑就是：

**传入地址这里是：`/api/movie`与时间戳用逗号拼接后，使用SHA1进行加密后，再与时间戳拼接，再用Base64编码一次就是Token**

这句话有点长，谅解下QAQ，到这里就可以用Python模拟了

当然也可以使用自带的重写功能：方便看其中的某些变量

> 步骤是：
>
> 1. 创建一个保存的文件夹
> 2. 被格式化的文件是不能修改的但是可以复制出来
> 3. 在编辑器里面修改好了然后在原文件里面全选然后替换
> 4. 下次刷新会自动替换

<img src="/img/JsReverse/image-20220210225656780.webp" alt="" width=90% />

##  获取详情页加密

正如最开始的时候会发现：跳转到详情页面的链接已经加密好了，而不是在请求时候进行加密：

<img src="/img/JsReverse/image-20220210233557960.webp" alt="" width=90% />

看到请求的链接的Token的话：应该是跟着请求加密生成的。

所以可以看出来我们现在要拿到`/detail/xxx`是怎么来的以及Token怎么出现的。

同时可以盲猜一波Token的加密方式应该是一样的，即只是换了上面的参数。

现在我们开始分析：**确定是在完成请求列表页后经过Js加密已经形成的**

<img src="/img/JsReverse/image-20220210234105022.webp" alt="" width=90% />

这样打断点即加载好了开始调试。**但是这里有个问题就是接下来不知道要走多久**<img src="/img/JsReverse/image-20220210234232496.webp" alt="" width=90% />

这个地方就是学到新东西的地方：因为token的样子应该是最后Base64编码过，而在前端Base64的库就那么几个

通常是`btao`或者是`crypto-js`等，则可以使用🪝钩子来截取到

> 这个意思就是func = object[attr]先拿到，然后重写之后，调用回去执行

```javascript
(function(){
  'use strict'
  function hook(object ,attr){
    var func = object[attr]
    object[attr] = function(){
      console.log("hooked", object, attr, arguments)
      var ret = func.apply(object, arguments)
      debugger
      console.log("res", ret)
      return ret
    }
  }
  hook(window, 'btoa')
})()
```

注入方式可以是：1. 控制台 2. 重写Js 3. 油猴脚本

因为打了断点，我们在控制台重写btoa方法，这样到了方法执行的时候就会到我们写入的里面去验证

这里有几个注意点：

1. 控制台输入后 可以调用`btoa('1')`进入临时文件中的debugger
2. 因为是`SPA`页面所以直接请求第二页可以触发写入的btoa，**刷新则不行**！

<img src="/img/JsReverse/image-20220211000800269.webp" alt="" width=90% />

点击第二页，加密规则都是一样的。直接触发（这里我是第二页去点第一页）

<img src="/img/JsReverse/image-20220211001051507.webp" alt="" width=90% />

往下找了一层发现已经出来了则继续往下找

<img src="/img/JsReverse/image-20220211001220739.webp" alt="" width=90% />

发现了拼凑：一串不知道是啥的+一个啥

<img src="/img/JsReverse/image-20220211001431465.webp" alt="" width=90% />

同时发现：`_0x11a046`是写死的

<img src="/img/JsReverse/image-20220211001556051.webp" alt="" width=90% />

拼凑的那个数字继续往下找：发现就是拿到结果的ID

<img src="/img/JsReverse/image-20220211001717040.webp" alt="" width=90% />

到这里详情页加密已经出来了。现在验证下token怎么来的，好的方式一模一样

<img src="/img/JsReverse/image-20220211002248300.webp" alt="" width=90% />

## 总结

1. 做Js逆向基础要懂要会
2. 心要大，要敢猜，不然真的一下子跟正确答案擦肩而过
3. 同时心要细，不同的写法同一种效果，不要放过觉得不可能的地方
4. Python实现的方式很多，自己多捣鼓下