---
title: SVG基础知识
tags: SVG
author: Laeni
date: '2023-06-02'
updated: '2023-06-02'
---

# 标签

## svg

SVG 标签包含图像元素并定义图像的框架。

宽度（`width`）和高度（`height`）属性用于定义图像在文档中占用的空间大小。同时，超过该空间的部分将不被显示。

`viewBox`属性定义画布坐标系，其前两个值定义画布原点坐标的位置，后两个值定义画布的大小。每个图像元素都基于此坐标系定位。

在代码`<svg width="100" height="100" viewBox="0 0 100 100">`中，SVG 图像大小和画布大小相同，原点坐标（`0,0`）位于图像左上角，如果想让原点坐标位于中间，可以这样`<svg width="100" height="100" viewBox="-50 -50 100 100">`。在此示例中，大小与宽度和高度处定义的大小相匹配，但这不是必需的。如果它们不匹配，则图像会放大或缩小。

## circle - 圆形

对于圆形，一般定义圆心坐标位置，然后定义半径。

示例：

1. `<circle cx="50" cy="61" r="35" fill="#D1495B"></circle>`

   圆心坐标为`cx=50,xy=61`，半径`r=35`，背景颜色为`fill=#D1495B`（这里与 HTML 不同，不使用`background-color`设置背景颜色）。

2. `<circle cx="50" cy="12.5" r="6" fill="none" stroke="#F79257" stroke-width="2"></circle>`

   相比于上个圆形，该圆形无背景颜色（`fill="none"`），边框颜色为`stroke="#F79257"`，边框半径为`stroke-width="2"`。即该图形表示一个圆环。

## rect - 矩形

对于矩形，一般先定义左上角坐标，然后定位大小。

示例：

1. `<rect x="41" y="18" width="18" height="10" fill="#F79257"></rect>`

   矩形坐标为`x="41",y="18"`，大小为`width="18",height="10"`，背景颜色为`fill="#F79257"`。

2. `<rect x="90" y="60" width="20" height="5" rx="4" fill="none" stroke="white" stroke-width="2px" />`

   `rx="x" ry="x"`用于设置边框圆角大小。

   <svg width="100" height="100" viewBox="0 0 100 100">
     <rect x="0" y="0" width="100" height="100" fill="#D1495B" />
     <rect x="10" y="35" width="80" height="30" rx="4" fill="none" stroke="#cd803d" stroke-width="2px" />
   </svg>

> 上述示例中的三个图形最终形状如下:
>
> <svg width="100" height="100" viewBox="0 0 100 100">
>   <circle cx="50" cy="61" r="35" fill="#D1495B"></circle>
>   <circle cx="50" cy="12.5" r="6" fill="none" stroke="#F79257" stroke-width="2"></circle>
>   <rect x="41" y="18" width="18" height="10" fill="#F79257"></rect>
> </svg>

## polygon - 多边形

在坐标系中描绘一系列的点，然后这些点将构成一个封闭的图形（至少三个点才能构成封闭图形）。

示例：

1. `<polygon points="80,70 160,170 0,170" fill="#234236" />`

   多边形的三个点分别是`80,70`、`160,170`和`0,170`。

<svg width="200" height="200" viewBox="0 0 200 200">
  <polygon points="80,70 160,170 0,170" fill="#234236" />
  <polygon points="80,40 140,130 20,130" fill="#0C5C4C" />
  <polygon points="80,0 120,80 40,80" fill="#38755B" />
  <rect x="60" y="170" width="40" height="30" fill="brown" />
</svg>
## line - 线条

<svg width="200" height="200" viewBox="0 0 200 200">
  <circle cx="100" cy="50" r="30" fill="#cd803d" />
  <circle cx="88" cy="45" r="3" fill="white" />
  <circle cx="112" cy="45" r="3" fill="white" />
  <rect x="90" y="60" width="20" height="5" rx="2" fill="none" stroke="white" stroke-width="2px" />
  <line x1="60" y1="90" x2="140" y2="90" stroke="#cd803d" stroke-width="35px" stroke-linecap="round" />
  <line x1="75" y1="150" x2="100" y2="85" stroke="#cd803d" stroke-width="35px" stroke-linecap="round" />
  <line x1="125" y1="150" x2="100" y2="85" stroke="#cd803d" stroke-width="35px" stroke-linecap="round" />
  <circle cx="100" cy="90" r="5" />
  <circle cx="100" cy="110" r="5" />
</svg>


# 参考

- [SVG tutorial (codepen.io)](https://codepen.io/HunorMarton/full/PoGbgqj)
