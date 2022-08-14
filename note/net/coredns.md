# Coredns 使用笔记

## 快速上手

### 本地启动

```sh
$ ./coredns -conf ./Corefile
```

### Docker启动

```sh
docker run --rm -it \
  --name coredns \
  -p 53:53/UDP \
  -p 53:53/TCP \
  -v /etc/resolv.conf:/etc/resolv.conf \
  coredns/coredns:1.9.3 -conf ./Corefile
```

## 概念

1. 区域（zone）

   区域是DNS服务器的管辖范围, 是由DNS名称空间中的单个区域或由具有上下隶属关系的紧密相邻的多个子域组成的一个管理单位. 因此, DNS名称服务器是通过区域来管理名称空间的,而并非以域为单位来管理名称空间,但区域的名称与其管理的DNS名称空间的域的名称是一一对应的.

   一台DNS服务器可以管理一个或多个区域, 而一个区域也可以由多台DNS服务器来管理(例如:由一个主DNS服务器和多个辅助DNS服务器来管理). 在DNS服务器中必须先建立区域, 然后再根据需要在区域中建立子域以及在区域或子域 中添加资源记录,才能完成其解析工作.

## 配置

配置基本上插件的配置，所以在此之前，最好先对插件有一定的了解。

一般会使用文件`Corefile`来配置 CoreDNS。当 CoreDNS 启动的时候，如果 `-conf` flag 没有被配置，就会在当前目录查找 `Corefile` 文件。

最简单的配置：

```
. {
}
```

配置文件包含了一个或者多个服务器 (Server Blocks)，。每个服务器块列出了一个或多个插件。那些插件也可以在后面使用指令配置。

在Corefile 文件中，插件的顺序不决定插件链的顺序。 插件执行的顺序，配置在[`plugin.cfg`](https://github.com/coredns/coredns/blob/master/plugin.cfg)文件中。

此外，在配置时，需要检查配置中用到的插件是否已经被编译进程序中，使用`coredns -plugins`查询插件列表。

### 环境变量

CoreDNS 在配置中支持环境变量。
 环境变量可以被使用在任何地方。语法是 `{$ENV_VAR}` ( Windows-类型的语法`{%ENV_VAR%}` 也是支持的)。CoreDNS 会在解析Corefile的时候替换这些pwdpwd变量内容。

### 导入其他文件

参考 [import](https://coredns.io/plugins/import/) plugin。这个插件有些特殊，可以被用在Corefile的任何地方。

支持导入文件、glob pattern (`*`) 和代码片段。

#### 导入代码片段

一个代码片段通过命名一个块（block）的特殊语法来定义。名字需要被放到圆括号内: `(name)`。然后，它就可以随着导入插件放置到配置文件的任何地方了。

```corefile
# define a snippet
(snip) {
    prometheus
    log
    errors
}

. {
    whoami
    import snip
    import config/common.conf
}
```

------

### 服务器块 Server Blocks

每个服务器块（Server Block）以server应该伺候的zones开头。在zone名字或者zone列表名（以空格分隔）之后，一个服务器块以大括号作为开头和结束。
 如下的服务器块定义了一个 server，负责root zone: `.`下所有的zones； 基本上，这个 server 应该处理所有的查询：

```corefile
. {
    # Plugins defined here.
}
```

服务器块（Server blocks）还可以指定监听端口。默认端口是 53 (DNS 服务标准端口)。指定端口，以冒号作为分隔符在zone后列出端口号。
 如下的 Corefile 指示 CoreDNS 创建一个 Server ， 监控端口 1053：

```corefile
.:1053 {
    # Plugins defined here.
}
```

> 注意： 如果你明确的定义了监听端口，你就不可以使用 `-dns.port`参数覆盖了。

如果有多个服务器块有着相同的定义：`zone:port`，则coredns会在启动的时候报错。

```txt
.:1054 {

}

.:1054 {

}
```

变更第二个端口为 1055 可以让这两个服务器块变成两个不同的服务器。

#### 规定协议

当前 CoreDNS 接受4种协议: DNS, DNS over TLS (DoT), DNS over HTTP/2 (DoH)
 and DNS over gRPC。可以通过在服务器配置文件，在zone 前加个前缀来指定服务器接收哪种协议。

- `dns://` for plain DNS (the default if no scheme is specified).
- `tls://` for DNS over TLS, see [RFC 7858](https://www.rfc-editor.org/rfc/rfc7858).
- `https://` for DNS over HTTPS, see [RFC 8484](https://www.rfc-editor.org/rfc/rfc8484).
- `grpc://` for DNS over gRPC.

## 插件

每个服务块都定义了一系列插件，最简单的方式，就是在服务器块内添加插件的名字。

大多数插件允许以指令（和插件名在同一行的其他配置）提供配置，比如chaos插件；如果插件有更多配置，可以使用插件块（Plugin Block）来配置更多配置，效果和服务块一样以大括号开头和结束：

```txt
. {
    plugin {
       # Plugin Block
    }
}
```

下面的 Corefile 设置 4 zones 运行于2个不同的端口：

```corefile
coredns.io:5300 {
    file db.coredns.io
}

example.io:53 {
    log
    errors
    file db.example.io
}

example.net:53 {
    file db.example.net
}

.:53 {
    kubernetes
    forward . 8.8.8.8
    log
    errors
    cache
}
```

当CoreDNS解析配置文件的时候，就会是如下的 Setups：

![img](https://upload-images.jianshu.io/upload_images/18908711-47a78749ba68349e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1056/format/webp)

当 CoreDNS 启动并且加载配置文件了，DNS服务就启动了。服务由zone和运行port来定义。每个服务器都有它自己的插件链。

> 这里的"服务"可以理解为多个进程，我结合上图（来源于网络）的理解是：coredns监听了多少个端口就开启了多少个服务，每个服务都可以处理一个或多个域（zone）。

### 插件（处理）查询类型

当一个查询提交到 CoreDNS，会执行如下的步骤：

1. 如果有多个服务器运行在同一个监听端口，它会检查哪个zone最匹配该次查询（最长后缀匹配）
    比如：如果存在两个服务器，一个为 `example.org` ，另一个为 `a.example.org`，查询请
    求是 `www.a.example.org`，CoreDNS会将查询请求路由向后者。
2. 当服务器被发现后，查询通过该服务器预先定义的插件链，路由向该服务器。这些总是按照同样的顺序处理的。顺序定义在  [`plugin.cfg`](https://github.com/coredns/coredns/blob/master/plugin.cfg)。
3. 每个插件都会检查查询，并且判断是否被处理 (有些查询允许基于query name 或其他属性过滤).
   会产生如下操作:
   1. query 被插件处理.
   2. query 不被插件处理
   3. query 被插件处理， 但是处理失败了，还是需要调用链中的下一个插件。我们称为 `fallthrough`。
   4. query 被插件处理， "hint" 被添加，下个插件被调用。 这个hint 提供了一个办法来 "see" the (eventual) response and act upon that.

处理一个 query 意思是 插件会向client返回消息。

#### Query 被处理

插件处理这个query。它找到 looks up (or 生成， or 插件拥有者决定插件做的任何事情) 一个反馈，然后发送回 client 。然后查询的处理就在这结束了，不会调用下个插件了。 以如此方式工作的一个 (简单) 插件，比如 [*whoami*]().

#### Query 不被处理

如果插件决定不处理这个query，它就直接调用链中的下一个插件。如果链中的最后一个插件也觉得不处理这个查询，CoreDNS 就会返回 SERVFAIL 给 client。

#### Query 以 Fallthrough 的方式处理

在某种情况下，一个插件完整的处理这个查询，但是答复来自后台 (i.e. maybe it got NXDOMAIN)，它希望链中的其他插件查询。 如果*fallthrough*被配置了，下一个插件将会被调用。

以这种方式工作的插件，比如 [hosts](https://coredns.io/plugins/hosts/) 插件。首先，先从主机列表文件 (`/etc/hosts`) 进行查询，如果查到了，就将其返回；如果没有查询到，它会 *fallthrough* 到下一个，以希望其他插件可以返回结果给 client 。

> *fallthrough* :  下坠。CoreDNS里面的意思，应该是自己不处理，让下个插件接替这个任务。

#### Query 以 Hint 的方式处理

这类插件处理查询，总是调用下一个插件。但是，它提供了一个 hint ，允许查看到被写回 client 的响应内容。
比如插件 [prometheus](https://coredns.io/plugins/metrics/)。它计量某个周期的时间It times the duration (among other things)，但是不做跟DNS 请求有关的任何事情。它只是透传和检查返回路径的源数据。

> *Hint* : 暗示，示意。我的理解，啥都不干，偷偷看着。

#### 未注册插件

还有一个特殊的插件不处理任何DNS的数据，但是以其他方式影响着 CoreDNS 的工作。比如， [bind]() 插件用于控制 CoreDNS  会绑定到哪个接口上。
 如下的插件都是这个类型的：

- [*bind*]() - 绑定接口。
- [*root*]() - set the root directory where CoreDNS plugins should look for files.
- [*health*]() - 开启 HTTP health 检查接口。
- [*ready*]() - support readiness reporting for a plugin.

### 常用插件

这里说明一些常用的插件，默认插件参见 [https://coredns.io/plugins](https://coredns.io/plugins)。

#### chaos

*chaos* 让 CoreDNS 以 CH class 应答查询 - 在确认服务器的时候很有用。

```corefile
. {
    chaos
}
```

通过如上配置，CoreDNS 会在收到请求后，应答它的版本：

```sh
$ dig @localhost -p 1053 CH version.bind TXT
...
;; ANSWER SECTION:
version.bind.       0   CH  TXT "CoreDNS-1.0.5"
...
```

可以在语法内定义 `VERSION` 和`AUTHORS`：

```css
chaos [VERSION] [AUTHORS...]
```

```corefile
. {
    chaos CoreDNS-001 info@coredns.io
}
```

> - **VERSION** 返回版本。 默认 `CoreDNS-<version>`，如果没设置的话。
> - **AUTHORS** 返回作者。默认定义在 OWNER 文件内。

#### dnsredir

[dnsredir](https://github.com/leiless/dnsredir)和官方[forward](https://coredns.io/plugins/forward/)插件功能差不多，是一个请求转发插件，但是功能更强大。常用于将一个或多个域名的解析转给指定其他上游服务器，比如用于将国外域名转发给国外DNS服务器，而`forward`目前只能将一个域（zone）转发给其他服务器。

由于该插件是第三方插件，所以需要注册并重新编译，注册时建议注册在`forward`插件的前面且与`forward`紧邻，紧邻的原因是因为他们的功能差不多，都是转发查询请求；而在前面是因为一般情况下`dnsredir`作用的域（zone）范围比`forward`小，比如将某部分域名用`dnsredir`转发，其他的用`forward`转发。

### 插件开发

#### 插件结构解析

插件由 Setup, Registration, and Handler 部分构成。

Setup 解析配置文件和插件的指令 (参见插件的 README 文档)。

Handler 是用于查询处理和逻辑实现的代码。

Registration 将插件注册到 CoreDNS - 当 CoreDNS 编译后。所有已注册的插件都可以被服务器使用。在运行的时候，会决定哪个服务器用哪个插件，由CoreDNS 的配置文件 Corefile 来完成。

## 可能的报错

插件 [*health*]() 的文档声明 "This plugin only needs to be enabled once"，这可能导致你认为如下是一个符合规定的Corefile：

```txt
health

. {
    whoami
}
```

但是，这不能工作，并且导致一些 简短的报错：

```bash
"Corefile:3 - Error during parsing: Unknown directive '.'".
```

为什么呢？ `health` 被看作一个 zone (and the start of a Server Block)。解析希望看到插件名字 (`cache`, `etcd`, etc.)，但是下一个标识是 `.`，这不是插件。
 正确的 Corefile 如下：

```corefile
. {
    whoami
    health
}
```

插件 *health* 文档里面的那段话，意思是一旦 *health* 被定义，它对整个CoreDNS 进程来说就是全局的，哪怕你是在一个server中定义它。

## 推荐的`Dockerfile`

```
FROM ubuntu:latest

ADD coredns /coredns

EXPOSE 53 53/udp
ENTRYPOINT ["/coredns"]
```

> 该配置是在官方的基础上进行“优化”，因为官方是基于`debian:stable-slim`镜像制作，并加上了证书（需要`apt-get update && apt-get -uy upgrade`），而`ubuntu`是过了不需要将证书复制进取，应该是默认内置的证书已经够用了，并且`ubuntu`默认带有`sh`等命令，方便进入容器赵问题。

## 参考资料

[CoreDNS配置说明 (aliyun.com)](https://help.aliyun.com/document_detail/380963.html)

[CoreDNS 手册__配置 - 简书 (jianshu.com)](https://www.jianshu.com/p/5af2c15f73c2)