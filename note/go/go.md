# 安装

虽然可以在[统一的网站](https://studygolang.com/dl)下载任意版本的go进行安装，但是推荐使用多版本管理工具[g](https://github.com/voidint/g/)进行安装，这里可以使go安装到约定的目录并且方便版本切换。

## 环境变量

go相关的环境变量有：`GOROOT`、`GOPATH`、`GOBIN`、`GO111MODULE`、`GOPROXY`、`GOSUMDB`、`GOPRIVATE`、`GONOPROXY`、`GONOSUMDB`、`GOMODCACHE`，但是大部分不需要关心。

设置环境变量：

```sh
$ export PATH=$PATH:/path/to/your/install/directory # linux and mac
$ set PATH=%PATH%;C:\path\to\your\install\directory # windo
$ go env -w GOBIN=C:\path\to\your\bin # 通用，但是只能设置 go 相关的环境变量
```

取消设置环境变量：

```sh
$ go env -u GOBIN
```

### GOROOT

go安装的位置，类似于Java中的`JAVA_HOME`，该变量有些工具需要使用，所以一般需要进行设置。如果手动安装时一般需要进行设置，而版本管理工具安装时会自动设置。

### GOPATH

工作空间。

GOPATH允许多个目录，当有多个GOPATH时默认将`go get`获取的包存放在第一个目录下。

如果没有启用模块(modules)，则`GOPATH`路径为`go run`等命令的工作路径(即在此路径下执行上述命令)；如果启用 modules 方式，GOPATH仅仅在 GOPATH/pkg/mod 存储下载的第三方依赖。

> GOPATH目录约定：
>
> bin - 编译后生成的可执行文件。为了方便，可以把此目录加入到`$PATH`变量中。
>
> pkg - 编译时生成的中间文件(比如：.a)
>
> src - 存放非go模块源代码(比如：.go .c .h .s等)

### GOBIN

`GOBIN`默认为`$GOPATH/bin`，但也可以明确指定。`go install`命令安装后的可执行文件就放在`GOBIN`下。

### GO111MODULE

是否启用go模块，取值为`on`、`off`和`auto`（默认）。一般不用设置，即使要设置也只是在项目范围内设置。

### GOPROXY

下载依赖时使用的代理地址，可以同时设置多个代理地址（用逗号分割）。

如：`https://goproxy.cn,https://mirrors.aliyun.com/goproxy,https://proxy.golang.com.cn,direct`

### GOMAXPROCS

设置 Go 携程中实际使用的线程数量，设置的值需要大于0，默认为CPU核心数（`runtime.NumCPU()`），设置后，程序可以通过`runtime.GOMAXPROCS(0)`获得。

# CTL

go命令行[官方文档](http://docscn.studygolang.com/cmd/go/)。

## go mod

Go mod 提供对模块操作的访问。

请注意，所有 go 命令都内置了对模块的支持，而不仅仅是“go mod”。 例如，日常添加、删除、升级和降级依赖项应该使用“go get”来完成。
有关模块功能的概述，请参阅`go help modules`。

用法:

        go mod <command> [arguments]

命令是:

        download    下载模块到本地缓存
        edit        从工具或脚本编辑 go.mod
        graph       打印模块依赖图
        init        在当前目录初始化新模块
        tidy        添加缺失和删除未使用的模块
        vendor      制作依赖项的 vendored 副本
        verify      验证依赖项是否具有预期内容
        why         解释为什么需要包或模块

使用`go help mod <command>`获取有关命令的更多信息。

### init

**用法**: go mod init [module-path]

Init 在当前目录中初始化并写入一个新的 go.mod 文件，实际上创建了一个以当前目录为根的新模块。 go.mod 文件必须不存在。

Init 接受一个可选参数，即新模块的模块路径。 如果省略模块路径参数，init 将尝试使用 .go 文件、vendoring工具配置文件（如 Gopkg.lock）和当前目录（如果在 GOPATH 中）中的导入注释来推断模块路径。

如果存在vendoring工具的配置文件，init 将尝试从中导入模块需求。

有关`go mod init`的更多信息，请参阅<https://golang.org/ref/mod#go-mod-init>。

```sh
$ go mod init example.com/user/hello
go: creating new go.mod: module example.com/user/hello
```

### tidy

下载依赖项并移除`go.mod`中不用的依赖项，相当于`go get`的增强。

## go run

**用法**: go run [build flags] [-exec xprog] package [arguments...]

Run 编译并运行`main`命名的 Go 包，可以看作是`go build -o xxx && ./xxx`的快捷方式。

如果包参数有版本后缀（如@latest 或@v1.0.0），"go run" 以模块感知模式构建程序，忽略当前目录或任何父目录中的 go.mod 文件，如果有的话。这对于运行程序而不影响主模块的依赖关系很有用；如果 package 参数没有版本后缀，“go run”可能会在模块感知模式或 GOPATH 模式下运行，具体取决于 GO111MODULE 环境变量和 go.mod 文件的存在。有关详细信息，请参阅“转到帮助模块”。如果启用了模块感知模式，则“run”在主模块的上下文中运行。

默认情况下，'go run' 直接运行编译后的二进制文件：'a.out arguments...'。
如果给出 -exec 标志，'go run' 使用 xprog 调用二进制文件： 'xprog a.out arguments...'。
如果未给出 -exec 标志，则 GOOS 或 GOARCH 不同于系统默认值，并且可以在当前搜索路径上找到名为`go_$GOOS_$GOARCH_exec`的程序，'go run' 使用该程序调用二进制文件，例如`go_js_wasm_exec a.out arguments...`。这允许在模拟器或其他执行方法可用时执行交叉编译的程序。

示例：

```shell
$ go run .
or
$ go run hello.go # hello.go 位于 main 方法
```

## go get

下载依赖项。常用示例如下：

```sh
# 下载本路径下的依赖，并且将其添加到 go.mod 中（如果不存在时）
$ go get .
```

## go install

将程序构建为可执行二进制并安装到`$GOIN`目录下，如果不存在`$GOIN`则安装到`$GOPATH`列表中第一个目录（允许有多个`$GOPATH`目录）的`bin`子目录中。更多参见[环境变量](#环境变量)。

```sh
$ go install example.com/user/hello
$ go install .
$ go install
```

> `go install`的工作目录应该在模块中，否则go将作为非模块对待，从而可能会失败。
>
> 命令接受相对于工作目录的路径，如果没有给出其他路径，则默认为当前工作目录中的包。所以上面三条命令都是等效的。

