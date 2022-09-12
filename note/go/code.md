---
title: 'Golang翻译文档（学习笔记）'
author: 'Laeni'
tags: 'go,golang'
date: '2022-08-20'
updated: '2022-09-04'
---

# go代码组织(结构)

## 包

包是同一目录中一起编译的源文件的集合。同一个包中定义的函数、类型、变量和常量对该包中的所有源文件都可见。

## 模块

一个存储库包含一个或多个模块。模块是一起发布的相关 Go 包的集合（一个或多个包）。虽然一个存储库可以包含多个模块，但是通常只直接包含一个模块，该模块位于存储库的根目录。以`go.mod`命名的文件声明模块路径：模块中所有包的导入路径前缀（除了作为导入路径前缀，还指示 go 命令应该从哪里下载该模块）。该模块包含包含其`go.mod`文件的目录中的包以及该目录的子目录，直到包含另一个`go.mod`文件（如果有）的下一个子目录。

请注意，在构建代码之前，您无需将代码发布到远程存储库。可以在本地定义模块，而不能属于存储库。但是，组织代码是一个好习惯，就好像有一天会发布它一样。

导入路径是用于导入包的字符串。 包的导入路径是其模块路径与其在模块中的子目录相连。 例如，模块 github.com/google/go-cmp 在`cmp/`目录中包含一个包。 该包的导入路径是`github.com/google/go-cmp/cmp`。 标准库中的包没有模块路径前缀。

初始化一个新模块：

```sh
$ go mod init example.com/user/hello
go: creating new go.mod: module example.com/user/hello
```

go自动根据模块地址下载模块原理：如果是常见的存储库（github或gitlab等）那就算为他们专门定制也很容易实现，其他的如果是以`.git`等明确带有源代码管理标识的则通过对应的协议进行拉取，否则先获取模块对于的`html`，然后解析其中的`meta`标签来决定下载方式，`meta`标签格式为`<meta name="go-import" content="github.com/xxx git https://github.com/xxx.git">`。

如果需要调用未发布的模块时，可以通过`replace`指令声明，参见[调用本地（未发布）的模块](#调用本地（未发布）的模块)。

## `.go`源文件

Go 源文件中的第一句有效语句（注释和空行不算）必须是`package <name>` 。可执行命令必须始终使用`package main` ，`main`包是程序的入口。

# [语言规范](https://go.dev/ref/spec)

// TODO

# [有效Go](https://go.dev/doc/effective_go)

## [格式化](https://go.dev/doc/effective_go#formatting)

Go提供了`gofmt`用于格式化源文件，Go库中所有代码都使用`gofmt`进行格式化。

## 注释

Go 提供 C 风格 `/* */`块注释和 C++ 风格的`//`行注释。但一般使用行注释，块注释主要为包注释，但在表达式中或禁用大量代码时很有用。 

直接出现在声明之前（中间没有换行符）的注释成为“[文档注释 ](https://go.dev/doc/comment)”。

## 命名约定

参见[官方文档](https://go.dev/doc/effective_go#names)。

## 变量

### 重新声明和重新分配

```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    return err
}
```

上述语句先声明了两个变量（ `f`和 `err`），之后调用 `f.Stat`， 看起来好像它声明 `d`和 `err`，但`err`出现在两个语句中，这种复制是合法的。`err`由第一条语句声明，而后面只是*重新分配*。这意味着调用`f.Stat`使用的是上面声明的`err`变量，只是给它一个新值。 

当使用`:=`申明多个变量时，至少要有一个变量是当前作用域中没有的，否则为无效语法。申明的多个变量中，如果遇到前面已经存在的变量，则如果和前面的属于不同作用域则重新创建，否则继续沿用已有变量。

## 控制结构 

Go 的控制结构与 C 的控制结构相关但有一些不同之处。有新的控制结构`select`，没有小括号但主体必须始终用大括号分隔。 

### if

```go
// 一般形式
if x > 0 {
    return y
}
// 有初始化语句
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

### for

Go的`for`循环与 C 类似，但不一样。它统一了`for`和`while`（ `while`可以使用`for`代替）而且没有 `do-while`，共有三种形式，其中只有一种有分号。 

```go
// 像 C 一样
for init; condition; post { }
// 像一个 C while
for condition { }
// 就像一个 C for(;;)
for { }
```

遍历数组、切片、字符串、映射或从通道读取时需要使用`range`管理循环。 

```go
// 如果只需要第一项，则第二项 value 可以省略；如果只需要第二项，则第一项需要使用空白标识符（_）忽略 key
for key, value := range oldMap {
    newMap[key] = value
}
```

如果使用`range`遍历字符串时将会做很多工作，因为每次遍历得到的是一个解析 UTF-8 后的 Unicode 码点。 错误的编码消耗一个字节并产生替换符文`U+FFFD`。（`rune`表示单个 Unicode 码点。）

```go
for pos, char := range "UTF8文\x80字" { // \x80 is an illegal UTF-8 encoding
	fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
//输出：
//character U+0055 'U' starts at byte position 0
//character U+0054 'T' starts at byte position 1
//character U+0046 'F' starts at byte position 2
//character U+0038 '8' starts at byte position 3
//character U+6587 '文' starts at byte position 4
//character U+FFFD '�' starts at byte position 7
//character U+5B57 '字' starts at byte position 8
```

### switch

Go的`switch`比C的更通用，表达式不必是常量甚至整数，`case`从上到下进行评估，直到找到匹配项，找到后不会再向 Java 的`switch`一样继续往下查找，因此可以将`if`- `else`- `if`- `else` 写为`switch`: 

```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

虽然不会自动往下查找所有匹配的的`case`，但可以使用逗号分隔多个条件（即条件分割的条件列表只要有一个满足即可）。 

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

由于 Go 匹配到满足条件的`case`后不会再继续往下匹配这一特性使得`break`在 Go 中并不像其他类似 C 中那样常见，但`break`可用于终止一个 `switch`中一个`case`块的剩余部分。此外还可以通过标签打破上层循环。 

```go
    src := "abcdefg"
Loop: // 定义了一个名为 Loop 的标签
	for i := 0; i < len(src); i++ {
		fmt.Println("i: ", i)
		switch {
		case src[i] < 'b':
			if i%2 == 0 {
				break // 这里只中断 switch 结构
			}
			fmt.Printf("%c\n", src[i])
		case src[i] < 'g':
			if i%2 == 0 {
				break Loop // 这里直接中断 for 循环
			}
			fmt.Printf("%c\n", src[i])
		}
	}
// 以上示例输出如下：
//i:  0
//i:  1
//b
//i:  2
```

> 当然， `continue`语句也可以接受可选标签，但它仅适用于循环。 

`switch`还可用于类型选择：

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default: // default 语句是可选的，并且放置顺序无所谓
	fmt.Printf("unexpected type %T\n", t) // %T prints whatever type t has
case bool:
	fmt.Printf("boolean %t\n", t) // t has type bool
case int:
	fmt.Printf("integer %d\n", t) // t has type int
case *bool:
	fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
	fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```

## 函数

### 多个返回值

Go 的一个不同寻常的特性是函数和方法 可以返回多个值。

### 命名结果参数

Go 函数的返回值可以像形参一样提前用变量定义好，这些命名结果参数在函数开始时被初始化为它们的类型的零值。

命名结果参数不是强制性的，但它们可以使代码更短、更清晰。

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### 延迟执行

在函数推出时执行的部分，一般用于关闭资源等。

延迟函数的参数（包括接收者 if  该函数是一种方法）在 *延迟*  执行，而不是在 *调用* 执行时。 除了避免烦恼  关于变量在函数执行时改变值，这意味着  单个延迟调用站点可以延迟多个功能  处决。 这是一个愚蠢的例子。  

延迟函数的参数在定义*延迟*执行时计算，且多个延迟函数按照后进先出顺序执行。

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}
func un(s string) {
    fmt.Println("leaving:", s)
}
func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}
func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}
func main() {
    b()
}
// 输出：
//entering: b
//in b
//entering: a
//in a
//leaving: a
//leaving: b
```

## 数据

### new

Go 有两个分配原语，内置函数`new`和`make`，它们做不同的事情并适用于不同的类型。

`new`是一个分配内存的内置函数，但不同于其他一些语言中的同名关键字，它不会*初始化*内存，只是将其*归零* 。也就是说，`new(T)`为类型为`T`的项目分配清零的内存，并返回其地址，即`T`类型的指针。

由于`new`返回的内存已归零，因此在设计数据结构时，安排每种类型的零值无需进一步初始化即可使用会很有帮助。即`new`创建的值可以直接使用。例如，`bytes.Buffer`的文档指出“`Buffer`的零值是可供使用的空缓冲区”。同样，`sync.Mutex`没有显式构造函数或`Init`方法。相反，`sync.Mutex`的零值被定义为未锁定的互斥体。

零值是有用的属性可传递地起作用，比如这种类型声明：

```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

`SyncedBuffer`类型的值也可以在分配或仅声明后立即使用。在下一个代码段中，`p`和`v`都将正常工作，而无需进一步操作。

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

### 构造函数和复合文本

有时零值不够好，需要初始化构造函数，比如`os`包的以下代码。

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```

以上代码有很多样板，我们可以使用*复合文本*对其进行简化，复合文本是一个表达式，每次计算时都会创建一个新实例。

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    // 采用 复合文本 创建新实例
    f := File{fd, name, nil, 0}
    return &f
}
```

请注意，与C不同，返回局部变量的地址是完全可以的；与变量关联的存储在函数返回后仍然存在。实际上，每次评估复合文本时都会分配一个新实例，因此我们可以组合最后两行。

```go
    return &File{fd, name, nil, 0}
```

复合文字的字段按顺序排列，并且必须全部存在。但是，可以通过将元素显式标记为`key:value`形式，这样就可以不按顺序传入，缺少的字段保留为它们各自的零值。 因此可以写为：  

```go
    return &File{fd: fd, name: name}
```

作为一种限制情况，如果复合文字根本不包含任何字段，它会创建类型的零值。表达式`new(File)`和`&File{}`是等价的。

也可以为数组、切片和Map创建复合文本，字段标签根据需要为索引或映射键。 在这些示例中，无论是`Enone` 、`Eio`还是`Einval`，只要它们是不同的就行。 

```go
a := [...]string   {1: "no error", 2: "Eio", 0: "invalid argument"}
s := []string      {1: "no error", 2: "Eio", 0: "invalid argument"}
m := map[int]string{1: "no error", 2: "Eio", 0: "invalid argument"}
```

### make

`make(T, args)`只创建切片、Map和通道，并返回`T`类型（非`*T` ）的*初始化*（非*清零*）值（不返回指针）。原因是这三种类型底层在使用前必须先初始化的数据结构的引用。例如，切片包含指向数据的指针（在数组内）、长度和容量，在初始化切片是 `nil`。对于切片、映射和通道，初始化内部数据结构并准备要使用的值。

```go
make([]int, 10, 100)
```

分配一个包含 100 个 int 的数组，然后创建一个长度为 10 且容量为 100 的切片结构，指向数组的前 10 个元素。（创建切片时，容量可省略） 相比之下， `new([]int)`返回指向新分配的归零切片的结构体的指针。

这些例子说明了两者之间的区别 `new`和 `make`. 

```go
var p *[]int = new([]int)       // 分配切片结构; *p == nil; 很少有用
var v  []int = make([]int, 100) // 切片 v 现在指向一个 100 个整数的新数组

// 不必要的复杂:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// 惯用语:
v := make([]int, 100)
```

### 数组

数组在对内存进行详细规划布局时很有用，有时可以帮助避免扩容，切片底层主要为数组。

数组在 Go 和 C 中的工作方式之间存在重大差异。 在 Go 中：

- 数组是值。将一个数组分配给另一个数组会复制所有元素。 
- 如果将数组传递给函数，它将收到数组的*副本*，而不是指向它的指针。
- 数组的大小是其类型的一部分。`[10]int`和`[20]int`是不同的类型。 

值传递可能很方便但代价也很大；如果想要类似 C 的行为和效率，可以传递数组的地址。

```go
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // 注意显式的地址操作符
```

虽然可以通过传递数组地址的方式，但不是 Go 的惯用方式，Go 一般用切片形式。 

### 切片

切片内部包装了数组，以提供更通用、更强大和更方便的数据序列接口。明确项目维数（例如变换矩阵）的除外，大多数数组编程在 Go 使用切片而不是简单的数组来完成。

切片保存对基础数组的引用，如果将一个切片分配给另一个切片，两者都引用同一个数组。如果函数采用切片参数，则调用方可以看到它对切片元素所做的更改，类似于将指针传递到基础数组。因此，`Read`函数可以接受切片参数，切片内的长度为要读取的数据量的上限。  

```go
func (f *File) Read(buf []byte) (n int, err error)
```

该方法返回读取的字节数和错误值（如果有）。要读入较大缓冲区的前 32 个字节`buf`，*请对缓冲区进行切片*（此处用作动词）。

```go
n, err := f.Read(buf[0:32])
```

这种切片是常见且有效的。实际上，暂时撇开效率不谈，以下代码段还将读取缓冲区的前 32 个字节。 

```go
var n int
var err error
for i := 0; i < 32; i++ {
    nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
    n += nbytes
    if nbytes == 0 || e != nil {
        err = e
        break
    }
}
```

切片的长度可以更改，只要它仍然适合基础数组的限制，只需将其分配给自身的一部分即可。可通过内置函数`cap`获取切片*容量*。下面是一个将数据追加到切片的函数。如果数据超出容量，则重新分配切片并返回生成的切片。

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}
```

之后我们必须返回切片，因为`Append`虽然可以修改`slice`的元素，但切片本身（保存指针、长度和容量的运行时数据结构）是按值传递的。以上函数实际上有内置函数`append`实现相同的功能。

### 二维切片 

Go 的数组和切片是一维的。要创建二维数组或切片，必须定义数组的数组或切片的切片，如下所示：

```go
type Transform [3][3]float64  // 一个 3x3 数组，实际上是一个数组的数组。
type LinesOfText [][]byte     // 一个字节切片的切片。
```

由于切片的长度可变，因此可以使每个内部切片具有不同的长度。这可能是一种常见的情况，如我们的示例所示：每行都有独立的长度。

```go
text := LinesOfText{
    []byte("Now is the time"),
    []byte("for all good gophers"),
    []byte("to bring some fun to the party."),
}
```

有时需要分配二维切片，例如，在处理像素扫描线时可能会出现这种情况。有两种方法可以实现此目的。一种是独立分配每个切片；另一种是分配单个数组并将各个切片指向其中。使用哪种方法取决于您的应用程序。如果切片可能增大或缩小，则应独立分配它们以避免覆盖下一行；如果不是，则使用单个分配构造对象可能会更有效。作为参考，以下是这两种方法的草图。

首先，一次一行：

```go
// 分配顶层切片.
picture := make([][]uint8, YSize) // One row per unit of y.
// 遍历行，为每一行分配切片.
for i := range picture {
    picture[i] = make([]uint8, XSize)
}
```

现在作为一个分配，分成几行： 

```go
// 分配顶层切片，和之前一样.
picture := make([][]uint8, YSize) // One row per unit of y.
// 分配一个大切片来保存所有像素.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// 循环遍历行，从剩余像素切片的前面对每一行进行切片。
for i := range picture {
    picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

### Map

Map 是一种方便且功能强大的内置数据结构，可将一种类型的值（*键*）与另一种类型的值（*元素*或*值*）相关联。键可以是定义相等运算符的任何类型的键，例如整数、浮点数和复数、字符串、指针、接口（只要动态类型支持相等）、结构和数组。切片不能用作映射键，因为切片上未定义相等性。与切片一样，Map 保存对基础数据结构的引用。如果将 Map 传递给更改映射内容的函数，则更改将在调用方中可见。

可以使用通常的复合文本语法和冒号分隔的键值对来构造映射，因此在初始化期间可以轻松构建它们。

```go
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

分配和获取 Map 值在语法上看起来就像对数组和切片执行相同的操作一样，只是索引不需要是整数。

```go
offset := timeZone["EST"]
```

尝试使用 Map 中不存在的键获取值将返回 Map 中条目类型的零值。例如，如果 Map 包含整数，则查找不存在的键将返回`0`。

```go
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // 如果 person 不在 Map 中，则为 false
    fmt.Println(person, "was at the meeting")
}
```

有时需要将缺失的条目与零值区分开来，是否有 0 的`"UTC"`条目，因为它可能根本不在 Map 中。这可以使用多重分配的形式进行区分。

```go
seconds, ok := timeZone[tz]
```

这被称为“comma ok”成语。在此示例中，如果`tz`存在，则`seconds`将正确设置并且`ok`为 true，如果不是，则`seconds`设置为零且`ok`为 false。

若要删除 Map 条目，请使用`delete`内置函数，其参数是映射和要删除的键。即使 Map 中已经不存在 Key，也可以安全地执行此操作。

```go
delete(timeZone, "PDT")
```

### 打印

Go 中的格式化打印使用类似于 C `printf`系列的样式，但更丰富、更通用。这些函数位于`fmt`包中，并具有大写的名称：`fmt.Printf`、`fmt.Fprintf`、`fmt.Sprintf`等。字符串函数 （ `Sprintf`等 ） 返回字符串，而不是填充提供的缓冲区。

您无需提供格式字符串。对于每个`Printf`、`Fprintf`和`Sprintf`并且还有另一对函数，例如`Print`和`Println`。这些函数不采用格式字符串，而是为每个参数生成默认格式。`Println`版本还在参数之间插入一个空格，并将换行符附加到输出中，而`Print`版本仅在两边的操作数都不是字符串时才添加空格。在此示例中，每行产生相同的输出。

```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

格式化打印函数 `fmt.Fprint`和与它同类型的函数将任何实现了 `io.Writer`接口的对象作为第一个参数，变量 `os.Stdout` 和 `os.Stderr`是熟悉的例子。 

在这里，事情开始与 C 不同。首先，数字格式，例如 `%d`不要采取标志作为符号或大小；  相反，打印例程使用参数的类型来确定这些属性。 

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
// 输出：
//18446744073709551615 ffffffffffffffff; -1 -1
```

如果您只想要默认转换，例如整数的十进制，您可以使用包罗万象的格式 `%v`（用于“值”）；结果正是 什么 `Print`和 `Println`会产生。  此外，该格式可以打印 *任何* 值，甚至是数组、切片、结构和Map。这是上一节中定义的时区映射的打印语句。  

```go
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
// 输出：
//map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]
```

对于 Map， `Printf`和它相关的函数按字典顺序对输出进行排序。 

打印结构时，格式`%+v`会使用结构的字段名称对字段进行批注，对于任何值，格式 `%#v`将以完整的 Go 语法打印值。

```go
type T struct {
    a int
    b float64
    c string
}
t := &T{ 7, -2.35, "abc\tdef" }
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
fmt.Printf("%#v\n", timeZone)
// 输出：
//&{7 -2.35 abc   def}
//&{a:7 b:-2.35 c:abc     def}
//&main.T{a:7, b:-2.35, c:"abc\tdef"}
//map[string]int{"CST":-21600, "EST":-18000, "MST":-25200, "PST":-28800, "UTC":0}
```

（注意与号。） 引用的字符串格式也可以通过 `%q`什么时候 应用于类型的值 `string`或者 `[]byte`. 替代格式 `%#q`如果可能，将使用反引号。 （这 `%q`格式也适用于整数和符文，产生 单引号符文常量。） 还， `%x`适用于字符串、字节数组和字节切片以及 在整数上，生成一个长的十六进制字符串，并使用 格式中的空格 ( `% x`) 它在字节之间放置空格。 

（请注意与号。）当应用于`string`类型或`[]byte`的值时，也可以通过`%q`格式获得。如果可能，备用格式`%#q`将使用反向引号。（`%q`格式也适用于整数和符文，产生单引号符文常量。）此外，`%x`适用于字符串、字节数组和字节切片以及整数，生成一个长十六进制字符串，并且格式为（`% x`）的空格，它将空格放在字节之间。

另一种方便的格式是 `%T`，它打印一个值的 *类型* 。 

```go
fmt.Printf("%T\n", timeZone)
// 输出：
//map[string]int
```

如果要控制自定义类型的默认格式，只需定义带有签名的方法`String() string`在类型上。 对于我们的简单类型 `T`，可能看起来像这样。 

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
// 输出：
//7/-2.35/"abc\tdef"
```

（如果需要打印*值*类型的`T`以及指向`T`的指针，则`String`的接收器必须是值类型；此示例使用指针，因为这对于结构类型更有效和惯用。）

我们的 `String`方法可以调用 `Sprintf`，因为打印例程是完全可重入的。 关于这种方法，有一个重要的细节需要了解：`String`方法中不要使用`Sprintf`来格式化它自己，否则将无限地重复调用到`String`方法，这是一个常见且容易犯的错误，如本例所示：

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

修复它也很容易：将参数转换为基本字符串类型，该类型没有该方法。

```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

在 [初始化部分 ](https://go.dev/doc/effective_go#initialization)，我们将看到另一种避免这种递归的技术。 

另一种打印技术是将打印例程的参数直接传递给另一个这样的例程。`Printf`的签名使用类型`...interface{}`为其最后一个参数指定任意数量的参数（任意类型） 可以出现在格式之后。 

```go
func Printf(format string, v ...interface{}) (n int, err error) {
```

在函数`Printf`中，`v`的作用类似于一个`[]interface{}`类型的变量，但是如果它被传递给另一个可变参数函数，它就像一个常规的参数列表。下面是我们上面使用的`log.Println`函数的实现。它将其参数直接传递给`fmt.Sprintln`。

```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

我们在调用`Sprintln`时在`v`后写`...`，告诉编译器将`v`视为参数列表；否则，它将`v`仅作为单个切片参数传递。

```go
func Min(a ...int) int {
    min := int(^uint(0) >> 1)  // largest int
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}
```

## 初始化

### 常量

Go 中的常量是在编译时创建的，即使在函数中定义为局部变量时也是如此，并且只能是数字、字符（符文）、字符串或布尔值。由于编译时限制，定义它们的表达式必须是常量表达式，可由编译器计算。例如，`1<<3`是一个常量表达式，而`math.Sin(math.Pi/4)`不是，因为函数调用`math.Sin`需要在运行时发生。

在 Go 中，枚举常量是使用`iota`枚举器创建的。由于`iota`可以成为表达式的一部分，并且表达式可以隐式重复，因此很容易构建复杂的值集。

```go
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

将方法（如`String`）附加到任何用户定义类型的功能使任意值可以自动设置自身格式以进行打印。尽管您会看到它最常应用于结构，但此技术对于标量类型（如 浮点类型`ByteSize`）也很有用。

```go
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```

表达式`YB`打印为`1.00YB` ，而`ByteSize(1e13)`打印为`9.09TB` 。

这里使用`Sprintf`实现`ByteSize`的`String`方法是安全的（避免无限期地重复），不是因为自动转换，而是因为它调用`Sprintf`和 `%f`（这不是字符串格式）：`Sprintf`只有在需要字符串或者`%f`需要浮点值时才会调用`String`方法。

### 变量

变量可以像常量一样初始化，但初始值设定项可以是运行时计算的通用表达式。

```go
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

### 初始化函数

每个源文件都可以定义`init`初始化函数。`init`所在包中的所有变量声明都计算了它们的初始值后调用，并且只有在初始化所有导入的包之后才会计算这些变量声明。

`init`函数的常见用途是在实际执行开始之前验证或修复程序状态的正确性。

```go
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

## 方法

### 指针与值

可以为任何命名类型（指针或接口除外）定义方法，而不要求一定是结构体才能定义方法。

在前面的切片讨论中，我们编写了一个`Append`函数，我们可以将它定义为切片上的方法。

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as the Append function defined above.
}
```

该方法仍然需要返回更新后的切片，我们可以*指针*来消除这种笨拙，因为该方法可以覆盖调用方的切片。

```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

事实上，我们可以做得更好。如果我们修改我们的函数，使它看起来像一个标准`Write`方法，就像这样，

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

然后类型`*ByteSlice`满足标准接口`io.Writer`，这很方便。例如，我们可以打印成一个。

```go
    var b ByteSlice
    fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

我们传递一个`ByteSlice`的地址，因为`*ByteSlice`满足`io.Writer`。关于接收器的指针与值的规则是，值方法可以在指针和值上调用，但指针方法只能在指针上调用。

出现此规则是因为指针方法可以修改接收器；对于值调用，它们将接收该值的副本，从而丢弃任何修改。但有一个例外：当该值可寻址时，语言会通过自动插入地址运算符来调用指针方法。在我们的示例中，变量`b`是可寻址的，因此我们可以仅使用`b.Write`，编译器将为我们重写为`(&b).Write`。

## 接口和其他类型

### 接口

Go 中的接口提供了一种指定对象行为的方法：如果某些东西可以*做到这一点*，那么它可以*在这里*使用。只有一个或两个方法的接口在Go代码中很常见，并且通常被赋予从该方法派生的名称，例如`io.Writer`。

### 转换

```go
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

此方法中从`Sequence`转换到`[]int`是安全的，因为如果我们忽略这两个类型（`Sequence`和`[]int`）的名称，则他们是相同的，所以在它们之间转换是合法的。该转换不会创建新值，它只是暂时充当现有值具有新类型的行为。（还有其他合法转换，例如从整数到浮点数，确实会创建新值。）

Go 程序的一个习惯用法是通过转换类型以访问一组不同的方法。例如，我们可以使用已有类型`sort.IntSlice`来简化代码：

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

### 接口转换和类型断言

[类型开关](https://go.dev/doc/effective_go#type_switch)是一种转换形式：它采用`interface{}`，并且对于 switch 中的每个 case，从某种意义上说，都是将其转换为该 case 的类型。下面的代码是`fmt.Printf`使用类型开关将值转换为字符串的简化版本。

```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

如果我们已经知道值就是某种类型，除了单例类型切换（转换），*类型断言*也可以做到。类型断言采用接口值并从中提取指定显式类型的值。该语法借用了类型开关的子句，但使用的是显式类型而不是`type`关键字：

```go
value.(typeName)
```

并且结果是静态类型`typeName`的新值。该类型必须是接口持有的具体类型，或者是值可以转换为的第二个接口类型。要提取我们知道值中的字符串，我们可以编写：

```go
str := value.(string)
```

但是，如果事实证明该值不包含字符串，则程序将崩溃并显示运行时错误。为了防止这种情况，请使用“逗号，ok”成语安全地测试该值是否为字符串：

```go
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```

### 共性

如果某个类型仅用于实现接口，并且永远不会有该接口之外的导出方法，则无需导出类型本身。仅导出接口可以清楚地看出，除了界面中描述的内容之外，该值没有有趣的行为。它还避免了在通用方法的每个实例上重复文档的需要。

在这种情况下，构造函数应返回接口值，而不是实现类型。例如，在哈希库中，`crc32.NewIEEE`和`adler32.New`都返回接口类型`hash.Hash32`。在 Go 程序中用CRC-32算法代替Adler-32只需要改变构造函数调用;代码的其余部分不受算法更改的影响。

类似的方法允许将`crypto`包中的各种流式密码算法与它们链接在一起的块密码分开。`crypto/cipher`包中的`Block`接口指定块密码的行为，该密码提供单个数据块的加密。然后，通过与`bufio`包的类比，实现此接口的密码包可用于构造由`Stream`接口表示的流式处理密码，而无需知道块加密的详细信息。

`crypto/cipher`接口如下所示：

```go
type Block interface {
    BlockSize() int
    Encrypt(dst, src []byte)
    Decrypt(dst, src []byte)
}

type Stream interface {
    XORKeyStream(dst, src []byte)
}
```

以下是计数器模式（CTR）流的定义，它将块密码转换为流式处理密码;请注意，块密码的详细信息被抽象出来了：

```go
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```

### 接口和方法

由于几乎任何东西都可以附加方法，因此几乎任何东西都可以满足接口。一个说明性示例是`http`包，它定义了`Handler`接口。实现`Handler`的任何对象都可以为 HTTP 请求提供服务。

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

`ResponseWriter`本身就是一个接口，它提供将响应返回给客户端所需的方法，这些方法包括标准`Write`方法。`Request`是一个结构，其中包含来自客户端的请求的解析表示形式。

下面是一个处理程序的简单实现，用于计算页面被访问的次数。

```go
// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    // 请注意 Fprintf 如何打印到 http.ResponseWriter
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

作为参考，下面介绍了如何将此类服务器附加到 URL 树上的节点。

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

但是为什么要做一个`Counter`结构呢？只需一个整数。（接收方必须是指针，以便调用方可以看到增量。

```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

如果您的程序具有一些内部状态，需要通知页面已被访问，该怎么办？将频道绑定到网页。

```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

最后，假设我们想要访问`/args`是获取启动服务器二进制文件时使用的参数。编写一个函数来打印参数很容易。

```go
func ArgServer() {
    fmt.Println(os.Args)
}
```

我们如何将其转换为HTTP服务器？有一种更干净的方法。由于我们可以为除指针和接口以外的任何类型定义方法，因此我们可以为函数编写方法。该包包含以下代码：

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

`HandlerFunc`是有方法的类型，因此该类型的值可以为 HTTP 请求提供服务。

要将`ArgServer`制作成HTTP服务器，我们首先对其进行修改以使其具有正确的签名。

```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

`ArgServer`现在具有与`HandlerFunc`相同的签名，因此可以将其转换为该类型以访问其方法，就像我们转换`Sequence`为`IntSlice`并访问`IntSlice.Sort`一样。设置它的代码很简洁：

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

## 空白标识符

可以使用任何类型的任何值分配或声明空白标识符，并且该值被无害地丢弃。这有点像写入Unix的`/dev/null`文件：它表示一个只写值，用作占位符，其中需要变量但实际值无关紧要。

### 多重分配中的空白标识符

在`for`-`range`循环中使用空白标识符是一般情况的特殊情况：多重赋值。

如果赋值左侧需要多个值，但程序不会使用其中一个值，则赋值左侧的空白标识符可避免创建虚拟变量的需要，并明确要丢弃该值。例如，当调用返回值和错误的函数时，只有错误是重要的，可以使用空白标识符丢弃不需要的值。

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
    fmt.Printf("%s does not exist\n", path)
}
```

有时，您会看到丢弃错误值以忽略错误的代码;这是可怕的做法。始终检查错误返回;提供它们是有原因的。

```go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
```

### 未使用的导入和变量

未使用导入包和未使用的声明变量是错误的。未使用的导入会使程序膨胀并编译缓慢，而已初始化但未使用的变量至少会浪费计算，其次表明可能存在更大的错误。然而，当一个程序处于活跃的开发阶段时，经常会出现未使用的导入和变量，并且只是为了让编译继续进行而删除它们可能会很烦人，因为在以后会使用它们，对此空白标识符提供了一种解决方法。

这个写了一半的程序有两个未使用的导入（`fmt`和`io`）和一个未使用的变量（`fd`），因此它不会编译，但到目前为止代码是正确的。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```

要消除对未使用导入的错误，使用空白标识符来引用导入包中的符号。同样，将未使用的变量`fd`分配给空白标识符将使未使用的变量错误静音。此版本的程序可以编译。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

按照惯例，消除导入错误的全局声明应紧跟在导入之后并加以注释，以便于查找，且提醒以后清理掉它们。

### 导入副作用

最终应使用或删除未使用的导入，但有时，导入包只是为了使用其副作用（例如`init函数`和`net/http/pprof`提供的 HTTP 处理程序调试信息）。这种情况下可以请包重命名为空白标识符：

```go
import _ "net/http/pprof"
```

这种导入形式清楚地表明，导入包是为了产生副作用，因为包没有其他可能的用途。

### 接口检查

类型仅通过实现接口的方法来实现接口，而不需要显式声明它实现了哪些接口。并且大多数接口转换都是静态的，因此在编译时进行检查。

但是，某些接口检查确实会在运行时进行。`encoding/json`包中有一个实例，它定义了一个`Marshaler`接口。当 JSON 编码器收到实现该接口的值时，编码器将调用值的编码方法将其转换为 JSON，而不是执行标准转换。编码器在运行时使用[类型断言](https://go.dev/doc/effective_go#interface_conversions)进行检查，如下所示：

```go
m, ok := val.(json.Marshaler)
```

如果只需要判断类型是否实现了接口，而没有实际使用值本身（可能作为错误检查的一部分），可使用空白标识符来忽略类型断言值：

```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

出现这种情况的一个地方是，当有必要在实现该类型的包中保证它必须满足某个接口时使用。如果某个类型（例如`json.RawMessage`）需要自定义 JSON 表示形式，则应实现`json.Marshaler`，但代码中没有静态转换让编译器自动验证这一点。如果类型无意中无法满足接口，JSON 编码器仍将工作，但不会使用自定义实现。为了保证实现是正确的，可以在包中使用使用空白标识符的全局声明：

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

在此声明中，涉及将`*RawMessage`转换为`Marshaler`的赋值需要`*RawMessage`实现`Marshaler`，并且该属性将在编译时进行检查。如果`json.Marshaler`接口发生变化，此包将不再编译，我们将注意到它需要更新。

此构造中空白标识符的外观指示声明仅用于类型检查，而不存在用于创建变量。但是，不要对满足接口的每种类型都执行此操作。按照惯例，仅当代码中不存在对应的静态转换时，才会使用这种方式，这种情况很少见。

## 嵌入

Go没有提供典型的类型驱动的子类概念（继承），但它确实能够通过在结构或接口中*嵌入*类型来“借用”实现的片段。只有接口可以嵌入到接口中。

接口嵌入非常简单。例如`io.ReadWriter`，一个接口同时包含`Read`和`Write`两个接口。我们可以通过显式列出这两种方法来指定，但是嵌入两个接口以形成新接口会更容易，更令人回味无穷，如下所示：

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

同样的基本思想也适用于结构，但具有更深远的影响。`bufio`包有两种结构类型（`bufio.Reader`和`bufio.Writer`） ，每种结构类型当然都实现了`io`包中的类似接口。并且`bufio`还实现了缓冲的读取器/写入器，它通过使用嵌入将读取器和写入器组合到一个结构中来实现：它列出了结构中的类型，但不为它们提供字段名称。

```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

嵌入的元素是指向结构的指针，当然必须初始化为指向有效的结构，然后才能使用它们。`ReadWriter`结构可以写成：

```go
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

但是，如果写成上面的形式，为了提升字段的方法并满足`io`接口，所以我们还需要提供转发方法：

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

通过直接嵌入结构，我们避免了这种冗余，因为嵌入类型的方法也是自动嵌入的。

嵌入与子类化有一个重要的区别。当我们嵌入一个类型时，该类型的方法成为外部类型的方法，但是当它们被调用时，该方法的接收器是内部类型，而不是外部类型。在我们的示例中，当调用`bufio.ReadWriter`的`Read`方法时，它与上面写的转发方法是完全相同的效果。接收器是`reader`的`ReadWriter`，而不是`ReadWriter`本身。

嵌入也可以是一个简单的便利。此示例显示一个嵌入的字段以及一个常规的命名字段。

```go
type Job struct {
    Command string
    *log.Logger
}
```

`Job`类型现在具有`Print`、`Printf`、`Println`和其他`*log.Logger`的方法。当然，我们可以给`Logger`一个字段名称，但没有必要这样做。现在，一旦初始化，我们就可以使用`Job`：

```go
job.Println("starting now...")
```

`Logger`是`Job`结构的常规字段，因此我们可以在`Job`构造函数中以通常的方式对其进行初始化，如下所示，

```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

或使用复合文本，

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

如果我们需要直接引用嵌入字段，则忽略包限定符的字段的类型名称将用作字段名称，就像我们在`ReadWriter`结构的`Read`方法中所做的那样。在这里，如果我们需要通过变量`job`访问`Job`中的`*log.Logger`，我们将编写`job.Logger`，如果我们想改进`Logger`的方法（重写），这将是有用的。

```go
func (job *Job) Printf(format string, args ...interface{}) {
    job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

嵌入类型引入了名称冲突的问题，但解决它们的规则很简单。首先，字段或方法`X`将任何其他项隐藏在`X`类型嵌套得更深的部分中。如果`log.Logger`包含一个名为`Command`的字段或方法，则`Command`的包装`Job`将支配它。

其次，如果相同的名称出现在相同的嵌套级别，则通常是错误的；如果`Job`结构包含另一个名为`Logger`的`log.Logger`字段或方法，则嵌入是错误的。但是，如果在类型定义之外的程序中从未提及重复的名称，则没有问题。此限定条件提供了一些保护，以防止对从外部嵌入的类型进行更改;如果添加的字段与另一个子类型中的另一个字段冲突（如果从未使用过这两个字段），则不会有问题。

## 并发

### 共享通信

并发编程是一个很大的话题，这里只有一些特定于Go的亮点。

许多环境中的并发编程由于实现对共享变量的正确访问所需的细微差别而变得困难。Go鼓励一种不同的方法，其中共享值在通道上传递，实际上，永远不会由单独的执行线程主动共享。在任何给定时间，只有一个`goroutine`可以访问该值。根据设计，数据竞争不会发生。为了鼓励这种思维方式，我们将其简化为一个口号：

> 不要通过共享内存进行通信；相反，通过通信来共享内存。

这种方法可能走得太远了。例如，给整数变量增加互斥锁来计算计数。但作为一种高级方法，使用通道来控制访问可以更轻松地编写清晰、正确的程序。

考虑此模型的一种方法是考虑在一个 CPU 上运行的典型单线程程序。它不需要同步原语。现在运行另一个这样的实例；它也不需要同步。现在让这两个人沟通，如果通信是同步器，则仍然不需要其他同步。例如，Unix管道非常适合这个模型。虽然 Go 的并发方法起源于 Hoare 的通信顺序进程（CSP），但它也可以被看作是 Unix 管道的类型安全泛化。

### goroutine

它们之所以被称为*goroutine*，是因为现有术语（线程、协程、进程等）传达了不准确的含义。*goroutine*有一个简单的模型：它是与同一地址空间中的其他*goroutine*同时执行的函数。它是轻量级的，成本略高于堆栈空间的分配。堆栈开始时很小，因此它们很轻量，并且根据需要分配（和释放）堆存储来。

Goroutines 被多路复用到多个操作系统线程上，因此，如果一个线程阻塞，例如在等待 I/O 时，其他线程将继续运行。他们的设计隐藏了许多线程创建和管理的复杂性。

在函数或方法调用前面加上关键字`go`，以便在新的 goroutine 中运行调用。当调用完成时，goroutine 静静地退出。（效果类似于 Unix shell 在后台运行命令的表示法`&`。

```go
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

函数文本在 goroutine 调用中可以很方便。

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

在 Go 中，函数字面量是闭包：实现确保只要引用函数的变量处于活动状态就会存活。

这些示例不太实用，因为这些函数无法发出完成信号。为此，我们需要通道。

### 通道

与 Map 一样，通道使用`make`分配，结果值会对基础数据结构的引用。如果提供了可选的整数参数，它将设置通道的缓冲区大小。对于无缓冲或同步通道，默认值为零。

```go
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

无缓冲信道将通信（值的交换）与同步相结合，保证两个计算（goroutine）处于已知状态。

有很多使用通道的好习惯。这里有一个让我们开始。在上一节中，我们在后台启动了排序。通道可以允许等待启动的 goroutine 排序完成。

```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

接收方总是阻塞，直到有数据要接收。如果通道未缓冲，则发送方将阻塞，直到接收方收到该值。如果通道有缓冲区，则发送方仅在将值复制到缓冲区之前才会阻塞；如果缓冲区已满，则意味着要等到某个接收方检索到值。

缓冲通道可以像信号量一样使用，例如限制吞吐量。在此示例中，传入的请求被传递到`handle`，后者将一个值发送到通道，处理请求，然后从通道接收一个值，以便为下一个使用者准备“信号量”。通道缓冲区的容量限制了同时调用`process`的次数。

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // 等待活动队列耗尽。
    process(r)  // 可能需要很长时间。
    <-sem       // 完毕; 处理下一个请求。
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // 不需要等待处理完成
    }
}
```

一次最多允许同时执行`MaxOutstanding`个`process`处理程序 ，任何更多的处理程序将阻止尝试发送到填充的通道缓冲区，直到其中一个现有处理程序完成并从缓冲区接收。

但是，这种设计有一个问题：`Serve`为每个传入的请求创建一个新的 goroutine，即使只有`MaxOutstanding`个请求可以同时运行。但如果请求来得太快，程序可能会消耗无限的资源。我们可以通过改变 goroutine 的创造来解决这一缺陷。这是一个明显的解决方案，但请注意，它有一个错误，我们随后会修复：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```

错误在于，在 Go `for`循环中，循环变量在每次迭代中都重用，因此`req`变量在所有 goroutine 之间共享。这不是我们想要的。我们需要确保`req`对每个 goroutine 都是唯一的。这里有一种方法可以做到这一点，将`req`的值作为参数传递给 goroutine 中的闭包：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req) // 这里的参数是立即计算的，和 defer 函数一样（如“defer a(b())”中，“b()”作为延迟函数的参数是提前计算好的；只不过这里的函数是匿名的）
    }
}
```

另一种解决方案是创建一个具有相同名称的新变量，如以下示例所示：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        req := req // 为 goroutine 创建新的 req 实例
        sem <- 1
        go func() {
            process(req)
            <-sem
        }()
    }
}
```

`req := req`看起来可能很奇怪，但在Go中这样做是合法的和惯用的。您将获得具有相同名称的变量的新版本，故意在局部隐藏循环变量，但对于每个 goroutine 是唯一的。

回到编写服务器的一般问题，另一种管理好资源的方法是启动固定数量的 goroutine，所有 goroutine 都从请求通道读取数据。goroutine 数限制了同时调用`process`的次数。

```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

### 通道的通道

Go最重要的属性之一是通道是值，可以像其他任何值一样进行分配和传递。此属性的常见用途是实现安全的并行解复用。

在上一节的示例中，`handle`是请求的理想化处理程序，但我们没有定义它正在处理的类型。如果该类型包含要回复的通道，则每个客户端都可以为回复提供自己的路径。下面是`Request`类型的示意图定义。

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

客户端提供一个函数及其参数，以及请求对象内部用于接收答案的通道。

```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

在服务器端，处理程序函数是唯一更改的内容。

```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

显然还有很多工作要做，以使其切合实际，但是此代码是限速、并行、非阻塞RPC系统的框架，并且看不到互斥锁。

### 并行

这些想法的另一个应用是跨多个CPU内核并行计算。如果计算可以分解成可以独立执行的单独部分，则可以并行化，并在每个部分完成时使用通道发出信号。

假设我们要对项的向量执行一个代价高昂的运算，并且每个项上的运算值是独立的，如这个理想化的例子所示。

```go
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}
```

我们在循环中独立启动这些部分，每个 CPU 一个。它们可以按任何顺序完成，但这并不重要;我们只是通过在发射所有 goroutine 后耗尽通道来计算完成信号。

```go
const numCPU = runtime.NumCPU() // CPU核心数，运行时动态地从运行时获取

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

还有一个函数`runtime.GOMAXPROCS`，它可以报告（或设置）Go 程序可以同时运行的核心数。它默认为`runtime.NumCPU()`得到的值，但可以通过设置`GOMAXPROCS`环境变量或用正数调用函数来覆盖。用零调用只是查询值。因此，如果我们想尊重用户的设置，我们应该写

```go
var numCPU = runtime.GOMAXPROCS(0)
```

确保不要混淆并发性（将程序构建为独立执行组件）和并行性（并行执行计算以提高多个 CPU 上的效率）的概念。虽然Go的并发功能可以使一些问题易于构建为并行计算，但Go是一种并发语言，而不是并行语言，并且并非所有并行化问题都适合Go的模型。有关区别的讨论，请参阅[此博客文章](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html)中引用的讨论。

### 泄漏的缓冲区

并发编程的工具甚至可以使非并发的想法更容易表达。下面是从 RPC 包中抽象出来的示例。客户端 goroutine 循环从某个源（可能是网络）接收数据。为了避免分配和释放缓冲区，它保留了一个可用列表，并使用缓冲通道来表示它。如果通道为空，则分配新的缓冲区。消息缓冲区准备就绪后，将发送到`serverChan`上的服务器。

```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}
```

服务器循环接收来自客户端的每条消息，对其进行处理，并将缓冲区返回到可用列表。

```go
func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

客户端尝试从`freeList`中检索缓冲区，如果没有可用的，它将分配一个新的。服务器会将`b`放回`freeList`可用列表中，除非该列表已满，在这种情况下，缓冲区将丢弃在地板上，由垃圾回收器回收。（`select`语句中的`default`子句在没有其他情况准备就绪时执行，这意味着`selects`永远不会阻塞。此实现只需几行即可构建一个泄漏的存储桶免费列表，依靠缓冲通道和垃圾收集器进行簿记。

## 错误

库例程通常向调用方返回某种错误指示。使用此功能提供详细的错误信息是很好的风格。例如，正如我们将看到的`os.Open`，它不仅返回失败的`nil`指针，还返回一个描述出错原因的错误值。

按照惯例，错误具有类型`error`，一个简单的内置接口。

```go
type error interface {
    Error() string
}
```

库编写者可以自由地使用更丰富的模型来实现此接口，这样不仅可以查看错误，还可以提供一些上下文。如前所述，`*os.File`除了通常的返回值外，`os.Open`还返回一个错误值。如果文件成功打开，则错误将是`nil`，但是当出现问题时，它将返回`os.PathError`：

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

`PathError`的`Error`生成如下字符串：

```
open /etc/passwx: no such file or directory
```

这样的错误，包括有问题的文件名，操作和它触发的操作系统错误，即使打印远离导致它的调用，也是有用的;它比普通的“没有这样的文件或目录”要丰富得多。

在可行的情况下，错误字符串应标识其来源，例如，使用前缀命名生成错误的操作或包。例如，在`image`包中，由于未知格式而导致的解码错误的字符串表示为“图像：未知格式”。

关心精确错误详细信息的调用方可以使用类型开关或类型断言来查找特定错误并提取详细信息。为此，`PathErrors`可能包括检查内部`Err`字段中的可恢复故障。

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

此处`if`的第二个语句是另一个[类型断言](https://go.dev/doc/effective_go#interface_conversions)。如果失败，`ok`则为假，并且`e`是`nil`。如果成功，`ok`则为 true，这意味着错误的类型为`*os.PathError`，然后`e`也是如此，我们可以检查有关错误的更多信息。

### 恐慌

向调用方报告错误的常用方法是将`error`作为额外的返回值。规范方法`Read`是一个众所周知的实例;它返回一个字节计数和一个`error`。但是，如果错误不可恢复怎么办？有时程序根本无法继续。

为此，有一个内置函数`panic`，该函数实际上会创建一个运行时错误，该错误将停止程序（但请参阅下一节）。该函数采用任意类型的单个参数（通常是字符串），以便在程序死机时打印出来。这也是一种表明发生了不可能的事情的方法，例如退出无限循环。

```go
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

这只是一个示例，但实际的库函数应避免`panic`。如果问题可以被掩盖或解决，那么最好让事情继续运行，而不是关闭整个程序。一个可能的反例是在初始化期间：如果库确实无法自行设置，那么可以这么说，恐慌可能是合理的。

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

### 恢复

当`panic`被调用时，包括隐式地针对运行时错误（例如将切片索引越界或类型断言失败）时，它会立即停止当前函数的执行，并开始展开 goroutine 的堆栈，并在此过程中运行任何延迟的函数。如果该展开达到 goroutine 堆栈的顶部，则程序将死亡。但是，可以使用内置函数`recover`重新获得对 goroutine 的控制并恢复正常执行。

调用`recover`停止展开并返回传递给`panic`的参数。因为展开时运行的唯一代码是在延迟函数内部，所以`recover`仅在延迟函数中有用。

`recover`的一个应用场景是关闭服务器内一个失败的 goroutine，而不会杀死其他正在执行的 goroutine。

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        log.Println("cell defer") // 即使发生了panic，该代码也会正常执行
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

在此示例中，如果`do(work)`出现恐慌，将记录结果，并且 goroutine 将干净利落地退出，而不会干扰其他人。在延迟关闭中无需执行任何其他操作；调用`recover`完全处理这种情况。

直接调用`recover`将总是返回`nil`，除非从延迟函数调用，所以延迟代码的`recover`前可以有其他代码，这些代码不会因为使用`panic`和`recover`而失败。例如，`safelyDo`中的延迟函数可能会在调用`recover`之前调用日志记录函数，并且该日志记录代码将不受 panic 状态的影响。

有了我们的恢复模式，`do`函数（以及它调用的任何内容）都可以通过调用`panic`来干净利落地摆脱任何不良情况。我们可以使用这个想法来简化复杂软件中的错误处理。让我们看一下`regexp`包的理想化版本，它通过使用本地错误类型调用`panic`来报告解析错误。

```go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

如果`doParse`发生 panic，恢复块会将返回值设置为`nil`（延迟函数可以修改命名的返回值）。然后，它将在分配`err`时检查错误是否是解析错误（断言它具有本地类型`Error`）。如果类型断言失败，将导致运行时错误，导致堆栈继续展开，就好像没有任何东西中断它一样。此检查意味着，如果发生意外情况（如索引超出边界），即使我们正在使用`panic`和`recover`处理解析错误，代码也会失败。

有了错误处理，`error`方法（因为它是一个绑定到类型的方法，所以它很好，甚至是自然的，因为它与内置类型`error`具有相同的名称）可以轻松报告解析错误，而不必担心手动展开解析堆栈：

```go
if pos == 0 {
    re.error("'*' illegal at start of expression")
}
```

尽管此模式很有用，但它应仅在包中使用。`Parse`将其内部`panic`调用转换为`error`值；它不会向其客户端公开`panics`。这是一条值得遵循的好规则。

顺便说一句，如果发生实际错误，此新`panic`会更改原始`panic`值，但原始错误和新错误都将显示在崩溃报告中，因此问题的根本原因仍然可见。因此，这种简单的重新恐慌方法通常就足够了 - 毕竟它是崩溃 - 但是如果您只想显示原始值，则可以编写更多的代码来过滤其他错误，并使用原始错误重新崩溃。

## 网络服务器

让我们用一个完整的Go程序，一个Web服务器来结束。Google在`chart.apis.google.com`提供了一项服务，可以自动将数据格式化为图表和图形。但是，很难以交互方式使用，因为需要将数据作为查询放入 URL 中。这里的程序为一种形式的数据提供了一个更好的接口：给定一小段文本，它调用图表服务器来生成QR码，一个编码文本的框矩阵。该图像可以用手机的相机抓取，并解释为URL，例如，将URL键入手机的小键盘。

```go
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET">
    <input maxLength=1024 size=70 name=s value="" title="Text to QR Encode">
    <input type=submit value="Show QR" name=qr>
</form>
</body>
</html>
`
```

# 特别注意

1. `make(T, args)`只创建切片、Map和通道，并返回`T`类型（非`*T` ）的*初始化*（非*清零*）值（不返回指针）。
2. 与切片一样，Map 保存对基础数据结构的引用。

# 其他常用操作

## 调用本地（未发布）的模块

一般模块的路径反映了其发布位置，但是如果一个模块或者某个版本尚未发布，而另一个模块需要调用它时，需要在`go.mod`中声明目标模块的具体位置。

使用`go mod edit`命令声明`example.com/greetings`模块为本地目录`../greetings`:

```sh
$ go mod edit -replace example.com/greetings=../greetings
```

上述命令会在`go.mod`文件中添加一行：

```sh
$ replace example.com/greetings => ../greetings
```

