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

配置一般指插件的配置，所以在此之前，最好先对插件有一定的了解。另外，配置还决定了插件是否启用，因为编译时可能会编译很多插件进去，但这些插件不一定都会启用，而是否启用则取决于配置中是否通过插件名称启用对应的插件。

一般会使用文件`Corefile`来配置 CoreDNS。当 CoreDNS 启动的时候，如果 `-conf` flag 没有被配置，就会在当前目录查找 `Corefile` 文件。

配置文件包含了一个或者多个服务块 (Server Blocks)。每个服务块列出了一个或多个插件。那些插件也可以在后面使用指令配置。CoreDNS的服务块配置格式为：

```
<zone> [<zone> ...] {
    <plugins>
}
```

最简单的配置：

```
. {}
```

上述配置等效于下面配置之一：

```
dns://.:53 {} # 完整配置
dns://. {}    # 省略端口时，端口默认为53
.:53 {}       # 省略协议时，协议默认为dns
```

在Corefile 文件中，插件的配置顺序不决定插件链的顺序。 插件执行的顺序，配置在[`plugin.cfg`](https://github.com/coredns/coredns/blob/master/plugin.cfg)文件中。

此外，在配置时，需要检查配置中用到的插件是否已经被编译进程序中，使用`coredns -plugins`查询插件列表。

### 环境变量

CoreDNS 在配置中支持环境变量。
 环境变量可以被使用在任何地方。语法是 `{$ENV_VAR}` ( Windows-类型的语法`{%ENV_VAR%}` 也是支持的)。CoreDNS 会在解析Corefile的时候替换这些变量内容。

### 导入其他文件

参考 [import](https://coredns.io/plugins/import/) 插件。这个插件有些特殊，可以被用在Corefile的任何地方。

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

### 服务块 Server Blocks

每个服务块（Server Block）以server应该服务的zones开头。在zone名字或者zone列表名（以空格分隔）之后，一个服务块以大括号作为开头和结束。

```
a.example.com b.example.com {
    # Plugins defined here.
}

.example.com {
	  # Plugins defined here.
}
```

上面的配置中定义了两个服务块，每个服务块对应一个完整且独立的插件链，其中第一个服务块只处理`a.example.com`和`b.example.com`两个**zone**的查询，第二个服务块只处理`c.example.com`的查询。如果想处理其他任意**zone**的查询，可以使用**root zone**(`.`):

```corefile
. {
    # Plugins defined here.
}
```

#### 端口

服务块（Server blocks）还可以指定监听端口。默认端口是 53 (DNS 服务标准端口)。指定端口，以冒号作为分隔符在zone后列出端口号。
 如下的 Corefile 指示 CoreDNS 创建一个 Server ， 监控端口 1053：

```corefile
.:1053 {
    # Plugins defined here.
}
```

> 注意： 如果你明确的定义了监听端口，你就不可以使用 `-dns.port`参数覆盖了。

如果有多个服务块有着相同的`zone:port`定义，则coredns会在启动的时候报错。

```txt
.:1054 {
}

.:1054 {
}
```

变更第二个端口为 1055 可以让这两个服务器块变成两个不同的服务器。

#### 协议

可以通过在服务器配置文件，在zone 前加个前缀来指定服务器接收哪种协议。

```
dns://.:1053 {
    # Plugins defined here.
}
```

当前 CoreDNS 接受4种协议: DNS, DNS over TLS (DoT), DNS over HTTP/2 (DoH)
 and DNS over gRPC。

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

#### errors

查询处理过程中遇到的任何错误都将打印到标准输出中。特定类型的错误可以每一段时间合并和打印一次。每个服务器块只能使用此插件一次。

> 注意和`debug`、`trace`插件的区别，因为这两个插件有特殊作用，并非表示输出的日志级别。

#### log

将所有查询（和回复部分）转储到标准输出上。请注意，对于繁忙的服务器，日志记录将影响性能，所以该插件并不是一定要开启。

> 注意和`debug`、`trace`插件的区别，因为这两个插件有特殊作用，并非表示输出的日志级别。

#### chaos

*chaos* 让 CoreDNS 以 CH class 方式（一般 class 为 `IN`，type 为 `A`，这在日志中有体现）响应CoreDNS服务器的相关信息，一般在确认服务器信息的时候使用。

```corefile
. {
    chaos
}
```

开启`chaos`插件后，可以通过`version.bind`、`version.server`、`authors.bind`、`hostname.bind`和`id.server`来获取服务器相关信息。注意，**只能使用上面的名称**，如果使用其他名称则会当做真实的查询来处理。

```sh
$ dig @localhost -p 53 CH TXT version.bind
...
;; ANSWER SECTION:
version.bind.		0	CH	TXT	"CoreDNS-1.10.0"
...

$ dig @localhost -p 53 CH TXT version.server
...
;; ANSWER SECTION:
version.server.		0	CH	TXT	"CoreDNS-1.10.0"
...

$ dig @localhost -p 53 CH TXT authors.bind
...
;; ANSWER SECTION:
authors.bind.		0	CH	TXT	"rajansandeep"
authors.bind.		0	CH	TXT	"ihac"
authors.bind.		0	CH	TXT	"bradbeam"
...

$ dig @localhost -p 53 CH TXT hostname.bind
...
;; ANSWER SECTION:
hostname.bind.		0	CH	TXT	"LaenideiMac.local"
...

$ dig @localhost -p 53 CH TXT id.server
...
;; ANSWER SECTION:
id.server.		0	CH	TXT	"LaenideiMac.local"
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
>
> 注意：虽然该插件还可以进一步配置，但是一般都不那样做，因为默认情况下返回服务器的真实版本等信息就很好。

#### whoami

该插件在生产中一般不会用到，但在测试DNS时可能会用到（当Corefile不存在时，它是默认加载的插件之一）。它的功能是无论查询什么域名，都永远返回解析器的本地 IP 地址、端口和传输端口。

DNS工具查询：

```sh
$ dig @127.0.0.1 example.com
...
;; ADDITIONAL SECTION:
example.com.		0	IN	A	127.0.0.1
_udp.example.com.	0	IN	SRV	0 0 63493 .
...
```

> 注意：不要用`nslookup`工具测试，因为该工具无法预期结果，但是可以用`dig`获取`ping`工具进行测试。

Ping测试：

```sh
# 由于 ping 默认使用的是系统默认 DNS 服务器，所以需要设置系统DNS服务器为测试的DNS服务器才能得到预期效果
$ ping example.com -c 1
PING example.com (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.027 ms

--- example.com ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.027/0.027/0.027/0.000 ms
```

另外，由于该插件的顺序往往会比较靠后，所以即使在正式环境中开启也影响不大，因为只要前面的插件有可用的结果即可。

#### dnsredir

[dnsredir](https://github.com/leiless/dnsredir)和官方[forward](https://coredns.io/plugins/forward/)插件功能差不多，是一个请求转发插件，但是功能更强大。常用于将一个或多个域名的解析转给指定其他上游服务器，比如用于将国外域名转发给国外DNS服务器，而`forward`目前只能将一个域（zone）转发给其他服务器。

由于该插件是第三方插件，所以需要注册并重新编译，注册时建议注册在`forward`插件的前面且与`forward`紧邻，紧邻的原因是因为他们的功能差不多，都是转发查询请求；而在前面是因为一般情况下`dnsredir`作用的域（zone）范围比`forward`小，比如将某部分域名用`dnsredir`转发，其他的用`forward`转发。

### 插件开发

一般情况下，插件开发应该遵循以下约定：

**包名**：整个插件都在同一个包中，如插件的名字是`example`，则包名为`example`。

**文件名**：至少存在`setup.go`和`<plugin_name>.go`两个文件，其中`setup.go`主要作用是通过`init`函数注册自己，而`<plugin_name>.go`则是核心逻辑的实现。

#### 插件结构解析

插件由 Setup, Registration, and Handler 部分构成。

Setup 解析配置文件和插件的指令 (参见插件的 README 文档)。

Handler 是用于查询处理和逻辑实现的代码。

Registration 将插件注册到 CoreDNS - 当 CoreDNS 编译后。所有已注册的插件都可以被服务器使用。在运行的时候，会决定哪个服务器用哪个插件，由CoreDNS 的配置文件 Corefile 来完成。

#### Setup

在`setup.go`的`init`函数中注册初始化函数：`func init() { plugin.Register("<plugin_name>", setup) }`。

其中，`setup`为初始化函数，该函数会在CoreDNS检测到配置文件中启用了该插件时才被调用，所以当该函数执行时是能够拿到对应的配置内容的。而该函数的作用是**解析配置**和**向插件列表中注册真正的插件实现**（`dnsserver.GetConfig(c).AddPlugin(<myPlugin>)`）。

```go
func setup(c *caddy.Controller) error {
	// TODO 这里可能需要解析插件的配置

	// 创建插件实例（可能需要用到前面解析的插件配置）.
	var pluginInstance plugin.Plugin = func(next plugin.Handler) plugin.Handler {
		return Whoami{}
	}
	// 将插件添加到 CoreDNS，以便服务器可以在其插件链中使用它。
	dnsserver.GetConfig(c).AddPlugin(pluginInstance)

	// 如果成功解析配置并且没有问题的话就返回 nil.
	return nil
}
```

#### 解析配置

解析配置主要是使用`caddyfile.Dispenser`结构体的相关方法，而`setup`函数的参数`caddy.Controller`本身是继承自`caddyfile.Dispenser`的，所以可以直接通过`caddy.Controller`使用这些方法。而`caddyfile.Dispenser`实际上是配置文件中该插件配置部分的结构化表示形式，而该插件支持怎样的配置完全由插件自己实现，所以`caddyfile.Dispenser`结构体仅仅提供几个很简单的且通用的便捷方法，更高层次的解析需要由插件自己实现。

`caddyfile.Dispenser`结构：

```go
type Dispenser struct {
	filename string  // 配置文件名，一般为 Corefile
  tokens   []Token // 已经解析后的配置标记，大致为 {"example", "arg1", "arg2", "{", "name1", "value1", "}"}
	cursor   int     // 游标，即 tokens 的下标
	nesting  int     // 嵌套层级
}
```

`caddyfile.Dispenser`常用方法：

假设有如下配置：

```
. {
    example o1 o2 o3 {
        n1 v1
        n2 v2
        n3 v3
        n3 v3
    }
    example {}
}
```

1. `Val()` - 获取当前游标所指向的token（可以理解为单词、标记）值。

   第一次（`cursor=-1`）调用时永远返回空字符串，但是如果调用了一次`Next()`则会返回插件的名字。再往后调用`Next()`则依次以toekn为单位返回配置的内容，如果有大括号，则大括号也会作为一个token看待（`cursor=4`时，`Val() = '{'`）。

2. Next - 往下移动游标

   1. `Next()` - 将游标往下移动一步(`cursor`值`+1`)，前提是还没到最后一步。

      假设当前`cursor=0`，调用`Next()`后`cursor=1`。

   2. `NextLine()` - 与`Next()`类似，也是将游标往下移动一步，但是不同之处在于`NextLine()`仅当游标处于一行的最后一个标记（token）时才移动，而`Next()`则是只要后面还有标记就一定会移动。

   3. `NextArg()` - 与`Next()`类似，同时与`NextLine()`相反。`NextArg()`仅当游标在不一行中最后一个标记时才往后移动一步。

   4. `NextBlock()` - 将游标置于大括号内的第一个标记处。第一次时，只有当下一个标记为左大括号（"{"）时才会执行这一操作，直到遇到右大括号为止（同时返回`false`）。

      本例中，只有当`cursor=3`时调用`NextBlock()`才会进入到块中，此时`cursor=5`，且`nesting=1`。

3. `RemainingArgs()` - 以切片形式返回当前行中当前游标后面的所有标记（不包含大括号），同时游标移动到当前行的最后一个标记上（也不包含大括号）。

   本例中，假如`cursor=0`，则调用`RemainingArgs()`会得到`['o1', 'o2', 'o3']`，同时游标变为`3`（`cursor=3`）。

配置中还可以重复对一个插件进行配置，但是是否支持重复配置也完全取决于插件自己。如果重复配置并且不在一起，最后CoreDNS解析器也会将他们放在一起并解析为`caddyfile.Dispenser`之后传递给插件。

```
. {
		example abc
		log
		example def
}
# 以上配置最终会被解析为类似如下格式后传递给 example 插件(忽略掉与 example 插件无关的配置)，这也表明配置顺序不决定插件顺序。
. {
		example abc
		example def
}
```

这里例举了一些常见的插件配置解析以供参考。

1. 插件配置不支持其他配置，如果出现其他配置则报错退出（这是最简单的配置），比如[whoami](https://github.com/coredns/coredns/tree/master/plugin/whoami)。

   正确配置：

   ```
   . {
   		whoami
   }
   ```

   错误配置：

   ```
   . {
   		# 该配置除了插件名称之外，还有额外的指令
   		whoami abcd
   }
   . {
   		# 该配置除了插件名称之外，还有额外的指令
   		whoami {
   				// ...
   		}
   }
   ```

   配置解析：

   ```go
   func setup(c *caddy.Controller) error {
   	c.Next() // 调用一次 Next() 之后将光标定位在'插件名'上，第一次调用永远返回 true
   
     // 如果'插件名'后还有其他参数则报错并退出
     if c.NextArg() {
   		return plugin.Error("whoami", c.ArgErr())
   	}
   
     // ...
   
   	return nil
   }
   ```

2. 插件仅支持可选指令，不支持块配置，比如[chaos](https://github.com/coredns/coredns/tree/master/plugin/chaos)。

   示例配置：

   ```
   . {
   		example o1 o2 o3
   }
   ```

   配置解析：

   ```
   func setup(c *caddy.Controller) error {
   	c.Next() // 调用一次 Next() 之后将光标定位在'插件名'上，第一次调用永远返回 true
   
     args := c.RemainingArgs() // 获取所有指令
     if len(args) == 0 {
     	// 指令为空
     } else {
     	// 指令不为空
     }
   
   	return nil
   }
   ```

3. 插件支持可选指令，同时也支持大括号括起来的可选配置，比如[hosts](https://github.com/coredns/coredns/tree/master/plugin/hosts)。

   示例配置：

   ```
   . {
   		hosts example.hosts example.org example.net {
           fallthrough
       }
   }
   ```

   配置解析：
   
   ```
   func setup(c *caddy.Controller) error {
   	c.Next() // 调用一次 Next() 之后将光标定位在'插件名'上，第一次调用永远返回 true
   
     args := c.RemainingArgs() // 获取所有指令
     if len(args) == 0 {
     	// 指令为空
     } else {
     	// 指令不为空
     }
   
   	for c.NextBlock() {
   		// 解析大括号内的配置
   	}
   
   	return nil
   }
   ```

## 推荐的`Dockerfile`

```
FROM ubuntu:latest

ADD coredns /coredns

EXPOSE 53 53/udp
ENTRYPOINT ["/coredns"]
```

> 该配置是在官方的基础上进行“优化”，因为官方是基于`debian:stable-slim`镜像制作，并加上了证书（需要`apt-get update && apt-get -uy upgrade`），经测试`ubuntu`不需要将证书复制进去，应该是默认内置的证书已经够用了，并且`ubuntu`默认带有`sh`等命令，方便进入容器等。

## 参考资料

[CoreDNS配置说明 (aliyun.com)](https://help.aliyun.com/document_detail/380963.html)

[CoreDNS 手册__配置 - 简书 (jianshu.com)](https://www.jianshu.com/p/5af2c15f73c2)