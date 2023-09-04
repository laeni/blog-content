---
title: 'Java 9 新特性之 Java 模块'
author: 'Laeni'
tags: Java
date: '2023-03-25'
updated: '2023-03-27'
---

**Java 模块**是 JDK 9 中引入的新特性，它带来的好处有**强封装**、**可靠的配置**（有时候也称“安全性”，即在编译时和运行时进行模块依赖性检查，以确保不会在运行时出现缺失或冲突的依赖关系）、**[可扩展的平台](https://dev.java/learn/modules/intro/#scalable-platform)**。这里需要与我们平时说的 Maven 模块或 Gradle 模块区分开，并且 Java 模块也不是为了替代它们。更多关于 Java 模块的知识请参见[官网中关于模块的部分](https://dev.java/learn/modules)。

# 为什么需要模块

在学习 Java 模块之前，我们首先要知道它解决了什么问题，只有知道为什么要学它才能学好它！

前面已经已经列出了一些 Java 模块的一些作用，而这些作用中，我认为最重要的就是“增强封装性”。再谈论 Java 模块怎样增强封装性前，有必要回顾一下**什么是封装**。可能大家有想过这么一个问题：Java 既然提供了封装性来将一些类或类成员限定在一个访问范围内，从而获得一定的安全性，但与此同时 Java 又提供了反射 API 来在运行时访问那些被保护的成员，所以到底是“封装”是多余的还是“反射”是多余的？而这个问题的答案是显而易见的，即它们都是必须的且互不冲突的，因为我们在平时的代码中也会同时使用到它们，只是可能缺少对它们的一个理论认识。核心在于：“封装”不是“保密”，“封装”所保护的成员并不是绝对安全（也不需要决定安全），被封装的东西就像是一个密封透明盒子里的东西，我们知道它们，却不能直接使用它们。由于“封装”针对的是使用它们的人，即给他们传达一个意思：被封装的这些成员不是给你们用的，所以你们不应该直接用它们，也最好不要通过反射来使用。这可能是因为它们并不具备通用性，它们仅适用于某个特定的场景，所以不建议别人使用；又或者是该部分 API 在未来的版本可能随时改动甚至删除，在做这些改动时不会考虑外部项目使用的情况。如果他人执意使用，则需要自担风险。

说起“封装”，大家应该想到 Java 9 已有的权限修饰符（`public`、默认、`protected`和`private`），比如`private`修饰的成员只能被它自己访问，就连继承它的子类也不行，这就将`private`成员给“保护”起来，达到封装的目的。但是单凭这些权限修饰符是不够的，比如下面这个例子：

假设我们有一个类库，该类库需要将一个字符串的第二位更改为大写，这个操作很显然并不是通用方法，所以按照我的习惯，如果在一个类中有多处需要使用，我可能会将其作为该类的一个`private`方法；但是某一天我发现有多个相同角色的类都需要使用该方法，所以我可能会新创建一个公共父类，让所有需要使用该方法的类集成它，并将该函数移动到父类中，这时候如果要子类正常使用它们，我们至少需要将该方法的访问权限从`private`提升为`protected`；随着时间的推移，该方法需要在该项目中多个相关性不大的类中使用，这时候再让它们集成同一个类可能已经不合适了，所以我们可能会将该方法提取到一个特定的工具类中，而按照我们的习惯，工具类往往不会和使用它的类在以其或在在它们的父包，而是在一个例如`util`的独立包中，这时候该方法的访问权限就要再次提升到`public`。至此虽然该类库其他类都能使用该工具了，但同时不是该类库的类也能使用这个工具类了，这不是我们所希望的，因为该工具我们只打算给该类库使用，并不打算给依赖该类库的其他代码使用，否则在未来我们对该工具类进行重构的时候，我们可能还需要考虑该重构是否对类库的依赖者造成影响，即需要考虑兼容性问题。

关于上述问题，如果有了模块系统之后，我们只需要将我们认为需要公开的包导出，其他未导出的包，即使里面的类是`public`的，其他代码也不能访问，这就是 Java 模块的增强封装特性。

# 模块声明/模块描述

模块声明是模块的核心，位于一个具有定义模块所有属性的名为`module-info.java`的文件。该文件推荐放在所有工具最容易识别它们的位置，即项目的源根文件夹中，通常是`src/main/java`。

模块声明包含**模块名称**、**依赖**、**导出的包**和**使用和提供的服务**四部分，但并不是所有指令都需要用上，而时根据自身的需要使用即可。完整的模块声明如下：

```java
module $NAME {
    // 依次列出本模块依赖的其他模块:
    requires $MODULE;
    // 默认情况下，依赖仅在本模块生效，但通过此方式依赖的模块可以向上传递
    requires transitive $MODULE;
    // 导入可选模块，通过此方式依赖的模块仅仅在编译时有效，运行时会被忽略
    requires static $MODULE;

    // 依次列出本模块导出的包:
    exports $PACKAGE;
    // 根允许某些特定的模块使用此包
    exports $PACKAGE to $MODULE;

    // 依次列出允许其他模块通过反射访问的包:
    opens $PACKAGE;
    // 根允许某些特定的模块使用通过反射访问此包
    opens $PACKAGE to $MODULE;

    // 依次列出本模块使用到的服务:
    // $TYPE 为权限定名，一般为接口或在抽象类，但是也可以是普通类
    uses $TYPE;

    // 依次列出本模块实现的服务提供者:
    // $CLASS 一定是实现或继承了 $TYPE，且该服务类型可以来自该模块自身
    provides $TYPE with $CLASS;
}
```

例如，JDBC API的平台的[`java.sql`](https://docs.oracle.com/en/java/javase/20/docs/api/java.sql/module-summary.html)模块定义如下：

```java
// 模块名称为 java.sql
module java.sql {
    // 通过 requires 指令导入它需要的其他模块，其中 requires transitive 表示该导入具有传递性，即依赖 java.sql 模块的模块也自动依赖这些模块，而无需再次通过 requires 声明
    requires transitive java.logging;
    requires transitive java.transaction.xa;
    requires transitive java.xml;

    // 表示导出包给其他依赖本模块的模块使用
    exports java.sql;
    exports javax.sql;

    // 表示本模块需要使用 java.sql.Driver 服务，类似于以前的接口或抽象类声明
    uses java.sql.Driver;
}
```

如果一个 JAR 是模块化的，则可以在 IDE 的帮助下查看 JAR 中的`module-info.class`文件来查询模块详情，或通过`jar --describe-module --file $FILE`命令查看模块详情。

> 为什么不将`requires`更改为`imports`来与导出的`exports`对应?
> 答：因为`requires`针对的是模块，而`exports`针对的是包，且代码中已经使用`import`来导入包了。

## 模块名称

模块名称与包名称具有相同的规则： 

- 允许使用的字符包括`A-Z`,`a-z`,`0-9`,`_`,`$`和`.`，其中`.`用作隔开符号。
- 按照惯例，模块名称都是小写的，并且`$`仅用于机械生成的代码。
- 名称应该是全球唯一的 

关于模块名称的唯一性，建议与包相同： 选择一个与项目关联的 URL 并将其反转以得出模块名称的第一部分，然后从那里进行优化。（这意味着这两个示例模块与域 example.com 相关联。） 如果将此过程应用于模块名称和包名称，前者通常是后者的前缀，因为模块比包更通用。这绝不是必需的，而是表明名称选择得当的指标。

## requires 依赖

`requires`指令按模块名称列出所有直接依赖项。关于子依赖项中使用`requires transitive`依赖的具有传递性的依赖要不要再次声明将在后续**传递依赖**中讨论。

这里的模块依赖项列表很可能与构建配置中列出的依赖项（比如 Maven 依赖项）非常相似。这通常会引发这样的疑问：模块依赖项是否是多余的或者应该自动生成？首先它不是多余的，因为模块名称不包含版本也不包含构建工具获取 JAR 所需的信息（如组 ID 和工件 ID），这些信息由构建配置列出而不是模块名称。其次，虽然可以通过给定的 JAR 推断出模块名称（更多详细信息参见后面**自动模块**部分），但考虑到对平台模块以及`static`和`transitive`修饰的模块很复杂，所以不能根据构建配置自动生成模块声明，且模块中**安全性**就是解决某些情况下构建依赖缺失的问题的。但是在未来，可能可以使用模块配置代替构建配置中的依赖项部分，毕竟很多语言（如 Python、Node.js、Golang）都有官方推出的依赖管理系统，而 Java 却没有。

## 导出和打开包

默认情况下所有类（包括`public`修饰的）只能在模块内部访问。要使模块外部的代码能够访问某个类，需要**导出**（`exports`）或**打开**（`opens`）包含该类的包。

关于*导出*和*打开*包要点是： 

- 导出包中的`public`类型和成员在编译和运行时可用。
- 打开包中的所有类型和成员仅可以在运行时通过反射访问。

以下是来自两个不同模块的`exports`和`opens`指令的示例： 

```java
// 来自 module java.sql
exports java.sql;
exports javax.sql;

// 来自 com.example.app
opens com.example.app.entities;
```

这表明`java.sql`导出了一个和模块同名的包以及`javax.sql`包。然而该模块还有其他更多的包，但它们不是其 API 的一部分，使用的人无需关心，所以不导出。`com.example.app`模块不导出包，这是正常的，因为用于启动应用程序的模块一般不会被其他模块依赖，因此没有人调用它，但它打开`com.example.app.entities`包给其他模块通过反射来访问，从名字上看可能是因为它包含其他模块（比如 JPA）想要通过反射与之交互的实体。

`exports`和`opens`指令有对应的[变体 ](https://dev.java/learn/modules/qualified-exports-opens/)允许用于仅将包导出/打开到特定模块，相关内容在后续讨论。

根据经验，尝试尽可能少的导出包（就像保持字段私有一样），仅在需要时使方法包可见或公开，以及默认情况下使类包可见，并且仅在另一个包中需要时才公开。这减少了在其他地方可见的代码量，从而降低了复杂性。

## 使用服务和提供服务

有专门关于[服务](#服务)的章节，但现在可以仅需要了解可以使用服务将 API 的使用者与 API 的实现分离，从而更容易在启动应用程序时替换它。如果模块使用类型（接口或类）作为服务，则需要在模块声明中使用`uses`指令指明具体服务，其中包括类的完全限定名称。提供服务的模块也要在它们的模块声明中表达了它们自己的哪些类型可以做到这一点（通常通过实现或扩展它）。

下面是 lib 和 app 两个示例模块：

```java
// 来自 com.example.lib
uses com.example.lib.Service;

// 来自 module com.example.app
provides com.example.lib.Service
    with com.example.app.MyService;
```

lib 模块使用`Service`，该服务是它自己中的其中一个类，app 模块依赖 lib 模块，并为 lib 模块提供`MyService`。在运行时，lib 模块将访问所有实现/扩展`Service`类型的类，方法是调用`ServiceLoader.load(Service.class)` API。这意味着 lib 模块执行在 app 模块中定义的行为，即使 lib 模块不依赖 app 模块，这对于理清依赖关系并使模块专注于它们的关注点非常有用。

# 相关概念

## Modular JARs - 模块化JAR

是指遵循 Java 模块系统标准的JAR包，这些JAR包包含了模块描述文件（`module-info.class`或`module-info.java`）。Java模块系统可以根据这些描述文件进行模块的自动装配和加载，从而实现模块化的开发和构建。

## Module Path - 模块路径

*模块路径*是一个与*类路径*平行的新概念，它是工件（JAR 或字节码文件夹）和包含工件的目录的列表。模块系统使用它来定位运行时（JRE）中未找到的所需模块，因此通常是所有应用程序、库和框架模块。模块系统将模块路径上的所有工件（甚至可以是普通的非模块化 JAR）变成模块，使它们变成[自动模块，从而实现增量模块化 ](https://dev.java/learn/modules/automatic-module/)。`javac`和`java`以及其他与模块相关的命令都理解并能处理模块路径。

**注：**一个 JAR 是否模块化并不能决定它是否被视为一个模块！因为类路径上的所有 JAR 统一被视为一个[未命名模块](https://dev.java/learn/modules/unnamed-module/) ，模块路径上的所有 JAR 都变成了单个独立的模块。这意味着项目负责人可以决定哪些依赖项最终会成为单独的模块，哪些不会（与依赖项的维护者相反）。

## Module Resolution and Module Graph - 模块解析和模块图

**Module Graph**（模块图）是 Java 模块系统根据初始模块中的模块声明中`requires`进来的模块，如果找到则继续解析找到的模块的模块声明，一直重复这个过程，将找到的模块已经它们的依赖关系构建出的关系图，模块在途中称为**节点**，两个节点之间的线称为**可读性边**（还有[其他创建边缘的方法](https://dev.java/learn/modules/implied-readability/)），而该过程被称为**Module Resolution（模块解析）**。*模块解析*一般发生在编译期和运行期，且两个时期中模块系统解析模块依赖的策略不同，详情参见[可选依赖项的解析](#可选依赖项的解析)。可以在命令行添加`--show-module-resolution`选项查看模块解析过程。

>如果一个模块在解析完成后没有进入模块图，那么即使该模块在模块路径上也是不可用的。

## 基础模块

base模块（`java.base`）包含像`Class`和`ClassLoader`这样的类，像`java.lang`和`java.util`的包，以及整个模块系统，所有模块都隐式依赖它，如果没有它，JVM 上的程序将无法运行，因此无需明确依赖该模块，因为对`java.base`模块的依赖是默认的。

由于及基础模块包含模块系统，所以实际上在模块解析时最开始解析的不是初始模块，而时基本模块，因为模块系统必须先加载基础模块并自行引导之后才能进行模块解析工作，只不过一般情况都忽略它。

## 服务绑定

参见[服务](#模块解析期间的服务)章节。

# 使用反射访问打开的包和打开的模块

模块系统的强封装也适用于反射，所以反射已经失去了闯入内部 API 的“超能力”。当然，反射是 Java 生态系统的重要组成部分，因此模块系统具有支持反射的特定指令。它允许打开特定包，使这些包在编译时无法访问，但可以在运行时进行反射访问。

## 为什么导出的包不适合反射访问

1. 一般情况下`exports`指令导出的包是该模块公共 API 的一部分。其他模块直接使用这些类型并传达一定程度的稳定性。这中较为稳定且直接通过编码方式使用的类型通常不适合处理 HTTP 请求或与数据库交互的类（如 JPA 中的实体）。
2. 在模块系统中，即使在导出的包中，反射也只能访问公共类型的公共成员。但是依赖于反射的框架通常会访问非公共类型、构造函数、访问器或字段，由于这些包没有打开，所以反射访问这些非公共的类成员时仍然会失败。

打开包和打开模块就是专门设计用于解决这两点问题的。当然，如果确实有需要，一个包可以同时被导出和打开。总之，需要反射访问的包需要使用**打开**而不是**导出**。

## 打开包

模块中可以使用`opens`指令在模块声明中*打开包*给其他模块进行反射：

```java
module com.example.app {
    opens com.example.entities;
}
```

在编译时，仅打开的包被完全封装，就好像`opens`指令不存在一样。这意味着`*com.example.app*`模块外部使用`com.example.entities`包中的类型的代码将无法编译。

在运行时，打开的包的类型可用于反射。这意味着反射可以自由地与所有类型和成员进行交互（但是对非公共成员的访问还是和以前一样需要使用[`AccessibleObject.setAccessible()`](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/lang/reflect/AccessibleObject.html#setAccessible(boolean))修改权限之后才能访问）。

## 打开模块

如果有一个大模块，其中包含许多需要打开的包，单独打开每个包可能会很麻烦，这时候可以通过`open`直接打开整个模块的中的所有包：

```java
open module com.example.entities {
    // ...
}
```

由于打开的模块会打开它包含的所有包，所以再次打开单个包是没有意义的，如果这样做会导致编译错误。

# 可选依赖项

默认情况下，依赖项需要在编译和运行时都存在。但是，有些代码是针对运行时不一定存在的项目编写的（比如最常见是**Spring Boot**自动配置，有些代码仅在相关依赖存在的情况下才执行）。`requires static`指令依赖的模块在编译时要求存在但在运行时容忍不存在来解决此问题：

```java
module com.example.app {
    requires static com.sample.solver;
}
```

## 可选依赖项的解析

模块解析是从根模块开始，通过解析`requires`指令构建模块图的过程，详情可参考[Module Resolution and Module Graph - 模块解析和模块图](#Module Resolution and Module Graph - 模块解析和模块图)章节。但是，模块系统在编译时和运行时处理可选依赖项的策略不同。

在编译时，可选依赖项就像常规依赖项一样，即所有可选依赖项都会被加入到模块图中。

在运行时，当模块系统遇到`requires static`指令时会直接忽略它，就像该指令不存在一样，所以可选依赖项也不会进入模块图中。除非有其他模块以常规依赖项进行依赖，或者用`--add-modules`命令行标志显式添加时，它们才会进入模块图。

## 针对可选依赖项进行编码

### 检查模块是否存在

一般来说，当正在执行的代码引用某个类型时，Java 运行时会检查该类型是否已加载。如果没有，它会通过类加载器加载该类，如果加载失败则会抛出`NoClassDefFoundError`异常，该异常通常会使应用程序崩溃或至少在正在执行的逻辑块中失败。

在模块系统中，模块系统在启动时会检茶模块声明，确保所有常规模块都存在才能启动。而对于通过`requires static`指令依赖的可选模块，可能需要我们自己在代码运行时手动检测：

```java
public class ModuleUtils {
    public static boolean isModulePresent(Object caller, String moduleName) {
        return caller.getClass()
                .getModule()
                .getLayer()
                .findModule(moduleName)
                .isPresent();
    }
}
```

调用方需要将自身传递给方法，以便它可以确定要查询所需模块的正确层：

```java
if (ModuleUtils.isModulePresent(this, "com.example.lib")) {
    // 存在 com.example.lib 模块
}
```

### 已建立的依赖项

但是，可能并不总是需要显式检查模块的存在。假设有一个`com.example.lib`库，它有很多功能，其中包括使用`java.sql`模块的 JDBC API，然后，如果该库的使用者不使用 JDBC，那么该库的这部分 JDBC 相关的功能永远不会被使用，这意味如果该部分功能被使用了，那么`java.sql`模块一定存在，这种情况下可以不检查。

即，如果使用可选依赖项的代码只会从依赖于相同依赖项的代码调用，则可以假定它的存在，不需要检查它的存在。

# 传递依赖项

模块系统对访问其他模块有严格的规则，其中之一是访问模块必须可读取被访问的模块，即对被访问模块有可读性。建立可读性的最常见方式是一个模块`requires`另一个模块，但这不是唯一的方式。如果 B 模块在其 API 中使用了 A 模块中的类型，那么每个使用 B 模块的模块都必须同时`requires` A 模块。除非 B 模块使用`requires transitive A;`，这意味着任何读取 B 模块的模块都可以读取 A 模块，即任何对 B 模块有可读性的模块都对 A 模块有隐含可读性。

## 隐含可读性

在常见情况下，模块内部使用的依赖项对外界一无所知。举个例子，[`java.prefs`](https://docs.oracle.com/en/java/javase/20/docs/api/java.prefs/module-summary.html)模块`requires java.xml`，因为它需要 XML 解析功能，但它自己的 API 既不接受也不返回`java.xml`包中的类型。但是有些时候，依赖项并不完全是内部的，而是存在于模块之间的边界上，在这种情况下，一个模块依赖另一个模块，并在其自己的公共 API 中公开依赖其他模块中的类型。

举个例子：

```java
// 来自 A 模块
module a {
    exports org.example.a;
}
// 来自 org.example.a.A
public class A {

    private final String name;

    public A(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

// 来自 B 模块
module b {
    requires a;
    exports org.example.b;
}
// 来自 org.example.b.B
public class B {
    public static A newA() {
        return new A("Example");
    }
    
    public static void setA(A a) {
        // ...
    }
}

// 来自 C 模块
module c {
    requires b;
}
// 来自 org.example.b.C
public class Main {
    public void test1() {
        String name = B.newA().getName();
        System.out.println(name);
    }

    public void test2() {
        A a = B.newA(); // 无法编译通过
        String name = B.newA().getName();
        System.out.println(name);
    }
}
```

在这个例子中，由于`b`模块使用`requires a;`引入`a`模块，但是在`b`模块的导出 API 中使用了来自`a`模块中的类型，这就导致`c`模块如果想使用`b`模块中的部分 API（如`B.setA(A)`）时必须要在`c`模块也申明对`a`模块的依赖，否则没办法时用`a`模块的类型，进而也无法使用`b`模块，这也是上面`c`模块中的`test2`方法中的`A a = B.newA();`语句无法编译通过的原因。然而，虽然可以在`c`模块中`requires a;`来解决，但是识别和手动解决此类隐藏的依赖项将是一项繁琐且容易出错的任务。

为了解决上述问题，*隐含可读性的*就有了它的用武之地。它扩展了模块声明，以便一个模块可以向依赖于它的任何模块授予它所依赖的模块的可读性。这种隐含可读性一般也叫依赖传递，只不过这种传递不是一直往上传递，而时仅传递到直接使用该模块的模块就停止了。语法为：`require transitive MODULE_NAME`。

所以可以将`b`模块中`requires a;`更改为`requires transitive a;`即可，这样在`b`模块中依赖的`a`模块就隐式传递到`c`模块中。

## 何时使用隐含可读性

模块系统的原始解释器包含何时使用隐含可读性的明确建议：

> 通常，如果一个模块导出的包中包含签名引用第二个模块中的包的类型，则第一个模块的声明应包括对第二个模块的`requires transitive`依赖关系。这将确保依赖于第一个模块的其他模块能够自动读取第二个模块，从而访问该模块导出包中的所有类型。

但如果一个模块隐含可读，时不是不需要明确`requires`它？比如在`java.sql`模块的例子中（该模块`requires transitive java.logging;`），`java.sql`模块的使用者是否需要依赖`java.logging`模块？毕竟从技术上讲，不需要这样的声明，而且似乎是多余的。

要回答这个问题，我们必须看看`java.sql`模块的使用者究竟如何使用`java.logging`模块中的类型。如果只需要读取`java.logging`模块中的类型，然后调用（例如更改记录器的日志级别，仅此而已），即`java.sql`模块的使用者与`java.logging`模块的交互发生在它们交互的附近（这称之为两个模块之间的**边界**）。但类型的使用范围也可能超越边界，比如，`java.sql`模块的使用者除了与`java.sql`模块交互外可能自身就需要使用`java.logging`模块。

所以，如果隐含读取的类型仅用于其模块（例如*java.sql*）的边界，则建议仅依赖模块（例如*java.logging*）的隐含可读性。否则，即使不是严格需要，也应该明确`requires`。

# [限定`exports`和`opens`](https://dev.java/learn/modules/qualified-exports-opens/)

模块系统允许模块导出和打开包，使外部代码可以访问它们，在这种情况下，每个读取导出/打开模块的模块都可以访问这些包中的类型。这意味着对于一个包，我们必须在强封装或让每个人都可以访问它之间做出选择。然而有时候并不容易在这二者之间作出选择，所以模块系统提供了限定的`exports`和`opens`指令的变体仅授予特定模块访问权限。

## 限定导出/打开包

`exports`指令可以通过跟`to $MODULES`来*限定*导出范围，`$MODULES`是以逗号分隔的目标模块名称列表。对于`exports to`指令中指定的模块，包将完全像常规`exports`指令一样可访问。对于其他模块，包将被紧密封装，就好像根本没有`exports`指令一样。`opens`指令也是如此，也可以用`to $MODULES`具有相同的效果：对于目标模块，包是开放的；对于其他模块，它被强烈封装。

JDK 本身就有很多限定导出的示例，比如`java.xml`模块。

关于编译的两个说明： 

- 如果声明了限定导出/打开的目标模块在编译时找不到，编译器会发出警告。但不会报错退出，因为提到的目标模块对编译来说不是必需的。
- 不允许同时对一个包中使用`exports`和`exports to`指令，否则会导致编译错误。

并且有两个细节需要指出：

- 目标模块可以依赖导出/打开的模块，从而创建一个循环。
- 每当新模块需要访问限定的导出包时，都需要更改包所属的模块，以便提供对这个新模块的访问权限。虽然让导出模块控制谁可以访问包就是限定导出的意义，但这样做可能很麻烦。

## 何时使用限定导出

如前所述，限定导出的目的是控制哪些模块可以访问相关包。但什么时候使用限定导出？一般来说，如果某个包需要在一组模块之间共享，但不希望在除这些模块之外公开时使用（比如一个框架包含多个包，其中有一个`utli`包，而这个包只希望在这个框架之间使用）。

这与引入模块系统之前隐藏工具类的问题是对称的。一旦工具类必须跨包可用，它就必须是公开的，但在 Java 9 之前，这意味着所有其他代码都可以访问它。强封装解决了这个问题，它允许我们使公共类在模块外不可访问。

现在我们处于类似的情况，我们想要隐藏一个包（以前是一个类）但是一旦它必须跨模块（以前是一个包）可用，它就必须被导出（公开），因此可以被所有其他模块（以前是所有其他类）访问，这是适合使用限定的的地方。限定导出允许模块在它们之间共享包而不使其普遍可用。这对于包含多个模块并希望在客户端无法使用的情况下共享代码的库和框架非常有用。对于想要限制对特定 API 的依赖性的大型应用程序也会派上用场。

## 何时使用限定打开 

限定导出可以防止同事和用户引入内部 API，而限定打开的目标模块通常是框架。无论你是面向所有模块打开一个包还是仅面向 Hibernate 打开，Spring 都不会根据它启动。因此，限定打开的适用场景比限定导出的场景小得多。

限定打开的一个缺点是，在框架开始采用基于`Lookup`/`VarHandle`的方法（允许“转发”反射访问）之前，必须始终将包打开到执行实际反射的确切模块。因此，在规范和实现分开的情况下（例如，JPA 和 Hibernate），您可能会发现自己必须打开实体包到实现（例如 Hibernate 模块）而不是 API（例如 JPA 模块）。如果你的项目试图坚持标准并避免在代码中提及所有实现，那将是不幸的。

总而言之，打开反射包的一个好的默认方法是不限定访问权限，除非您的项目对自己的代码使用大量反射，在这种情况下，其好处与限定导出的好处相似。只对框架开放似乎不值得麻烦，并且在需要针对特定实现模块的情况下可能应该完全避免。

综上所述，打开包进行反射的一个很好的方法是默认不限定访问权限，除非项目对其自己的代码使用大量反射，在这种情况下，其好处与限定导出的好处相似。而只对框架打开是没必要的，特别是在规范与实现分离的情绪下更是应该避免这样做。

# 服务

在 Java 中，通常将 API 建模为接口（有时是抽象类），然后根据情况选择最佳实现。理想情况下，API 的使用者与实现完全分离，这意味着它们之间没有直接依赖关系。Java 的服务加载器 API 允许将这种方法应用于 JAR（模块化或非模块化），并且模块系统将其作为一等概念与模块声明中的`uses`和`provides`指令集成在一起。

## Java 模块系统中的服务 

### 问题示例

让我们从一个*在三个模块中使用这三种类型*的示例开始： 

- 类`Main`在`com.example.app`中 
- 接口`Service`在`com.example.api`中
- 类`Implementation`（实现`Service`接口) 在`com.example.impl`中 

`Main`想用`Service`，但需要创建`Implementation`才能得到`Service`实例： 

```java
public class Main {

    public static void main(String[] args) {
        Service service = new Implementation();
        use(service);
    }

    private static void use(Service service) {
        // ...
    }

}
```

这导致以下模块声明： 

```java
module com.example.api {
    exports com.example.api;
}

module com.example.impl {
    requires com.example.api;
    exports com.example.impl;
}

module com.example.app {
    // dependency on the API: ✅
    requires com.example.api;
    // dependency on the implementation: ⚠️
    requires com.example.impl;
}
```

如您所见，使用接口来分离 API 的使用者和提供者的挑战在于：在某些时候必须实例化特定的实现。如果这是作为常规构造函数调用发生的（如`Main`)，则会创建对实现的依赖关系（`com.example.app`中的`requires com.example.impl;`），从而创建了两个模块之间的依赖。这就是服务解决的问题。 

### 解决方案之服务定位器模式

Java 通过实现[服务定位器模式](https://en.wikipedia.org/wiki/Service_locator_pattern)来解决此问题，其中[`ServiceLoader`](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/ServiceLoader.html)类充当中央注册表（这是它的工作原理）。 

服务是一种可访问的类型（不一定是接口；抽象类甚至具体类也可以），一个模块想要使用它，另一个模块提供以下实例：

- *使用*服务的模块必须在其模块描述符中使用`uses $SERVICE`指令来表达其要求，其中`$SERVICE`是服务类型的完全限定名称。
- *提供服务*的模块必须使用`provides $SERVICE with $PROVIDER`指令来进行申明，其中`$SERVICE`与`uses`指令中的类型相同，并且`$PROVIDER`是另外一个类的完全限定名称，该类需要满足以下要求：
  - *扩展或实现的具体*`$SERVICE`类并且有一个公共的、无参数的构造函数（称为*提供者构造函数* ） 
  - *或*具有公共、静态、无参数的`provide`方法，该方法返回值类型必须扩展或实现`$SERVICE`类型（称为 *提供者方法* ）

在运行时，依赖模块可以使用`ServiceLoader`类并调用`ServiceLoader.load($SERVICE.class)`来获取服务的所有提供的实现，然后模块系统将返回一个`ServiceLoader<$SERVICE>`。您可以通过各种方式使用它来访问服务提供商，[`ServiceLoader`](https://docs.oracle.com/en/java/javase/20/docs/api/java.base/java/util/ServiceLoader.html)的*Javadoc*对此进行了详细说明（与服务相关的所有其他内容）。 

### 解决方案示例

以下是我们之前研究的三个类和模块如何使用服务。我们从模块声明开始：

```java
module com.example.api {
    exports com.example.api;
}

module com.example.impl {
    requires com.example.api;

    provides com.example.api.Service
        with com.example.impl.Implementation;
}

module com.example.app {
    requires com.example.api;

    uses com.example.api.Service;
}
```

请注意`com.example.app`不再需要`com.example.impl`。相反，它声明它使用`Service`，并且`com.example.impl`声明它提供了`Implementation`。此外，`com.example.impl`不再导出`com.example.impl`包。服务加载器不要求服务实现可以在模块外部访问，如果该包中的其他类在其他类不需要导出，则我们可以不导出它。这是服务的额外好处，因为它可以减少模块的 API 面。 

就是这样`Main`可以得到一个`Service`示例: 

```java
public class Main {

    public static void main(String[] args) {
        Service service = ServiceLoader
            .load(Service.class)
            .findFirst()
            .orElseThrow();
        use(service);
    }

    private static void use(Service service) {
        // ...
    }

}
```

### 一些 JDK 服务 

JDK 本身也使用服务。例如，包含 JDBC API 的`java.sql`模块使用`java.sql.Driver`作为服务： 

```java
module java.sql {
    // requires...
    // exports...
    uses java.sql.Driver;
}
```

这也表明一个模块可以使用它自己的类型作为服务。

JDK 中服务的另一个示例是`java.lang.System.LoggerFinder`的用法。这是 API 的一部分，允许用户将 JDK 的日志消息（而不是运行时的！）通过管道传输到他们选择的日志框架（例如，Log4J 或 Logback）中。 简而言之，JDK 不是写入标准输出，而是使用`LoggerFinder`创建`Logger`实例，然后用它们记录所有消息。因为它使用`LoggerFinder`作为一项服务，日志框架可以提供它的实现。

```java
module com.example.logger {
    // `LoggerFinder` is the service interface
    provides java.lang.System.LoggerFinder
        with com.example.logger.ExLoggerFinder;
}

public class ExLoggerFinder implements System.LoggerFinder {

    // `ExLoggerFinder` must have a parameterless constructor

    @Override
    public Logger getLogger(String name, Module module) {
        // `ExLogger` must implement `Logger`
        return new ExLogger(name, module);
    }

}
```

## 模块解析期间的服务

如果你曾经使用`--show-module-resolution`命令行选项启动过简单的模块化应用程序并观察模块系统究竟在做什么，你可能会对已解析的平台模块数量感到惊讶。对于一个足够简单的应用程序，唯一的平台模块应该是`java.base`，也许还有一两个，那么为什么还有那么多其他的模块呢？*服务*就是答案。

从[模块系统基础知识](https://dev.java/learn/modules/intro/)中说过，只有在模块解析期间进入模块图的模块在运行时才可用。为确保服务的所有提供者都是这种情况，解析过程需要考虑到`uses`和`provides`指令。因此，除了跟踪依赖关系之外，一旦它解析了使用服务的模块，它还会将所有提供该服务的模块添加到模块图中。这个过程称为**服务绑定**。

# 类路径上的代码 - 未命名模块

模块系统希望一切都是模块，因为那样它可以统一应用其规则，但与此同时，并不强制模块化。调和这两个看似矛盾的要求的机制是[未命名模块](https://dev.java/learn/modules/unnamed-module/)。未命名模块包含类路径中的所有类，并应用了一些特殊规则，完成这些工作后，它就可以像其他普通模块一样工作。

这意味着如果您从类路径启动代码，未命名模块将起作用。除非您的应用程序相当小，否则它可能需要增量模块化，这涉及混合 JAR 和模块、类路径和模块路径。 这使得理解模块系统的“类路径模式”如何工作变得很重要。 

## 未命名模块

所有“非模块化类”构成**未命名模块**，*非模块化类*条件如下： 

- 在编译阶段，不包含模块描述符进行编译的类。
- 在编译和运行时，从类路径加载所有类（即使该 Jar 是已经编译好的模块，只要来自类路径就是非模块化的类）。

所有模块都具有三个核心属性，未命名模块也是如此： 

- 模块名称：未命名的模块没有名称，这是有意义的，因为这意味着没有其他模块可以在其声明中使用它（例如`requires`它） 。
- 依赖：未命名的模块读取（`requires`）进入模块图中的所有模块。 
- 导出/打开包：未命名模块导出它的所有包并[打开它们以供反射访问](https://dev.java/learn/modules/opening-for-reflection/)。

与未命名模块相反，其他模块都被称为**命名模块**。`ServiceLoader`将获取`META-INF/services`下的服务提供者列表。 

## 混乱的类路径 

未命名模块的主要目的是获取类路径的内容并使其在模块系统中工作。由于类路径上的 JAR 之间一直都没有边界，因此现在建立它们没有任何意义，因此整个类路径只有一个未命名的模块。在其中，就像在类路径上一样，所有公共类都可以相互访问，并且包可以跨 JAR 拆分。 

未命名模块具有独特的作用和专注于向后兼容的特点，因此具有一些特殊属性。其中之一是在Java 9到16中间不时访问[强封装API](https://dev.java/learn/modules/strong-encapsulation/)的机会。另一个是它不会受到应用于命名模块的许多检查的限制。因此，分布在它和其他模块之间的包不会被发现，而类路径部分也不可用。这意味着，如果同一包也存在于命名模块中，则可能会因为缺少实际存在于类路径上的类而出现错误。

一个有点违反直觉且容易出错的细节是未命名模块的确切构成。模块化 JAR 变成模块似乎很明显，因此普通 JAR 则进入未命名模块，实际真的是这样吗？而事实并非如此，*类路径上的所有 JAR*统一称为未命名模块，不管这些 JAR 是否模块化。因此，模块化 JAR 不一定作为模块加载，主要还是看该 JAR 在类路径上还是模块路径上！因此，假如一个库开始交付模块化 JAR，它的用户也不会被迫将它们作为模块使用，因为他们可以将它们留在类路径中，在类路径上的代码被捆绑到未命名的模块中。这允许生态系统几乎相互独立地模块化。 

要尝试这一点，您可以将以下两行代码放入一个打包为模块化 JAR 的类中： 

```java
String moduleName = this.getClass().getModule().getName();
System.out.println("Module name: " + moduleName);
```

当从类路径启动时，输出为`Module name: null`，表明该类最终出现在未命名的模块中。从模块路径启动时，会得到预期的`Module name: $MODULE`，`$MODULE`是该模块的名字。

## 未命名模块的解析

未命名模块与模块图中的其他模块相互关系的一个重要因素是它可以读取哪些其他模块。如前所述，就是这些被纳入到模块图中的模块。但是哪些模块会被纳入呢？从[模块系统基础知识](https://dev.java/learn/modules/intro/)中可以了解到，模块解析通过从根模块（尤其是初始模块）开始，然后迭代地添加所有它们的直接和传递依赖项来构建模块图。如果正在编译的代码或应用程序的主方法在未命名模块中，例如从类路径启动应用程序时，那么该怎么办呢？毕竟，普通的JAR文件不表示任何依赖关系。

如果初始模块是未命名的模块，则模块解析将从一组预定义的根模块开始。根据经验，这些是在运行时（JRE）找到的模块，但实际规则更详细一些：

- 对于`java.*`模块，哪些`java.*`成为根模块取决于`java.se`模块（即表示整个 Java SE API 的模块；它存在于完整的 JRE 映像中，但可能不存在于通过`jlink`创建的自定义运行时映像中）是否存在: 
- 如果`java.se`存在，它将成为根模块。
  - 如果不存在，则所有至少导出一个包且不进行限定的`java.*`模块都会成为根模块。 

- `java.*`之外的其他运行时（JRE）模块，运行时中不是孵化模块且至少导出一个包且不进行限定的包的模块都会成为根模块。这些模块主要与`jdk.*`模块相关。 

- 通过[`--add-modules`](https://dev.java/learn/modules/add-modules-reads/)选项模块列出的模块始终是根模块。 

请注意，将未命名模块作为初始模块，根模块集始终是运行时（JRE）映像中包含的模块的子集。模块路径上存在的模块永远不会被解析，除非明确使用`--add-modules`选项添加。如果手工制作模块路径以包含应用所需要的其他模块，则可以通过`--add-modules ALL-MODULE-PATH`[将这些这些类路径上的模块全部添加为根模块](https://dev.java/learn/modules/add-modules-reads/)。 

## 取决于未命名模块

模块系统的主要目标之一是可靠的配置：模块必须表达其依赖关系，并且模块系统必须能够保证它们的存在。 我们讨论了具有模块描述符的显式模块，但是如果我们尝试将可靠配置扩展到类路径会发生什么？ 

模块系统的主要目标之一是可靠的配置：模块必须表达其依赖关系，模块系统必须能够保证它们的存在。 对于带有模块描述符的显式模块，我们讨论了这个问题，但是如果我们尝试将可靠配置扩展到类路径会发生什么？

### 思想实验 

想象一下模块可能依赖于类路径的内容，可能有类似的东西 `requires class-path`在他们的描述符中。 模块系统可以为这种依赖提供什么保证？ 事实证明，几乎没有。 只要类路径上至少有一个类，模块系统就必须假定依赖关系已实现。 那不会很有帮助。 更糟糕的是，它会严重破坏可靠的配置，因为你可能最终依赖于一个模块 `requires class-path`.   但这几乎不包含任何信息—— *究竟* 需要在类路径上做什么？ 

进一步旋转这个假设，假设两个模块 *com.example.framework* 和 *com.example.library* 依赖于同一个第三个模块，比如 SLF4J。   一个在 SLF4J 模块化之前声明了依赖关系，因此 `requires class-path`，另一个声明它依赖于模块化的 SLF4J，因此 `requires org.slf4j`.   现在，依赖于 *com.example.framework* 和 *com.example.library* 的人会在哪条路径上放置 SLF4J JAR？   无论他们选择哪个，模块系统都必须确定两个传递依赖项之一未满足。 

仔细思考会得出这样的结论：如果您想要可靠的模块，那么依赖任意类路径内容并不是一个好主意。 由于这个确切的原因，没有 `requires class-path`. 

### 因此，无名 

那么如何最好地表达最终持有类路径内容的模块不能被依赖呢？   在使用名称引用其他模块的模块系统中？   不给那个模块一个名字，让它 *unnamed*  ，可以这么说，听起来很合理。   你有它：   未命名模块没有名称，因为任何模块都不应该在 `requires`指令 - 或任何其他指令，就此而言。 没有 `requires`，没有可读性优势，没有该优势，模块无法访问未命名模块中的代码。 

总之，对于依赖于工件的显式模块，该工件必须在模块路径上。 这很可能意味着您将普通 JAR 放在模块路径上，这会将它们变成自动模块——我们接下来将探讨的概念。 

# 什么时候建议进行模块化

# 不建议进行模块化

如果一个Spring Boot应用程序只有一个模块，那么这个模块就是默认模块。默认模块不需要使用`module-info.java`文件进行模块化描述，因为默认模块可以访问类路径上的所有类和资源。

虽然理论上可以将默认模块进行模块化描述，但是这并不是必须的，并且也不推荐这么做。因为将默认模块进行模块化描述可能会导致一些问题，例如在某些情况下可能会导致应用程序无法启动或无法访问类路径上的类和资源。

# 模块相关的命令

1. 查看模块详情
   ```sh
   $ jar --describe-module --file $FILE
   ```

2. 启动模块应用程序

   ```sh
   # modules are in `app-jars` | initial module is `com.example.app`
   $ java --module-path app-jars --module com.example.app
   # or 简化命令
   $ java -p app-jars -m com.example.app
   ```

# 模块替代方法

1. 如果某些类不希望外部使用，我们可以通过类注释甚至是包注释加以说明。

