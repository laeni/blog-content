---
title: 'RxJS简单使用笔记'
author: 'Laeni'
tags: JavaScript, Reactive
date: '2021-07-28'
updated: '2021-09-16'
---

[RxJS](https://rxjs-dev.firebaseapp.com/) 是一个使用可观察序列编写异步和基于事件的程序的库。它提供了一种核心类型[Observable](https://rxjs-dev.firebaseapp.com/guide/observable)、卫星类型（Observer、Scheduler、Subjects）和受[Array#extras](https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/1.6)启发的操作符（map、filter、reduce、every 等），以允许将异步事件作为集合处理。

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
// 每次点击时都会输出'Clicked!'
fromEvent(document, 'click').subscribe(() => console.log('Clicked!'));
```

#### interval - 创建一个以指定时间间隔发出连续的数字的Observable 

 该 Observable 可以指定 IScheduler。

```typescript
// 从 0 开始,每 1s 发射一个数字
interval(1000).subscribe(x => console.log(x)); // => 0 1 2 3 4 5 6 ...
```

#### take - 从其他`Observable `创建一个只转发一定数量的新`Observable `

```typescript
interval(1000)
    // 值转发 Observable 的初始 3 个值
    .pipe(take(3))
    .subscribe(v => console.log(v)); // => 0 1 2
```

#### scan - 扫描|"累加"源 Observable 的值

为了演示一般以累加数字为例，实际上可以“累加”任意类型的值

```typescript
interval(1000)
    // acc - 为上一次 return 的结果,第一次的结果为初始值
    // value - 当前发射的数据
    // index - 该数据的索引
    .pipe(scan((acc, value, index) => {
        return acc + value;
    }, 0)) // 0 为初始数据，比如空数组 [] 等
    .subscribe(count => console.log(count)); // 0 1 3 6 10 15 21 28 ...
```

#### throttleTime 重复忽略指定时间范围内的值

```typescript
interval(100)
      .pipe(throttleTime(1000))
      .subscribe(count => console.log(count)); // 0 11 22 33 44 ...
```

#### map - 转换

类似于数组的`map`函数

#### buffer - 缓冲

缓冲源 Observable 的值直到 `closingNotifier` 发出。

将过往的值收集到一个数组中，并且仅当另一个 Observable 发出通知时才发出此数组。

![img](F:\Objects\cn.laeni\blog-content\note\rx\rxjs.assets\buffer.png)

将 Observable 发出的值缓冲起来直到 `closingNotifier` 发出数据, 在这个时候在输出 Observable 上发出该缓冲区的值并且内部开启一个新的缓冲区, 等待下一个`closingNotifier`的发送。

#### bufferTime - 特定时间周期内缓冲源 Observable 的值

将过往的值收集到数组中，并周期性地发出这些数组。

![img](F:\Objects\cn.laeni\blog-content\note\rx\rxjs.assets\bufferTime.png)

在一个特定的持续时间`bufferTimeSpan`内缓存源 Observable 的值。 除非指定了可选参数`bufferCreationInterval` , 它会发出数组并且重置缓冲区每个`bufferTimeSpan`毫秒。这个操作符会在每个 bufferCreationInterval 毫秒时开启缓冲区， 并在每个 bufferTimeSpan 毫秒时关闭(发出并重置)缓冲区。如果可选参数`maxBufferSize`被指定, 缓冲区会在`bufferTimeSpan`毫秒 之后或者缓冲区元素个数达到`maxBufferSize`时发出。
