---
title: 'JDK 工具'
author: 'Laeni'
tags: 'Java, JVM'
date: '2023-03-19'
updated: '2023-03-19'
---

# javac - 编译 .java 源文件

[javac](https://docs.oracle.com/en/java/javase/17/docs/specs/man/javac.html)命令读取用 Java 编程语言编写的类和接口定义，并将它们编译为字节码类文件。 该命令还可以处理 Java 源文件和类中的注释。

```sh
# 编译 .java 源文件
$ javac Test.java
$ javac javac com/example/Test.java
# 默认情况下，编译器将每个类文件放在与其源文件相同的目录中。 可以使用 -d 选项指定一个目录存输出文件
$ javac -d dist/ com/example/Test.java
```

JDK 9 中引入了启动器环境变量`JDK_JAVAC_OPTIONS`，该变量将其内容附加到 javac 的命令行前。

```sh
$ JDK_JAVAC_OPTIONS='-d dist/' javac com/example/Test.java
```

**选项**（部分）

- `@filename` - 从文件中读取选项（除了`-J`选项）和文件名，以缩短或简化 `javac`命令。

  可以指定一个或多个包含 javac 命令参数的文件，这可以在任何操作系统上创建任何长度的 javac 命令。 

  - 示例1 - 将全部选项和源文件参数放在一个文件中

    ```sh
    $ cat <<EOF | tee argfile
    -d classes
    -g
    -sourcepath /java/pubs/ws/1.3/src/share/classes
    MyClass1.java
    MyClass2.java
    MyClass3.java
    EOF
    $ javac @argfile
    ```

  - 示例2 - 选项和源文件参数分开放

    ```sh
    $ cat <<EOF | tee options
    -d classes
    -g
    -sourcepath /java/pubs/ws/1.3/src/share/classes
    EOF
    $ cat <<EOF | tee classes
    MyClass1.java
    MyClass2.java
    MyClass3.java
    EOF
    $ javac @options @classes
    ```

  - 示例3 - 编译简单SpringBoot项目

    ```sh
    $ javac -d dist \
    --class-path ./spring-boot-3.0.4.jar:./spring-boot-autoconfigure-3.0.4.jar:./spring-context-6.0.6.jar \
    src/main/java/com/example/Application.java
    ```

  > 当`@filename`文件路径中含有空格时，必须使用单引号`'`或双引号`"`引用包含空白字符的参数
  >
  > ```
  > javac @"C:\white spaces\argfile" ...
  > javac "@C:\white spaces\argfile" ...
  > javac @C:\"white spaces"\argfile ...
  > ```

# javap - 反编译 .class 文件

[javap](https://docs.oracle.com/en/java/javase/17/docs/specs/man/javap.html) - 反汇编一个或多个类文件。

```sh
$ javap path/to/MyClass.class # 通过文件名指定
$ javap jar:file:///path/to/MyJar.jar!/mypkg/MyClass.class # 通过URL指定
$ javap java.lang.Object # 在类路径中能找到的完全限定类名
```

# jshell - Java Shell 工具

[jshell](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jshell.html) - 提供一种交互式评估 Java 编程语言的声明、语句和表达式的方法， 使学习语言、探索不熟悉的代码和 API 以及制作复杂代码的原型变得更加容易。 接受 Java 语句、变量定义、方法定义、类定义、导入语句和表达式。 输入的代码位称为片段。 

简单使用如下：

```sh
$ jshell \
  DEFAULT `# [预定义脚本]加载通常用作导入的默认条目`\
  JAVASE `# [预定义脚本]导入所有 Java SE 包`\
  PRINTING `# [预定义脚本]将 print、println 和 printf 定义为 jshell在工具中使用的方法`
|  欢迎使用 JShell -- 版本 17.0.4.1
|  要大致了解该版本, 请键入: /help intro
$ jshell> int a=4
a ==> 4
$ jshell> println(a)
4
$ jshell> int add(int i) {
   ...>     return i*i;
   ...> }
|  已创建 方法 add(int)
$ jshell> println(add(6))
36
```

# jar - 存档工具

[jar ](https://docs.oracle.com/en/java/javase/19/docs/specs/man/jar.html) - 为类和资源创建存档，并可以从存档中操作或恢复单个类或资源。jar 命令是一种基于 ZIP 和 ZLIB 压缩格式的通用归档和压缩工具。 

jar 命令语法示例 

```shell
# 创建一个`classes.jar`存档，包含两个类文件， `Foo.class`和 `Bar.class`. 
$ jar --create --file classes.jar Foo.class Bar.class
#创建一个`classes.jar`存档，通过使用现有清单， `mymanifest`，其中包含目录 foo/ 中的所有文件。 
$ jar --create --file classes.jar --manifest mymanifest -C foo/
#创建一个模块化 JAR 归档文件 foo.jar，其中模块描述符位于 classes/module-info.class 中。 
$ jar --create --file foo.jar --main-class com.foo.Main --module-version 1.0 -C foo/classes resources
#将现有的非模块化 JAR foo.jar 更新为模块化 JAR 文件。 
$ jar --update --file foo.jar --main-class com.foo.Main --module-version 1.0 -C foo/module-info.class
```

# 其他可能用到的工具

- [jarsigner](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jarsigner.html) - 对 Java 归档 （JAR） 文件进行签名和验证
- Java - 启动 [Java](https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html) 应用程序
- [javadoc](https://docs.oracle.com/en/java/javase/17/docs/specs/man/javadoc.html) - 从 Java 源文件生成 API 文档的 HTML 页面
- [jcmd](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jcmd.html) - 将诊断命令请求发送到正在运行的 Java 虚拟机 （JVM）
- [jconsole](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jconsole.html) - 启动图形控制台以监视和管理 Java 应用程序
- [JDB](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jdb.html) - 查找并修复 Java 平台程序中的错误
- [jdeprscan](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jdeprscan.html) - 静态分析工具，用于扫描 JAR 文件（或类文件的其他一些聚合）以使用已弃用的 API 元素
- [jdeps](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jdeps.html) - 启动 Java 类依赖关系分析器
- [JFR](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jfr.html) - 解析和打印飞行记录器文件
- [jhsdb](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jhsdb.html) - 附加到 Java 进程或启动事后调试器以分析崩溃的 Java 虚拟机 （JVM） 中的核心转储的内容
- [jinfo](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jinfo.html) - 为指定的 Java 进程生成 Java 配置信息
- [jlink](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jlink.html) - 将一组模块及其依赖项组装并优化为自定义运行时映像
- [JMAP](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jmap.html) - 打印指定进程的详细信息
- [jmod](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jmod.html) - 创建 JMOD 文件并列出现有 JMOD 文件的内容
- [jpackage](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jpackage.html) - 打包一个独立的 Java 应用程序
- [JPS](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jps.html) - 列出目标系统上已检测的 JVM
- [jrunscript](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jrunscript.html) - 运行支持交互和批处理模式的命令行脚本外壳
- [jstack](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jstack.html) - 为指定的 Java 进程打印 Java 线程的 Java 堆栈跟踪
- [jstat](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jstat.html) - 监视 JVM 统计信息
- [jstatd](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jstatd.html) - 监视已检测的 Java HotSpot VM 的创建和终止
- [keytool](https://docs.oracle.com/en/java/javase/17/docs/specs/man/keytool.html) - 管理加密密钥、X.509 证书链和受信任证书的密钥库（数据库）
- [rmid](https://docs.oracle.com/en/java/javase/17/docs/specs/man/rmid.html) - 启动激活系统守护程序，该守护程序允许在 Java 虚拟机 （JVM） 中注册和激活对象
- [RMIregistry](https://docs.oracle.com/en/java/javase/17/docs/specs/man/rmiregistry.html) - 在当前主机上的指定端口上创建并启动远程对象注册表
- [serialver](https://docs.oracle.com/en/java/javase/17/docs/specs/man/serialver.html) - 以适合复制到不断发展类的形式返回一个或多个类的“serialVersionUID”