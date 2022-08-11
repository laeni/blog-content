---
title: WEB技术框架选型参考
description: 
author: Laeni
tags: React, Vue, Angular, WEB技术选型
date: '2021-05-31'
updated: '2021-06-02'
---

以下为目前三大框架的的一些优劣的不完全举例，可供选型参考：

## [React](https://react.docschina.org/)方向

### 优势

1. 轮子众多，几乎在各个领域都能找到比较好的解决方案
2. 由于第一点原因，所以它也同时适用于前台或中后台
3. 几乎所有大厂都是以[React](https://react.docschina.org/)为主要技术栈
4. 对[TypeScript](https://www.typescriptlang.org/)支持良好，且可以不使用[TypeScript](https://www.typescriptlang.org/)
5. 可以不使用[nodejs](https://nodejs.org/zh-cn/)（但稍微复杂项目一般都会使用[nodejs](https://nodejs.org/zh-cn/)环境开发）

### 劣势

1. 由于学习曲线稍微比[Vue](https://cn.vuejs.org/v2/guide/)更陡峭，所以导致新人学习意愿不高，以至于会的人不多
2. 相比于[Vue](https://cn.vuejs.org/)，入门门槛稍微偏高
3. 大部分情况下需要自己考虑代码拆分（像[UmiJS](https://v2.umijs.org/zh/guide/)、[Next.js](https://nextjs.org/docs/getting-started)这样更高级的框架会自动进行代码拆分）

### 常用相关框架

1. [UmiJS](https://v2.umijs.org/zh/guide/)：蚂蚁金服倾力打造，是一款基于[React](https://react.docschina.org/)的通用型js框架，起设计大部分参考自[Next.js](https://nextjs.org/docs/getting-started)
2. [Next.js](https://nextjs.org/docs/getting-started)：基于[React](https://react.docschina.org/)的一款高级框架，和[UmiJS](https://v2.umijs.org/zh/guide/)类似
3. [Ant Design](https://ant.design/index-cn)：一套前端设计规范，目前官方实现了[React](https://react.docschina.org/)版本，但社区实现了[Vue](https://cn.vuejs.org/)和[Angular](https://angular.cn/)版本
4. [Ant Design Pro](https://preview.pro.ant.design/dashboard)：主要面向中后台网站，有很多成熟可快速使用的组件

## [Vue](https://cn.vuejs.org/)方向

### 优势

1. 快速上手。正如[Vue官网](https://cn.vuejs.org/)所述，其是一款渐进式框架（即不需要知道太多知识也能快速上手），很适合新手学习
2. 同时吸纳了[React](https://react.docschina.org/)的“自上而下”的数据流思想，同时也有类似[Angular](https://angular.cn/)的变量修改检测等。（是优势也是劣势，由于混合使用常常让新手分不清该用哪种）
3. 模板与HTML原生及其接近，可以使新人更容易上手
4. 可以不使用[nodejs](https://nodejs.org/zh-cn/)（但稍微复杂项目一般都会使用[nodejs](https://nodejs.org/zh-cn/)环境开发）

### 劣势

1. 对[TypeScript](https://www.typescriptlang.org/)支持较弱（[Vue3](https://v3.vuejs.org/guide/introduction.html)版本有一定改善，但是因为要兼容[Vue2](https://cn.vuejs.org/v2/guide/)的原因，所以也不能很好支持或者使用起来很别扭）
2. 对复杂的需求很难处理（虽然有方法处理，但是最后的实现肯定会比[React](https://react.docschina.org/)或者[Angular](https://angular.cn/)的实现更难以理解和维护）
3. 大部分情况下需要自己考虑代码拆分（像[Nuxt.js](https://nuxtjs.org/)这样更高级的框架会自动进行代码拆分）

### 常用

### 相关框架

1. [Nuxt.js](https://nuxtjs.org/)：类似于[Next.js](https://nextjs.org/docs/getting-started)，提供了一整套解决方案，区别在于一个是[Vue](https://cn.vuejs.org/v2/guide/)的实现，一个是[React](https://react.docschina.org/)的实现。但是目前该框架只支持[Vue2](https://cn.vuejs.org/v2/guide/)，上不支持[Vue3](https://v3.vuejs.org/guide/introduction.html)。
2. [Ant Design for Vue](https://2x.antdv.com/components/overview-cn/)：[Ant Design](https://ant.design/index-cn) 规范的[Vue](https://cn.vuejs.org/v2/guide/)实现
3. [Element](https://element.eleme.cn/#/zh-CN)：饿了么基于[VUE](https://cn.vuejs.org/v2/guide/)实现的组件库

## [Angular](https://angular.cn/)方向

### 优势

1. 对[TypeScript](https://www.typescriptlang.org/)支持良好
2. 同[Vue](https://cn.vuejs.org/v2/guide/)一样，模板与HTML原生及其接近，可以使新人更容易上手
3. 使用引用变更检测来检查变量是否被修改，且该检测能支持到对象内部（比较符合一般人的思维习惯）
4. 原生设计上就支持代码拆分（通过模块实现）

### 劣势

1. 学习曲线陡峭，需要同时对几个相关知识点有一定了解后才能上手
2. 新版只能使用[TypeScript](https://www.typescriptlang.org/)进行开发
3. 必须使用[nodejs](https://nodejs.org/zh-cn/)环境进行开发

### 相关框架

1. [NG-ZORRO](https://ng.ant.design/docs/introduce/zh)：[Ant Design](https://ant.design/index-cn) 规范的[Angular](https://angular.cn/)实现

## 总结

**方向选择**

1. 中后台项目优先考虑[React](https://react.docschina.org/)、[Angular](https://angular.cn/)
2. 简单项目可以考虑[Vue](https://cn.vuejs.org/v2/guide/)，其次考虑[React](https://react.docschina.org/)，因为你永远不知道实际编码的时候会有多复杂
3. 有[SEO](https://baike.baidu.com/item/搜索引擎优化/3132?fr=aladdin)诉求（需要考虑服务端渲染或者预渲染）的优先考虑[React](https://react.docschina.org/)，其次考虑[Vue](https://cn.vuejs.org/v2/guide/)

