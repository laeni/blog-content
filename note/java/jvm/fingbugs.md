# 常用操作

注: 很多命令可能无权限查看，所以将用户切到和程序同一用户操作。

## 查看进程资源占用

`top`

![image-20230720100156210](./fingbugs.assets/image-20230720100426802.png)

> PID: 进程号

## 查看某进程所有线程资源占用

```sh
$ top -H -p <PID> # 可简写为 top -Hp <PID>
# -p <PID> 指定进程号
# -H 线程模式
```

![image-20230720100156210](./fingbugs.assets/image-20230720100156210.png)

> PID: 线程号

## 查询某线程栈

`jstack -l <进程PID> | grep $(printf 'nid=0x%x' <线程PID>) --color -A 100`

![image-20230720101143876](./fingbugs.assets/image-20230720101143876.png)

![image-20230720101501035](./fingbugs.assets/image-20230720101501035.png)

> 如果没有堆栈的基本上就是GC线程了。

## 查看 GC 相关信息

`jstat -gc <进程PID>`

GC 信息除了可以通过 GC 日志查看，还可以通过`jstat`命令查看。

```
[imixxx@HXVM-BMS-01 tomcat]$ jstat -gc 43798
 S0C     S1C       S0U  S1U   EC        EU        OC         OU         PC        PU        YGC YGCT   FGC   FGCT      GCT
490880.0 502656.0  0.0  0.0   1785600.0 1785600.0 5592448.0  5592447.9  1048576.0 129885.0  41  10.431 1740  14941.005 14951.436
```

>- S0C: Young Generation第一个survivor space的内存大小 (kB).
>- S1C: Young Generation第二个survivor space的内存大小 (kB).
>- S0U: Young Generation第一个Survivor space当前已使用的内存大小 (kB).
>- S1U: Young Generation第二个Survivor space当前已经使用的内存大小 (kB).
>- EC: Young Generation中eden space的内存大小 (kB).
>- EU: Young Generation中Eden space当前已使用的内存大小 (kB).
>- OC: Old Generation的内存大小 (kB).
>- OU: Old Generation当前已使用的内存大小 (kB).
>- PC: Permanent Generation的内存大小 (kB)
>- PU: Permanent Generation当前已使用的内存大小 (kB).
>- YGC: 从启动到采样时Young Generation GC的次数
>- YGCT: 从启动到采样时Young Generation GC所用的时间 (s).
>- FGC: 从启动到采样时Old Generation GC的次数.
>- FGCT: 从启动到采样时Old Generation GC所用的时间 (s).
>- GCT: 从启动到采样时GC所用的总时间 (s).
>
>JDK8的结果稍微有所不同，结果含义可以参考：http://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html

## 拉取内存堆栈

```sh
jmap -dump:live,format=b,file=java_pid<pid>.hprof <pid>
```

# 相关工具

- [arthas](https://arthas.aliyun.com/) - 代码性能分析工具

# 参考文档

- [ JVM故障分析及性能优化系列之三：jstat命令的使用及VM Thread分析_"vm thread\" prio=10 tid= nid= runnable"](https://blog.csdn.net/a704397849/article/details/101351373)
- [必须要会的JVM性能监测工具（JVisualVM) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/339676111)