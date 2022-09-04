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

Go 的数组和切片是一维的。 要创建二维数组或切片的等价物，需要定义一个数组数组 或切片，如下所示：

Go 的数组和切片是一维的。要创建二维数组或切片，必须定义数组的数组或切片的数组，如下所示：

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

有时您需要将丢失的条目与 一个零值。  是否有条目 `"UTC"`或者是 0 因为它根本不在地图中？ 您可以通过多重分配的形式进行区分。

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

您不需要提供格式字符串。  对于每个 `Printf`, `Fprintf`和 `Sprintf`还有一对 功能，例如 `Print`和 `Println`. 这些函数不采用格式字符串，而是生成默认值 每个参数的格式。  这 `Println`版本也插入一个空白 在参数之间并在输出中附加一个换行符，而 这 `Print`只有当两边的操作数都不是字符串时，版本才会添加空格。 在此示例中，每一行都产生相同的输出。 

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

函数内 `Printf`,  `v`就像一个类型的变量 `[]interface{}`但如果它被传递给另一个可变参数函数，它的行为就像 一个常规的参数列表。 这里是执行 功能 `log.Println`我们在上面使用过。  它将其参数直接传递给 `fmt.Sprintln`对于实际的格式。 

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

每个源文件都可以定义自己的 niladic `init`函数来设置所需的任何状态（实际上每个文件可以有多个`init`函数）。意味着：`init`所在包中的所有变量声明都计算了它们的初始值后调用，并且只有在初始化所有导入的包之后才会计算这些变量声明。

除了不能表示为声明的初始化之外，`init`函数的常见用途是在实际执行开始之前验证或修复程序状态的正确性。

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

