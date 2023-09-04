---
title: WEB中同时使用多张SVG图片导致颜色混乱
tags: WEB, React, SVG
author: Laeni
date: '2021-09-04'
updated: '2021-09-04'
---

在同一个页面上同时使用多张SVG图片（直接将SVG代码嵌入到HTML中）可能会导致部分图片颜色错乱，原因是大部分SVG都是通过软件制作的，软件制作时往往会增加一些样式，这些样式一般又是以类似`CSS内联`的方式申明的，这就需要给元素进行标记（一般是用`id`），而`id`是由软件自动生成，这就很容易造成`id`重复。而同一个`id`在一张SVG图片中是唯一的，但是在整个页面中就不一定了，所以这些冲突就是导致颜色等错乱的根本原因。

下图为颜色错误的效果：

![SVG颜色错乱示例](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/web/fix/svg-color-error/svg-color-error.jpg)

而实际应该是类似这样的：

![SVG正确的样式](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/web/fix/svg-color-error/svg-color-ok.jpg)

解决方法至少有以下两种：

1. 使用`<img>`标签来使用SVG图片，这样每张图片的`id`都是隔离的,所以不会出现这种情况。

2. 使每个`id`在所有图片中都是唯一的，这种方式一般是给一张SVG图片的所有`id`都加上一个唯一前缀。

   增加`id`的方式又可以分为两种:

   1. 从SVG源码级别增加唯一前缀。如果图片少并且SVG简单的话，手动就可以完成，否则可能得借助工具进行。

      现提供一个windows和Linux版本的简单小工具，可直接[下载使用](https://gitee.com/laeni/svg-id-prefix/releases/1.0.0)，如需mac版本可直接通过[源码](https://gitee.com/laeni/svg-id-prefix)进行编译。

   2. 使用组件（如React组件或Vue组件）来解决该问题，比如在打包时，该组件自动进行前缀的添加工作。

