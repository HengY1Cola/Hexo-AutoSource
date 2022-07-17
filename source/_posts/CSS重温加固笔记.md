---
title: CSS重温加固笔记
categories: 分享
keywords: CSS
top_img: /img/2987551.webp
cover: /img/1631859.webp
abbrlink: 2c8a606b
date: 2022-07-17 21:08:10
---

慕课链接：https://coding.imooc.com/class/chapter/164.html
暑假第一步：因为好久没弄前端的东西了，先系统温故下知识，别说还真有新收获

## 基础知识

### 设配移动端的第一步：

在header里面加上**“viewport”**

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### HTML 相关需要关注的点

<img src="https://pic.hengy1.top/typoraimage-20220712234549562.png" alt="image-20220712234549562" style="zoom:50%" />

```html
<button type="button">普通按钮</button>
<button type="submit">提交按钮一</button>
<input type="submit" value="提交按钮二"/>
<button type="reset">重置按钮</button>
```

### HTML版本迭代

新的HTML5的新增语法：http://h5o.github.io/  可以像看大纲一样看

<img src="https://pic.hengy1.top/typoraimage-20220712235818022.png" alt="image-20220712235818022" style="zoom:50%;" />

验证网站：https://validator.w3.org/check

```html
<!--在XML中-->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    
<!--在HTML5中-->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8" />
```

### HTML分类

会不会独占一行：块级 block  --- inline-block --- 行内 inline

<img src="https://pic.hengy1.top/typoraimage-20220713001258558.png" alt="image-20220713001258558" style="zoom:80%;" />

### HTML嵌套关系

1. 块级元素可以包含行内元素的

2. 块级元素不一定能包含行内元素的

3. “行内元素一般不能包含块级元素” \<a\>可以包含块级元素

4. 引入CSS Reset 来重置浏览器带来的样式

   1. https://meyerweb.com/eric/tools/css/reset/

   2. https://clarle.github.io/yui3/yui/docs/cssreset/

   3. ```html
      <style>
          *{
             margin:0;
             padding:0;
           }
      </style>
      ```

   4. https://necolas.github.io/normalize.css/

## CSS布局

### 选择器的分类

<img src="https://pic.hengy1.top/typoraimage-20220716104004182.png" alt="image-20220716104004182" style="zoom:50%;" />

### 选择器权重

> 累加是不进位的

<img src="https://pic.hengy1.top/typoraimage-20220716104540664.png" alt="image-20220716104540664" style="zoom:50%;" />

<img src="https://pic.hengy1.top/typoraimage-20220716104934422.png" alt="image-20220716104934422" style="zoom:50%;" />

### 字体

<img src="https://pic.hengy1.top/typoraimage-20220716105112755.png" alt="image-20220716105112755" style="zoom:50%;" />

<img src="https://pic.hengy1.top/typoraimage-20220716110055652.png" alt="image-20220716110055652" style="zoom:50%;" />

文字的 vertical-align 来设置一行文字通过什么线进行对齐

使用display: block 将inline换成块元素

### 滚动

- visible：滚动条隐藏
- hidden：滚动条隐藏
- scroll：滚动条显示
- auto：滚动条自动显示

### 文字折行

- overflow-wrap 通用换行控制
- word-break 针对多字节文字
- white-space 空白处是否可以换行

### 如何隐藏checkbox

<img src="https://pic.hengy1.top/typoraimage-20220716185203611.png" alt="image-20220716185203611" style="zoom:50%;" />

### 布局知识体系

<img src="https://pic.hengy1.top/typoraimage-20220716185910455.png" alt="image-20220716185910455" style="zoom:50%;" />

<img src="https://pic.hengy1.top/typoraimage-20220716190327636.png" alt="image-20220716190327636" style="zoom:50%;" />

- Float
  - 可以将inline形成块
  - 位置尽量靠上
  - 不影响其他块级元素位置
  - 影响其他块级元素内部文本

清除浮动的方式：

<img src="https://pic.hengy1.top/typoraimage-20220716192921244.png" alt="image-20220716192921244" style="zoom:66%;" />

### 响应式

1. 隐藏 + 折行 + 自适应空间

2. 加上 viewport

<img src="https://pic.hengy1.top/typoraimage-20220716203041824.png" alt="image-20220716203041824" style="zoom:67%;" />

3. rem单位：根据font-size进行设置

## 动画和效果

<img src="https://pic.hengy1.top/typoraimage-20220716213401401.png" alt="image-20220716213401401" style="zoom:67%;" />

<img src="https://pic.hengy1.top/typoraimage-20220716215915364.png" alt="image-20220716215915364" style="zoom:67%;" />

<img src="https://pic.hengy1.top/typoraimage-20220716220434018.png" alt="image-20220716220434018" style="zoom:67%;" />

<img src="https://pic.hengy1.top/typoraimage-20220716220616559.png" alt="image-20220716220616559" style="zoom:50%;" />

1. transition 使用
2. 关键帧使用：

<img src="https://pic.hengy1.top/typoraimage-20220717095309743.png" alt="image-20220717095309743" style="zoom:67%;" />

<img src="https://pic.hengy1.top/typoraimage-20220717100332207.png" alt="image-20220717100332207" style="zoom:67%;" />

## 框架集成和CSS工程

### 预处理器

<img src="https://pic.hengy1.top/typoraimage-20220717105944216.png" alt="image-20220717105944216" style="zoom:67%;" />

1. 变量：

<img src="https://pic.hengy1.top/typoraimage-20220717110347199.png" alt="image-20220717110347199" style="zoom:67%;" />

<img src="https://pic.hengy1.top/typoraimage-20220717111026614.png" alt="image-20220717111026614" style="zoom:67%;" />

2. mixin 复用样式：直接加在css中

<img src="https://pic.hengy1.top/typoraimage-20220717113431832.png" alt="image-20220717113431832" style="zoom:80%;" />

<img src="https://pic.hengy1.top/typoraimage-20220717113501351.png" alt="image-20220717113501351" style="zoom:67%;" />

<img src="https://pic.hengy1.top/typoraimage-20220717114417447.png" alt="image-20220717114417447" style="zoom:67%;" />

3. extend 

<img src="https://pic.hengy1.top/typoraimage-20220717114908970.png" alt="image-20220717114908970" style="zoom:80%;" />

<img src="https://pic.hengy1.top/typoraimage-20220717114932023.png" alt="image-20220717114932023" style="zoom:80%;" />

4. loop 循环

<img src="https://pic.hengy1.top/typoraimage-20220717115545611.png" alt="image-20220717115545611" style="zoom:80%;" />

<img src="https://pic.hengy1.top/typoraimage-20220717115813693.png" alt="image-20220717115813693" style="zoom:80%;" />

 <img src="https://pic.hengy1.top/typoraimage-20220717115910993.png" alt="image-20220717115910993" style="zoom:80%;" />







5. import 模块化

<img src="https://pic.hengy1.top/typoraimage-20220717120556629.png" alt="image-20220717120556629" style="zoom:80%;" />

6. 框架处理

- compass
- est

###  Bootstrap

去官网看文档：提供了CSS与JS组件

<img src="https://pic.hengy1.top/typoraimage-20220717153750093.png" alt="image-20220717153750093" style="zoom:67%;" />

<img src="https://pic.hengy1.top/typoraimage-20220717154552536.png" alt="image-20220717154552536" style="zoom:67%;" />

### 工程化

1. PostCSS

<img src="https://pic.hengy1.top/typoraimage-20220717161609959.png" alt="image-20220717161609959" style="zoom:67%;" />



<img src="https://pic.hengy1.top/typoraimage-20220717161750054.png" alt="image-20220717161750054" style="zoom:67%;" />

2. webpack

<img src="https://pic.hengy1.top/typoraimage-20220717162946940.png" alt="image-20220717162946940" style="zoom:67%;" />

<img src="https://pic.hengy1.top/typoraimage-20220717164330742.png" alt="image-20220717164330742" style="zoom:67%;" />