---
title: 'electronjs + nextjs入门'
author: 'Laeni'
date: '2022-05-31'
updated: '2022-05-31'
hide: true
---

[electron-react-boilerplate](https://github.com/electron-react-boilerplate/electron-react-boilerplate)是社区中一个`electron`+`react`的项目，可以参考。但是平时使用的`react`框架一般是`umi`和`next`，所以此次是参考该项目搭建的一个`electron`+`next`的项目。

## 工程搭建

1. 参考next创建next工程

   ```shell
   $ npx create-next-app@latest --typescript11
   ```

2. 安装`electron`依赖

   默认情况下直接安装大概率会失败，因为`electron`包含具体实现（浏览器），该部分并不是直接放在`npm`的，而是单独存放的，这这部分内容常常因为网络问题无法下载。由于不是放在`npm`，所以设置npm代理是无法解决问题，还需要设置`electron`代理。

   设置代理方式为将内容`ELECTRON_MIRROR=http://npm.taobao.org/mirrors/electron/`设置为环境变量或者添加到`~/.npmrc`配置文件中即可。

   然后正常安装依赖即可：

   ```shell
   $ npm i -D electron@latest
   ```

3. 2



## 参考

1. <https://blog.csdn.net/weixin_38080573/article/details/105113219>

