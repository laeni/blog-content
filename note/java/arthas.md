---
title: Arthas
author: Laeni
tags: Java
date: 2024-03-20
updated: 2024-03-20
---

# 快速安装

```bash
curl -O https://arthas.aliyun.com/arthas-boot.jar
```

# 启动

```bash
java -jar arthas-boot.jar [PID]
```

# 常用操作

## 线程

- 打印线程 ID 为 1 的栈

  ```sh
  thread 1
  ```

- 查找 JVM 里已加载的类

  格式: `sc [-d] <类名>`（类名支持通配符），如：

  ```sh
  sc -d *MathGame
  ```

  > 如果搜索的是接口，还会搜索所有的实现类。
  >
  > 通过`-d` 参数，可以打印出类加载的具体信息。

- 查找类的具体函数

  格式: `sm [-d] <类名> [函数名]`，如：

  ```sh
  sc -d *MathGame
  ```

  > 通过`-d` 参数，可以打印函数的具体属性。

  查找特定的函数，比如查找构造函数：

  ```bash
  sm java.math.RoundingMode <init>
  ```

- 查询某类的 **classLoaderHash**

  ```sh
  sc -d *MathGame | grep classLoaderHash
  ```

- 查看函数的参数/返回值/异常信息

  ```sh
  watch demo.MathGame primeFactors returnObj
  ```

- 获取 class 实例

  ```bash
  vmtool --action getInstances --className demo.MathGame --limit 10
  ```

  > 此示例是获取`demo.MathGame`的实例，最多显示`10`条记录。

- 调用实例方法

  ```shell
  vmtool --action getInstances --className demo.MathGame --express 'instances[0].primeFactors(3)' -x 3
  ```

  > 调用`demo.MathGame`实例的`primeFactors`方法。

- 打印所有的 **System Prop** 信息

  ```bash
  sysprop
  ```

  > 也可以指定单个 key： `sysprop java.version`
  >
  > 也可以通过`grep` 来过滤： `sysprop | grep user`
  >
  > 可以设置新的 value： `sysprop testKey testValue`

- 获取到**环境变量**

  ```bas
  sysenv
  ```

  > 其他操作和 `sysprop` 命令类似

- 打印出**`JVM`** 的各种详细信息

  ```bash
  jvm
  ```

- **反编译**代码

  格式1: `jad <类名>`，如：

  ```sh
  jad demo.MathGame
  ```

  通过`--source-only` 参数可以只打印出在反编译的源代码：

  ```bash
  jad --source-only com.example.demo.arthas.user.UserController
  ```
  
- 编译原代码为字节码 -  (Memory Compiler)

  ```bash
  mc --classLoaderHash 1efbd816 /tmp/UserController.java -d /tmp
  mc -c 1efbd816 /tmp/UserController.java -d /tmp
  ```

  当只有一个 ClassLoader 实例时可以直接指定 ClassLoader 的类名：

  ```bash
  mc --classLoaderClass org.springframework.boot.loader.LaunchedURLClassLoader /tmp/UserController.java -d /tmp
  ```

- 重新加载字节码

  ```bash
  redefine -c 1efbd816 /tmp/com/example/demo/arthas/user/UserController.class
  ```

  > 只有 JVM 已经存在该字节码时才能使用 `redefine` 重新加载，如果需要增加新的类，可以将类对应的字节码直接放在类路径即可。

- 跟踪调用

  跟踪所有 `javax.servlet.Filter` 的调用。

  ```bash
  trace javax.servlet.Filter *
  ```

  

