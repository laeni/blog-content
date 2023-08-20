# JVM 参数

# Java在线工具开发

- 通过GC日志自动生成JVM优化参数

## Native Memory Tracking (NMT)

可以使用 NMT 来追踪了解 JVM 的内存（ JVM Memory）使用详情，帮助我们排查内存增长与内存泄漏相关的问题。

如果要开启 NMT，需要添加 JVM 参数（`-XX:NativeMemoryTracking=[off | summary | detail]`）来开启，这里的取值就像日志级别一样，`detail`为详细级别，`summary`为简要级别，`off`为关闭，当开起高级别时，可以通过 jcmd 获取该级别及其低级别数据。

开启后可以通过`jcmd <pid | main class> VM.native_memory <option>`（支持的"option"可参考`jcmd <pid | main class> help VM.native_memory`）拿到运行中程序的 NMT 数据，也可以通过增加 JVM 参数（`-XX:+UnlockDiagnosticVMOptions -XX:+PrintNMTStatistics`）使程序退出时答应 NMT 数据。

### 参考文档

- [Native Memory Tracking 详解（1）:基础介绍](https://heapdump.cn/article/4644018)

# Java工具

## jcmd

### Help

```sh
$ jcmd -h
Usage: jcmd <pid | main class> <command ...|PerfCounter.print|-f file>
   or: jcmd -l
   or: jcmd -h

  命令必须是所选 jvm 的有效 jcmd 命令。
  使用命令“help”查看可用的命令。
  如果 pid 为 0，命令将发送到所有 Java 进程。
  main class 参数将用于匹配（部分或完全）用于启动 Java 的类。
  如果未给出选项，则列出 Java 进程（与 -l 相同）。

  PerfCounter.print 显示此进程公开的计数器
  -f  从文件中读取并执行命令
  -l  列出本地机器上的 JVM 进程
  -? -h --help print this help message
```

#### VM.native_memory

输出本地内存使用：

```
jcmd <pid> VM.native_memory [summary | detail | baseline | summary.diff | detail.diff | shutdown] [scale= KB | MB | GB]
```

jcmd 操作 NMT 选项如下表所示：

| **jcmd NMT 选项** | **说明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| summary           | 打印按类别汇总的摘要信息                                     |
| detail            | 打印按类别汇总的内存使用情况打印虚拟内存映射打印按 call site 汇总的内存使用情况 |
| baseline          | 创建一个新的内存使用状况的快照，用以进行比较                 |
| summary.diff      | 根据上一个 baseline 基线打印新的 summary 对比报告            |
| detail.diff       | 根据上一个 baseline 基线打印新的 detail 对比报告             |
| shutdown          | 停止NMT                                                      |

> - NMT 默认打印的报告是 KB 来进行呈现的，为了满足我们不同的需求，我们可以使用 `scale=MB | GB` 来更加直观的打印数据。
> - 创建 baseline 之后使用 diff 功能可以很直观地对比出两次 NMT 数据之间的差距。

# 堆栈分析相关文档

- [(58条消息) 获取JVM堆内存转储的常用方法_jmap dump内存的命令是_铁锚的博客-CSDN博客](https://blog.csdn.net/renfufei/article/details/108785603)
- [(58条消息) Eclipse Memory Analyzer(MAT)使用方法_memoryanalyzer怎么使用_瓜头居的博客-CSDN博客](https://blog.csdn.net/lgd0602/article/details/123593284)
- [内存溢出排查基本步骤 - 老王子H - 博客园 (cnblogs.com)](https://www.cnblogs.com/laowz/p/10096757.html)
- 

# 相关文档

- [Java 语言和虚拟机规范](https://docs.oracle.com/javase/specs/index.html)
- <https://dev.java/learn/>

