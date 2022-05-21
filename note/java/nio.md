---
title: 'Java Nio 笔记'
author: 'Laeni'
tags: 'Java, Nio, Netty, 异步'
date: '2021-09-25'
updated: '2021-09-25'
---

[原文链接](http://tutorials.jenkov.com/java-nio/overview.html)   **作者：**Jakob Jenkov   **译者：**airu   **校对：**丁一

Java NIO 由以下几个核心部分组成：

- Channels
- Buffers
- Selectors

Channel，Buffer 和 Selector 构成了核心的API。其它组件，如Pipe和FileLock，只不过是与三个核心组件共同使用的工具类。

## Channel 和 Buffer

[Channel](http://docs.oracle.com/javase/7/docs/api/java/nio/channels/Channel.html) 是 NIO 基本的结构，它代表了一连接。现在，把 Channel 想象成一个可以“打开”或“关闭”，“连接”或“断开”和作为传入和传出数据的运输工具。

基本上，所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。 数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。

![img](http://ifeve.com/wp-content/uploads/2013/06/overview-channels-buffers1.png)

Channel和Buffer有好几种类型。下面是JAVA NIO中的一些主要Channel的实现：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

正如你所看到的，这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。

与这些类一起的有一些有趣的接口，但为简单起见，我尽量在概述中不提到它们。本教程其它章节与它们相关的地方我会进行解释。

以下是Java NIO里关键的Buffer实现：

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。

Java NIO 还有个 MappedByteBuffer，用于表示内存映射文件， 我也不打算在概述中说明。

## Callback (回调)

Netty 内部使用回调处理事件时。一旦这样的回调被触发，事件可以由接口 ChannelHandler 的实现来处理。如下面的代码，一旦一个新的连接建立了，调用 channelActive()，并将打印一条消息。

```java
public class ConnectHandler extends ChannelInboundHandlerAdapter {
    // 当建立一个新的连接时调用 ChannelActive()
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(
                "Client " + ctx.channel().remoteAddress() + " connected");
    }
}
```

## Future

Future 提供了另外一种通知应用操作已经完成的方式。这个对象作为一个异步操作结果的占位符,它将在将来的某个时候完成并提供结果。用起来感觉和 Callback 刚刚相反，因为 Callback 是由调用者生成传递为被调用者，Future 则是由被调用者生成返回给调用者，类似于 javascript 中的 Promise 。

JDK 附带接口 java.util.concurrent.Future ,但所提供的实现只允许手动检查操作是否完成或阻塞了。这是很麻烦的，所以 Netty 提供自己了的实现,ChannelFuture,用于在执行异步操作时使用。

ChannelFuture 提供多个附件方法来允许一个或者多个 ChannelFutureListener 实例。这个回调方法 operationComplete() 会在操作完成时调用。事件监听者能够确认这个操作是否成功或者是错误。如果是后者,我们可以检索到产生的 Throwable。简而言之, ChannelFutureListener 提供的通知机制不需要手动检查操作是否完成的。

每个 Netty 的 outbound I/O 操作都会返回一个 ChannelFuture;这样就不会阻塞。这就是 Netty 所谓的“自底向上的异步和事件驱动”。

下面例子简单的演示了作为 I/O 操作的一部分 ChannelFuture 的返回。当调用 connect() 将会直接是非阻塞的，并且调用在背后完成。由于线程是非阻塞的，所以无需等待操作完成，而可以去干其他事，因此这令资源利用更高效。

```java
Channel channel = ...;
//不会阻塞 - 异步连接到远程地址
ChannelFuture future = channel.connect(new InetSocketAddress("192.168.0.1", 25));
```

下面代码描述了如何利用 ChannelFutureListener 。首先，连接到远程地址。接着，通过 ChannelFuture 调用 addListener() 来 注册一个新ChannelFutureListener。当监听器被通知连接完成，我们检查状态。如果是成功，就写数据到 Channel，否则我们检索 ChannelFuture 中的Throwable。

注意，错误的处理取决于你的项目。当然,特定的错误是需要加以约束 的。例如,在连接失败的情况下你可以尝试连接到另一个。

```java
Channel channel = ...;
//1.异步连接到远程对等节点。调用立即返回并提供 ChannelFuture
ChannelFuture future = channel.connect(new InetSocketAddress("192.168.0.1", 25));
//2.操作完成后通知注册一个 ChannelFutureListener
future.addListener(new ChannelFutureListener() {
    @Override
    public void operationComplete(ChannelFuture future) {
        //3.当 operationComplete() 调用时检查操作的状态。
        if (future.isSuccess()) {
            //4.如果成功就创建一个 ByteBuf 来保存数据。
            ByteBuf buffer = Unpooled.copiedBuffer("Hello", Charset.defaultCharset());
            //5.异步发送数据到远程。再次返回ChannelFuture。
            ChannelFuture wf = future.channel().writeAndFlush(buffer);
            // ...
        } else {
            //6.如果有一个错误则抛出 Throwable,描述错误原因。
            Throwable cause = future.cause();
            cause.printStackTrace();
        }
    }
});
```

## Event 和 Handler

Netty 使用不同的事件来通知我们更改的状态或操作的状态。这使我们能够根据发生的事件触发适当的行为。

这些行为可能包括：

- 日志
- 数据转换
- 流控制
- 应用程序逻辑

由于 Netty 是一个网络框架,事件很清晰的跟入站或出站数据流相关。因为一些事件可能触发传入的数据或状态的变化包括:

- 活动或非活动连接
- 数据的读取
- 用户事件
- 错误

出站事件是由于在未来操作将触发一个动作。这些包括:

- 打开或关闭一个连接到远程
- 写或冲刷数据到 socket

每个事件都可以分配给用户实现处理程序类的方法。这说明了事件驱动的范例可直接转换为应用程序构建块。

图1.3显示了一个事件可以由一连串的事件处理器来处理

**Figure 1.3 Event Flow**

![Figure%201](https://atts.w3cschool.cn/attachments/image/20170808/1502159113476213.jpg)

Netty 的 ChannelHandler 是各种处理程序的基本抽象。想象下，每个处理器实例就是一个回调，用于执行对各种事件的响应。

在此基础之上，Netty 也提供了一组丰富的预定义的处理程序,您可以开箱即用。比如，各种协议的编解码器包括 HTTP 和 SSL/TLS。在内部,ChannelHandler 使用事件和 future 本身,创建具有 Netty 特性抽象的消费者。

## 整合

### FUTURE, CALLBACK 和 HANDLER

Netty 的异步编程模型是建立在 future 和 callback 的概念上的。所有这些元素的协同为自己的设计提供了强大的力量。

拦截操作和转换入站或出站数据只需要您提供回调或利用 future 操作返回的。这使得链操作简单、高效,促进编写可重用的、通用的代码。一个 Netty 的设计的主要目标是促进“关注点分离”:你的业务逻辑从网络基础设施应用程序中分离。

### SELECTOR, EVENT 和 EVENT LOOP

Netty 通过触发事件从应用程序中抽象出 Selector，从而避免手写调度代码。EventLoop 分配给每个 Channel 来处理所有的事件，包括

- 注册感兴趣的事件
- 调度事件到 ChannelHandler
- 安排进一步行动

该 EventLoop 本身是由一个线程驱动，它给一个 Channel 处理所有的 I/O 事件，并且在 EventLoop 的生命周期内不会改变。这个简单而强大的线程模型消除你可能对你的 ChannelHandler 同步的任何关注，这样你就可以专注于提供正确的回调逻辑来执行。该 API 是简单和紧凑。

## Selector

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。

这是在一个单线程中使用一个Selector处理3个Channel的图示：

![img](http://ifeve.com/wp-content/uploads/2013/06/overview-selectors.png)

要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。

## 概览、组件

- 一个服务器 handler：这个组件实现了服务器的业务逻辑，决定了连接创建后和接收到信息后该如何处理
- Bootstrapping： 这个是配置服务器的启动代码。最少需要设置服务器绑定的端口，用来监听连接请求。

## API

### ChannelInboundHandler

- channelRead() - 每个信息入站都会调用

- channelReadComplete() - 通知处理器最后的 channelread() 是当前批处理中的最后一条消息时调用

- exceptionCaught() - 读操作时捕获到异常时调用

  覆盖 exceptionCaught 使我们能够应对任何 Throwable 的子类型。在这种情况下我们记录，并关闭所有可能处于未知状态的连接。它通常是难以 从连接错误中恢复，所以干脆关闭远程连接。当然，也有可能的情况是可以从错误中恢复的，所以可以用一个更复杂的措施来尝试识别和处理 这样的情况。

  如果异常没有被捕获，会发生什么？

  每个 Channel 都有一个关联的 ChannelPipeline，它代表了 ChannelHandler 实例的链。适配器处理的实现只是将一个处理方法调用转发到链中的下一个处理器。因此，如果一个 Netty 应用程序不覆盖exceptionCaught ，那么这些错误将最终到达 ChannelPipeline，并且结束警告将被记录。出于这个原因，你应该提供至少一个 实现 exceptionCaught 的 ChannelHandler。

  关键点要牢记：

  - ChannelHandler 是给不同类型的事件调用
  - 应用程序实现或扩展 ChannelHandler 挂接到事件生命周期和 提供自定义应用逻辑。

#### ChannelInboundHandlerAdapter



---

本文摘录自<https://www.w3cschool.cn/essential_netty_in_action/>

