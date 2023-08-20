---
title: 微信小程序开发
tags: '微信小程序'
author: Laeni
date: '2023-06-07'
updated: '2023-06-07'
---

# 小程序的运行环境

| 运行环境         | 逻辑层         | 渲染层           |
| :--------------- | :------------- | :--------------- |
| iOS              | JavaScriptCore | WKWebView        |
| 安卓             | V8             | chromium定制内核 |
| 小程序开发者工具 | NWJS           | Chrome WebView   |

# 账号相关

# 项目结构

```
.
├── miniprogram
│   ├── app.json // 当前小程序的全局配置 - https://developers.weixin.qq.com/miniprogram/dev/framework/config.html
│   ├── app.ts
│   ├── app.wxss // 当前小程序的全局样式
│   ├── pages
│   │   ├── index
│   │   │   ├── index.json // 页面级别配置，用于对 app.json 配置的覆盖
│   │   │   ├── index.ts
│   │   │   ├── index.wxml // 页面模板，类似于 .html ,用来描述当前页面的结构
│   │   │   └── index.wxss // 页面样式，会覆盖 app.wxss 的全局样式
│   │   └── logs
│   ├── sitemap.json // https://developers.weixin.qq.com/miniprogram/dev/framework/sitemap.html
│   └── utils
│       └── util.ts
├── package.json
├── project.config.json // 工具配置.在工具上做的任何配置都会写入到这个文件,所以一般情况不需要手动编辑该文件 - https://developers.weixin.qq.com/miniprogram/dev/devtools/projectconfig.html
├── project.private.config.json
├── tsconfig.json
└── typings
    ├── index.d.ts
    └── types
        ├── index.d.ts
        └── wx
            ├── index.d.ts
            ├── lib.wx.api.d.ts
            ├── lib.wx.app.d.ts
            ├── lib.wx.behavior.d.ts
            ├── lib.wx.cloud.d.ts
            ├── lib.wx.component.d.ts
            ├── lib.wx.event.d.ts
            └── lib.wx.page.d.ts
```

# ?

##  WXML 模板

### 标签/组件

小程序里没有`div`、`p`、`span`这样的标签，取而代之的是具有特定功能的组件，如`view`、`button`、`text`、“地图”、“视频”、“音频”。

支持的组件参考: [小程序的能力](https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/framework.html)、[WXML](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxml/)。

- `{{}}` - 字符串显示

  在模板中，可以通过`<text>{{msg}}</text>`方式显示变量`msg`的值，而 js 中只需要管理状态即可: `this.setData({ msg: "Hello World" })`。

## WXSS 样式

`WXSS`具有`CSS`大部分的特性，小程序在`WXSS`也做了一些扩充和修改。

1. 新增了尺寸单位。在写`CSS`样式时，开发者需要考虑到手机设备的屏幕会有不同的宽度和设备像素比，采用一些技巧来换算一些像素单位。`WXSS`在底层支持新的尺寸单位`rpx`，开发者可以免去换算的烦恼，只要交给小程序底层来换算即可，由于换算采用的浮点数运算，所以运算结果会和预期结果有一点点偏差。
2. 提供了全局的样式和局部样式。和`app.json`、`page.json`的概念相同，可以写一个`app.wxss`作为全局样式，会作用于当前小程序的所有页面，局部页面样式`page.wxss`仅对当前页面生效。
3. `WXSS`仅支持部分`CSS`选择器

更详细的文档可以参考[WXSS](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxss.html)。

## JS 逻辑交互

一个服务仅仅只有界面展示是不够的，还需要和用户做交互：响应用户的点击、获取用户的位置等等。在小程序里边，我们就通过编写`JS`脚本文件来处理用户的操作。

```html
<view>{{ msg }}</view>
<button bindtap="clickMe">点击我</button>
```

点击`button`按钮的时候，我们希望把界面上`msg`显示成`"Hello World"`，于是我们在`button`上声明一个属性:`bindtap`，在 JS 文件里边声明了`clickMe`方法来响应这次点击操作：

```js
Page({
  clickMe: function() {
    this.setData({ msg: "Hello World" })
  }
})
```

响应用户的操作就是这么简单，更详细的事件可以参考文档 [WXML - 事件](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxml/event.html) 。

此外你还可以在 JS 中调用小程序提供的丰富的 API，利用这些 API 可以很方便的调起微信提供的能力，例如获取用户信息、本地存储、微信支付等。在前边的 QuickStart 例子中，在 `pages/index/index.js` 就调用了 [wx.getUserInfo](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/user-info/wx.getUserInfo.html) 获取微信用户的头像和昵称，最后通过 `setData` 把获取到的信息显示到界面上。更多 API 可以参考文档 [小程序的API](https://developers.weixin.qq.com/miniprogram/dev/framework/app-service/api.html) 。

通过这个章节，你了解了小程序涉及到的文件类型以及对应的角色，在[下个章节](https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/framework.html)中，我们把这一章所涉及到的文件通过 “小程序的框架” 给 “串” 起来，让他们都工作起来。
