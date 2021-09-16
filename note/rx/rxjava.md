---
title: 'RxJava简单使用笔记'
author: 'Laeni'
tags: 'RxJava, ReactiveX, Observables'
date: '2021-08-11'
updated: '2021-08-11'
---

RxJava 是[Reactive Extensions](http://reactivex.io/)的 Java VM 实现：一个使用可观察序列组合异步和基于事件的程序的库。

## 基础类

RxJava 3 具有几个基础类：

- [`io.reactivex.rxjava3.core.Flowable`](http://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Flowable.html): 0..N 流，支持反应流和背压
- [`io.reactivex.rxjava3.core.Observable`](http://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Observable.html): 0..N 流，不支持背压
- [`io.reactivex.rxjava3.core.Single`](http://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Single.html): 只能发送1个数据或者一个错误
- [`io.reactivex.rxjava3.core.Completable`](http://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Completable.html)：没有发送任何数据，但只处理 onComplete 和 onError 事件
- [`io.reactivex.rxjava3.core.Maybe`](http://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Maybe.html)：能够发射0或者1个数据，要么成功，要么失败

## 术语

### 上游（Upstream），下游（Downstream）

RxJava 中的数据流由一个源、零个或多个中间步骤组成：

```java
source
  .operator1()
  .operator2()
  .operator3()
  .subscribe(consumer)
```

在这里，对于`operator2`来说，在它前面叫做**上游**。在它后面的叫做**下游**。

### 流对象（Objects in motion）

在 RxJava 的文档中，**emission**、**emits**、**item**、**event**、**signal**、**data**和**message**被认为是同义词，代表数据流中被传递的数据对象。

### 背压（Backpressure）

当上下游在不同的线程中，通过`Observable`发射、处理、响应数据流时，如果上游发射数据的速度快于下游接收处理数据的速度，这样对于那些没来得及处理的数据就会造成积压，这些数据既不会丢失，也不会被垃圾回收机制回收，而是存放在一个异步缓存池中，如果缓存池中的数据一直得不到处理，越积越多，最后就会造成内存溢出，这便是响应式编程中的背压（`Backpressure`）问题。

为此，`RxJava`带来了`Backpressure`的概念。背压是一种流量的控制步骤，在不知道上流还有多少数据的情形下控制内存的使用，表示它们还能处理多少数据。背压是指在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略。

在`Rxjava1.0`中，有的`Observable`支持背压，有的不支持，为了解决这种问题，2.0把支持背压和不支持背压的`Observable`区分开来，专用`Flowable`类用于支持背压，`Observable`专用于非背压操作（短序列、GUI 交互等）。其他类型`Single`，`Maybe`和`Completable`不支持且不应该支持背压。

在订阅的时候如果使用`FlowableSubscriber`，那么需要通过`s.request(Long.MAX_VALUE)`去主动请求上游的数据项。如果遇到背压报错的时候，`FlowableSubscriber`默认已经将错误`try-catch`，并通过`onError()`进行回调，程序并不会崩溃；在订阅的时候如果使用`Consumer`，那么不需要主动去请求上游数据，默认已经调用了`s.request(Long.MAX_VALUE)`。如果遇到背压报错、且对`Throwable`的`Consumer`没有`new`出来，则程序直接崩溃；

背压策略的上游的默认缓存池是128。

背压策略：

```java
public enum BackpressureStrategy {
    /**
     * 不做任何操作。
     * onNext事件的写入没有任何缓冲或丢弃。 下游必须处理任何溢出。
     * 当应用自定义参数 onBackpressureXXX 运算符之一时很有用。
     */
    MISSING,
    /**
     * 如果下游无法跟上，则发出 MissingBackpressureException。
     */
    ERROR,
    /**
     * 缓冲所有onNext值，直到下游使用它。
     */
    BUFFER,
    /**
     * 如果下游跟不上，则丢弃最近的onNext值。
     */
    DROP,
    /**
     * 仅保留最新的onNext值，如果下游无法跟上，则覆盖任何先前的值。
     */
    LATEST
}
```

### 线程调度器（Schedulers）

对于Android开发者而言，RxJava最简单的是通过调度器来方便地切换线程。在不同平台还有不同的调度器，例如我们Android的主线程：AndroidSchedulers.mainThread()。

调度器	功能
AndroidSchedulers.mainThread()	需要引用rxandroid, 切换到UI线程
Schedulers.computation()	用于计算任务，如事件循环和回调处理，默认线程数等于处理器数量
Schedulers.io()	用于IO密集型任务，如异步阻塞IO操作，这个调度器的线程池会根据需求，它默认是一个CacheThreadScheduler
Schedulers.newThread()	为每一个任务创建一个新线程
Schedulers.trampoline()	在当前线程中立刻执行，如当前线程中有任务在执行则将其暂停， 等插入进来的任务执行完成之后，在将未完成的任务继续完成。
Scheduler.from(executor)	指定Executor作为调度器

### 事件调度器

RxJava事件发出去并不是置之不顾，要有合理的管理者来管理它们，在合适的时机要进行释放事件，这样才不会导致内存泄漏，这里的管理者我们称为事件调度器(或事件管理者)CompositeDisposable。

### Observables的"热"和"冷"

Observable什么时候开始发射数据序列？这取决于Observable的实现，一个"热"的Observable可能一创建完就开始发射数据，因此所有后续订阅它的观察者可能从序列中间的某个位置开始接受数据（有一些数据错过了）。一个"冷"的Observable会一直等待，直到有观察者订阅它才开始发射数据，因此这个观察者可以确保会收到整个数据序列。

在一些ReactiveX实现里，还存在一种被称作Connectable的Observable，不管有没有观察者订阅它，这种Observable都不会开始发射数据，除非Connect方法被调用。

### 组装时间

通过应用各种中间运算符来准备数据流发生在所谓的**组装时间**：

```java
Flowable<Integer> flow = Flowable.range(1, 5)
.map(v -> v * v)
.filter(v -> v % 3 == 0)
;
```

此时，数据还没有流动，也没有发生任何副作用。

### 订阅时间

这是在`subscribe()`内部建立处理步骤链的流程上调用时的临时状态：

```java
flow.subscribe(System.out::println)
```

这是触发**订阅副作用**的时间（请参阅 参考资料`doOnSubscribe`）。在这种状态下，某些来源会立即阻止或开始发射物品。

### 运行

这是流主动发出项目、错误或完成信号时的状态：

```java
Observable.create(emitter -> {
     while (!emitter.isDisposed()) {
         long time = System.currentTimeMillis();
         emitter.onNext(time);
         if (time % 2 != 0) {
             emitter.onError(new IllegalStateException("Odd millisecond!"));
             break;
         }
     }
})
.subscribe(System.out::println, Throwable::printStackTrace);
```

实际上，这是上面给定示例的主体执行的时间。

### 其他

- Reactive 直译为反应性的，有活性的，根据上下文一般翻译为反应式、响应式
- **Observable (可观察对象/异步数据流):** 表示一个概念，这个概念是一个可调用的未来值或事件的集合。
- **Observer (观察者):** 一个回调函数的集合，它知道如何去监听由 Observable 提供的值。
- **Subscription (订阅):** 表示 Observable 的执行，主要用于取消 Observable 的执行。
- **Operators (操作符):** 采用函数式编程风格的纯函数 (pure function)，使用像 `map`、`filter`、`concat`、`flatMap` 等这样的操作符来处理集合。
- **Subject (主体):** 相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式。
- **Schedulers (调度器):** 用来控制并发并且是中央集权的调度员，允许我们在发生计算时进行协调，例如 `setTimeout` 或 `requestAnimationFrame` 或其他。
- bscriber 负责处理事件，他是事件的消费者
- subscribeOn(): 指定 subscribe() 所发生的线程
- subscribe - 订阅
- observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程
- observe - 观察
- emit - 直译为发射，发布，发出，含义是Observable在数据产生或变化时发送通知给Observer，调用Observer对应的方法
- items - 直译为项目，条目，在Rx里是指Observable发射的数据项

# 常用操作符

## 创建操作

Create, Defer, Empty/Never/Throw, From, Interval

### Just  <->  of

等同于RxJs中的of操作符，意思是将当前时间点已经存在值包装为集合（流）。

```java
Flowable.just("hello word").subscribe(System.);
```

### Range

生成指定范围的集合。

```java
Flowable.range(1, 5).subscribe(System.out::println);
// 这种情况下两种方式产生的效果相同
Flowable.just(1, 2, 3, 4, 5).subscribe(System.out::println);
```

Repeat, Start, Timer

## 变换操作

Buffer, FlatMap, GroupBy, Map, Scan和Window

## 过滤操作

Debounce, Distinct, ElementAt, Filter, First, IgnoreElements, Last, Sample, Skip, SkipLast, Take, TakeLast

## 组合操作

And/Then/When, CombineLatest, Join, Merge, StartWith, Switch, Zip

## 错误处理

Catch和Retry

## 辅助操作

Delay, Do, Materialize/Dematerialize, ObserveOn, Serialize, Subscribe, SubscribeOn, TimeInterval, Timeout, Timestamp, Using

## 条件和布尔操作

All, Amb, Contains, DefaultIfEmpty, SequenceEqual, SkipUntil, SkipWhile, TakeUntil, TakeWhile

## 算术和集合操作

Average, Concat, Count, Max, Min, Reduce, Sum

## 转换操作

To

## 连接操作

Connect, Publish, RefCount, Replay

## 反压操作

用于增加特殊的流程控制策略的操作符



# 参考

* [Rxjava3文档级教程一： 介绍和基本使用](https://blog.csdn.net/LucasXu01/article/details/105279367)
* [Rxjava3文档级教程二： 操作符全解](https://blog.csdn.net/LucasXu01/article/details/105312731)
* [Rxjava3文档级教程三： 实战演练](https://blog.csdn.net/LucasXu01/article/details/105318137)

