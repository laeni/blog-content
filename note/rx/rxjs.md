---
title: 'RxJS简单使用笔记'
author: 'Laeni'
tags: 'RxJS, ReactiveX, Observables'
date: '2021-07-28'
updated: '2021-07-28'
---

[RxJS](https://rxjs-dev.firebaseapp.com/) 是使用 Observables 的响应式编程的库，它使编写异步或基于回调的代码更容易。这个项目是 [Reactive-Extensions/RxJS](https://github.com/Reactive-Extensions/RxJS)(RxJS 4) 的重写，具有更好的性能、更好的模块性、更好的可调试调用堆栈，同时保持大部分向后兼容，只有一些破坏性的变更(breaking changes)是为了减少外层的 API 。

## 基本概念

- **Observable (可观察对象):** 表示一个概念，这个概念是一个可调用的未来值或事件的集合。
- **Observer (观察者):** 一个回调函数的集合，它知道如何去监听由 Observable 提供的值。
- **Subscription (订阅):** 表示 Observable 的执行，主要用于取消 Observable 的执行。
- **Operators (操作符):** 采用函数式编程风格的纯函数 (pure function)，使用像 `map`、`filter`、`concat`、`flatMap` 等这样的操作符来处理集合。
- **Subject (主体):** 相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式。
- **Schedulers (调度器):** 用来控制并发并且是中央集权的调度员，允许我们在发生计算时进行协调，例如 `setTimeout` 或 `requestAnimationFrame` 或其他。

## 入门

### 安装

```sh
npm install rxjs
```

## 操作符

### 创建

#### fromEvent - 从事件源创建

```typescript
// 原始
document.addEventListener('click', () => console.log('Clicked!'));

// RxJs
import { fromEvent } from 'rxjs';
fromEvent(document, 'click').subscribe(() => console.log('Clicked!'));
```



