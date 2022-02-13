---
title: 'Iris 框架使用示例与笔记'
author: 'Laeni'
tags: 'WEB框架,Iris,golang'
date: '2022-01-12'
updated: '2022-02-13'
---

## [安装](https://www.iris-go.com/docs/#/?id=installation)

Iris是一款跨平台软件。

唯一的要求是[Go编程语言](https://golang.org/dl/)，版本1.14及更高版本。

```bash
$ mkdir myapp
$ cd myapp
$ go mod init myapp
$ go get github.com/kataras/iris/v12@master
```

将其导入到代码中：

```go
import "github.com/kataras/iris/v12"
```

## [快速入门](https://www.iris-go.com/docs/#/?id=quick-start)

```go
func main() {
	app := iris.New()
	booksAPI := app.Party("/books")

	// 普通用法
	//commonTest(booksAPI)
	// 普通用法
	mvcTest(booksAPI)

	_ = app.Listen(":8080")
}

// Book example
type Book struct {
	Title string `json:"title"`
}

// 普通用法
func commonTest(booksAPI router.Party) {
	// [可选] Compression 是一种中间件，可以使用提供的最佳压缩方式进行写入和读取。用法：app.Use（用于匹配的路由） app.UseRouter（用于匹配和 404 或其他 HTTP 错误）。
	booksAPI.Use(iris.Compression)

	// GET: http://localhost:8080/books
	booksAPI.Get("/", func(ctx iris.Context) {
		books := []Book{
			{"三国演绎"},
			{"红楼梦"},
			{"水浒传"},
			{"西游记"},
		}

		_, _ = ctx.JSON(books)
		// 提示：在服务器的优先级和客户端的要求之间协商响应，而不是 ctx.JSON：
		// TIP: negotiate the response between server's prioritizes and client's requirements, instead of ctx.JSON:
		// ctx.Negotiation().JSON().MsgPack().Protobuf()
		// ctx.Negotiate(books)
	})

	// POST: http://localhost:8080/books
	booksAPI.Post("/", func(ctx iris.Context) {
		var b Book
		err := ctx.ReadJSON(&b)
		// 提示：使用 ctx.ReadBody(&b) 来绑定任何类型的传入数据。
		// TIP: use ctx.ReadBody(&b) to bind any type of incoming data instead.
		if err != nil {
			// e.g: {"status": 400, "title": "创建书失败", "detail": "invalid character '/' looking for beginning of object key string"}
			ctx.StopWithProblem(iris.StatusBadRequest, iris.NewProblem().Title("创建书失败").DetailErr(err))
			// 提示：当错误只需要纯文本响应时，请使用 ctx.StopWithError(code, err)。
			// TIP: use ctx.StopWithError(code, err) when only plain text responses are expected on errors.
			// e.g: invalid character '/' looking for beginning of object key string
			//ctx.StopWithError(iris.StatusBadRequest, err)
			// StopWithPlainError 类似于 `StopWithError` 但它不会向响应编写器写入任何内容，它会存储错误，因此任何匹配给定“statusCode”的错误处理程序都可以自己处理它。
			//ctx.StopWithPlainError(iris.StatusBadRequest, err)
			// e.g: Bad Request
			//ctx.StopWithStatus(iris.StatusBadRequest)
			return
		}

		println("收到书: " + b.Title)
		ctx.StatusCode(iris.StatusCreated)
	})
}

//#region MVC用法
func mvcTest(booksAPI router.Party) {
	m := mvc.New(booksAPI)
	m.Handle(new(BookController))
}
type BookController struct {
}
func (c BookController) Get() []Book {
	return []Book{
		{"三国演绎"},
		{"红楼梦"},
		{"水浒传"},
		{"西游记"},
	}
}
func (c BookController) Post(b Book) int {
	println("收到书: " + b.Title)
	return iris.StatusCreated
}
//#endregion
```

### [使用获取、发布、放置、修补、删除和选项](https://www.iris-go.com/docs/#/?id=using-get-post-put-patch-delete-and-options)

```go
func main() {
    // 使用默认中间件创建一个 iris 应用程序:
    // 日志级别默认为"debug".
    // 在"./locales"目录和 HTML模板目录("./views"或"./templates")上启用了本地化。
    // It runs with the AccessLog on "./access.log",
    // Recovery (crash-free) and Request ID middleware already attached.
    app := iris.Default()

    app.Get("/someGet", getting)
    app.Post("/somePost", posting)
    app.Put("/somePut", putting)
    app.Delete("/someDelete", deleting)
    app.Patch("/somePatch", patching)
    app.Header("/someHead", head)
    app.Options("/someOptions", options)

    app.Listen(":8080")
}
```

### [路径参数](https://www.iris-go.com/docs/#/?id=parameters-in-path)

```go
func main() {
    app := iris.Default()

    // 匹配 /user/xxx 但不会匹配 /user/ 或 /user
    app.Get("/user/{name}", func(ctx iris.Context) {
        name := ctx.Params().Get("name")
        ctx.Writef("Hello %s", name)
    })

    // 但是，这个将匹配 /user/john/ 和 /user/john/send
    // 如果没有其他路由器匹配 /user/john，它将 /user/john/ 重定向到 /user/john
    app.Get("/user/{name}/{action:path}", func(ctx iris.Context) {
        name := ctx.Params().Get("name")
        action := ctx.Params().Get("action")
        message := name + " is " + action
        ctx.WriteString(message)
    })

    // 对于每个匹配的请求上下文将保存路由定义 | For each matched request Context will hold the route definition
    app.Post("/user/{name:string}/{action:path}", func(ctx iris.Context) {
        ctx.GetCurrentRoute().Tmpl().Src == "/user/{name:string}/{action:path}" // true
    })

    app.Listen(":8080")
}
```

内置可用参数类型：

| 参数类型        | 转到类型 | 检索帮助程序         | 验证                                                         |
| --------------- | -------- | -------------------- | ------------------------------------------------------------ |
| `:string`       | string   | `Params().Get`       | 任何内容（单路径段）                                         |
| `:uuid`         | string   | `Params().Get`       | uuidv4 或 v1（单路径段）                                     |
| `:int`          | int      | `Params().GetInt`    | -9223372036854775808到9223372036854775807 （x64） 或 -2147483648 到 2147483647 （x32），取决于主机拱门 |
| `:int8`         | int8     | `Params().GetInt8`   | -128 到 127                                                  |
| `:int16`        | int16    | `Params().GetInt16`  | -32768 到 32767                                              |
| `:int32`        | int32    | `Params().GetInt32`  | -2147483647 2147483648                                       |
| `:int64`        | int64    | `Params().GetInt64`  | -9223372036854775808 到 9223372036854775807                  |
| `:uint`         | uint     | `Params().GetUint`   | 0 到 18446744073709551615 （x64） 或 0 到 4294967295 （x32），取决于主机拱门 |
| `:uint8`        | uint8    | `Params().GetUint8`  | 0 到 255                                                     |
| `:uint16`       | uint16   | `Params().GetUint16` | 0 到 65535                                                   |
| `:uint32`       | uint32   | `Params().GetUint32` | 0 到 4294967295                                              |
| `:uint64`       | uint64   | `Params().GetUint64` | 0 到 18446744073709551615                                    |
| `:bool`         | 布尔     | `Params().GetBool`   | "1"或"t"或"T"或"TRUE"或"true"或"True"或"True"或"0"或"f"或"F"或"FALSE"或"false"或"False" |
| `:alphabetical` | 字符串   | `Params().Get`       | 小写或大写字母                                               |
| `:file`         | 字符串   | `Params().Get`       | 小写或大写字母、数字、下划线 （_）、短划线 （-）、点 （.），且无空格或其他对文件名无效的特殊字符 |
| `:path`         | 字符串   | `Params().Get`       | 任何东西，可以用斜杠（路径段）分隔，但应该是路由路径的最后一部分 |

更多示例可在以下位置找到：[_examples/路由](https://github.com/kataras/iris/tree/master/_examples/routing)。

### [查询字符串参数](https://www.iris-go.com/docs/#/?id=querystring-parameters)

```go
func main() {
    app := iris.Default()

    // 使用现有的底层请求对象解析查询字符串参数。
    // 请求响应 url 匹配:  /welcome?firstname=Jane&lastname=Doe
    app.Get("/welcome", func(ctx iris.Context) {
        firstname := ctx.URLParamDefault("firstname", "Guest")
        lastname := ctx.URLParam("lastname") // shortcut for ctx.Request().URL.Query().Get("lastname")
        // 同时获取相同名字的多个参数 - ?id=a&id=b&id=c
        ids := ctx.URLParamSlice("id")

        ctx.Writef("Hello %s %s", firstname, lastname)
    })
    app.Listen(":8080")
}
```

### [Multipart/Urlencoded Form](https://www.iris-go.com/docs/#/?id=multiparturlencoded-form)

```go
func main() {
    app := iris.Default()

    app.Post("/form_post", func(ctx iris.Context) {
        message := ctx.PostValue("message")
        nick := ctx.PostValueDefault("nick", "anonymous")

        ctx.JSON(iris.Map{
            "status":  "posted",
            "message": message,
            "nick":    nick,
        })
    })
    app.Listen(":8080")
}

// form-data
curl --location --request POST 'http://127.0.0.1:8080/form_post' \
--header 'Content-Type: multipart/form-data' \
--form 'message="hhhdsgsd"' \
--form 'nick="Nick"'
// x-www-form-urlencoded
curl --location --request POST 'http://127.0.0.1:8080/form_post' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'message=hhhdsgsd' \
--data-urlencode 'nick=Nick'
```

### [上传文件](https://www.iris-go.com/docs/#/?id=upload-files)

#### [单个文件](https://www.iris-go.com/docs/#/?id=single-file)

```go
const maxSize = 8 * iris.MB

func main() {
    app := iris.Default()

    app.Post("/upload", func(ctx iris.Context) {
        // 设置 multipart form 最大限制大小 (default is 32 MiB)
        ctx.SetMaxRequestBodySize(maxSize)
        // OR
        // app.Use(iris.LimitRequestBodySize(maxSize))
        // OR
        // iris.WithPostMaxMemory(maxSize)

        // single file
        file, fileHeader, err:= ctx.FormFile("file")
        if err != nil {
            ctx.StopWithError(iris.StatusBadRequest, err)
            return
        }

        // Upload the file to specific destination.
        dest := filepath.Join("./uploads", fileHeader.Filename)
        ctx.SaveFormFile(file, dest)

        ctx.Writef("File: %s uploaded!", fileHeader.Filename)
    })

    app.Listen(":8080")
}
```

如何：`curl`

```bash
curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/kataras/test.zip" \
  -H "Content-Type: multipart/form-data"
```

#### [多个文件](https://www.iris-go.com/docs/#/?id=multiple-files)

请参阅详细的[示例代码](https://github.com/kataras/iris/tree/master/_examples/file-server/upload-files)。

```go
func main() {
    app := iris.Default()
    app.Post("/upload", func(ctx iris.Context) {
        files, n, err := ctx.FormFiles("file")
        if err != nil {
            ctx.StopWithStatus(iris.StatusInternalServerError)
            return
        }

        ctx.Writef("%d files of %d total size uploaded!", len(files), n)
    })

    app.Listen(":8080", iris.WithPostMaxMemory(8 * iris.MB))
}
```

如何：`curl`

```bash
curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/kataras/test1.zip" \
  -F "file=@/Users/kataras/test2.zip" \
  -H "Content-Type: multipart/form-data"
```

### [分组路由](https://www.iris-go.com/docs/#/?id=grouping-routes)

```go
func main() {
    app := iris.Default()

    // Simple group: v1
    v1 := app.Party("/v1")
    {
        v1.Post("/login", loginEndpoint)
        v1.Post("/submit", submitEndpoint)
        v1.Post("/read", readEndpoint)
    }

    // Simple group: v2
    v2 := app.Party("/v2")
    {
        v2.Post("/login", loginEndpoint)
        v2.Post("/submit", submitEndpoint)
        v2.Post("/read", readEndpoint)
    }

    app.Listen(":8080")
}
```

### [无任何中间件](https://www.iris-go.com/docs/#/?id=blank-iris-without-middleware-by-default)

用

```go
app := iris.New()
```

而不是

```go
// Default with "debug" Logger Level.
// Localization enabled on "./locales" directory
// and HTML templates on "./views" or "./templates" directory.
// It runs with the AccessLog on "./access.log",
// Recovery and Request ID middleware already attached.
app := iris.Default()
```

### [使用中间件](https://www.iris-go.com/docs/#/?id=using-middleware)

```go
package main

import (
    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/middleware/recover"
)

func main() {
    // 默认情况下创建一个没有任何中间件的 iris 应用程序
    app := iris.New()

    // 使用 `UseRouter` 的全局中间件。| Global middleware using `UseRouter`.
    //
    // Recovery middleware recovers from any panics and writes a 500 if there was one.
    // 恢复中间件从任何 panic 中恢复并写入 500（如果有）
    app.UseRouter(recover.New())

    // 每个路由中间件，可以根据需要添加任意数量的中间件。| Per route middleware, you can add as many as you desire.
    app.Get("/benchmark", MyBenchLogger(), benchEndpoint)

    // Authorization group
    // authorized := app.Party("/", AuthRequired())
    // exactly the same as:
    authorized := app.Party("/")
    // 每组中间件！ 在这种情况下，我们在“授权”组中使用自定义创建的 AuthRequired() 中间件。
    // per group middleware! in this case we use the custom created AuthRequired() middleware just in the "authorized" group.
    authorized.Use(AuthRequired())
    {
        authorized.Post("/login", loginEndpoint)
        authorized.Post("/submit", submitEndpoint)
        authorized.Post("/read", readEndpoint)

        // nested group
        testing := authorized.Party("testing")
        testing.Get("/analytics", analyticsEndpoint)
    }

    // Listen and serve on 0.0.0.0:8080
    app.Listen(":8080")
}
```

### [应用程序文件记录器](https://www.iris-go.com/docs/#/?id=application-file-logger)

```go
func main() {
    app := iris.Default()

    f, _ := os.Create("iris.log")
    app.Logger().
        // [可选]记录到文件,写入文件时会自动禁用颜色.
    	SetOutput(f).
    	// [可选]同时将日志写入文件和控制台
		AddOutput(os.Stdout).
		// [可选]更改输出格式 - 未测试成功
		SetFormat("json", "    ").
    	// [可选]注册自定义格式化程序
    	RegisterFormatter(new(myFormatter))

    app.Get("/ping", func(ctx iris.Context) {
        ctx.WriteString("pong")
    })

   app.Listen(":8080")
}
```

### [控制日志输出着色](https://www.iris-go.com/docs/#/?id=controlling-log-output-coloring)

默认情况下，控制台上的日志输出应根据检测到的 TTY 进行着色。

自定义关卡标题，文本，颜色和样式。

导入和 ：`golog``pio`

```go
import (
    "github.com/kataras/golog"
    "github.com/kataras/pio"
    // [...]
)
```

获取要自定义的级别，例如：`DebugLevel`

```go
level := golog.Levels[golog.DebugLevel]
```

您可以完全控制他的文本，标题和样式：

```go
// 命名（小写）的级别名称将用于将 `SetLevel` 上的字符串级别转换为正确的级别类型。
// The Name of the Level that named (lowercased) will be used to convert a string level on `SetLevel` to the correct Level type.
Name string
// 备用名称是可以引用此特定日志级别的名称。| AlternativeNames are the names that can be referred to this specific log level.
// i.e Name = "warn"
// AlternativeNames = []string{"warning"}, 这是一个可选字段，因此我们将 Name 保留为一个简单的字符串并创建了这个新字段。 | it's an optional field, therefore we keep Name as a simple string and created this new field.
AlternativeNames []string
// Tha Title 是日志级别的前缀。| Tha Title is the prefix of the log level.
// 请参阅“颜色代码”和“样式”。| See `ColorCode` and `Style` too.
// `ColorCode` 和 `Style` 都应该在作家之间得到尊重。| Both `ColorCode` and `Style` should be respected across writers.
Title string
// ColorCode 为 `Title` 指定颜色。| ColorCode a color for the `Title`.
ColorCode int
// 为`Title`设置一个或多个丰富选项的样式。| Style one or more rich options for the `Title`.
Style []pio.RichOption
```

示例代码：

```go
level := golog.Levels[golog.DebugLevel]
level.Name = "debug" // default
level.Title = "[DBUG]" // default
level.ColorCode = pio.Yellow // default
```

[golog.Formatter](https://github.com/kataras/golog/blob/master/formatter.go)接口如下所示：

```go
// Formatter 负责将日志打印到 logger 的 writer。
type Formatter interface {
    // 格式化程序的名称。
    String() string
    // 设置任何选项并返回一个克隆，通用。 请参阅`Logger.SetFormat`。
    Options(opts ...interface{}) Formatter
    // 将"log"写入"dest"记录器。
    Format(dest io.Writer, log *Log) bool
}
```

**要更改每个级别的输出和格式， 请执行以下操作：**

```go
app.Logger().SetLevelOutput("error", os.Stderr)
app.Logger().SetLevelFormat("json")
```

### [请求记录](https://www.iris-go.com/docs/#/?id=request-logging)

我们在上面看到的应用程序记录器用于记录应用程序相关的信息和错误。另一方面，访问记录器（如下所示）用于记录传入的 HTTP 请求和响应。

```go
package main

import (
    "os"

    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/middleware/accesslog"
)

// 仔细阅读示例及其注释.
func makeAccessLog() *accesslog.AccessLog {
	// 初始化一个新的 access 日志中间件.
	ac := accesslog.File("./access-0.log")
	// 删除此行以禁用日志记录到控制台:
	ac.AddOutput(os.Stdout)

	// 默认配置:
	ac.Delim = '|'
	ac.TimeFormat = "2006-01-02 15:04:05"
	ac.Async = false
	ac.IP = true
	ac.BytesReceivedBody = true
	ac.BytesSentBody = true
	ac.BytesReceived = false
	ac.BytesSent = false
	ac.BodyMinify = true
	ac.RequestBody = true
	ac.ResponseBody = false
	ac.KeepMultiLineError = true
	ac.PanicLog = accesslog.LogHandler

	// 缺少格式化程序时的默认行格式 | Default line format if formatter is missing:
	// Time|Latency|Code|Method|Path|IP|Path Params Query Fields|Bytes Received|Bytes Sent|Request|Response|
	//
	// 设置自定义格式化程序:
	ac.SetFormatter(&accesslog.JSON{
		Indent:    "",   // 格式化缩进，为空时不格式化
		HumanTime: true, // 是否将时间格式化为人类可读
	})
	// ac.SetFormatter(&accesslog.CSV{})
	// ac.SetFormatter(&accesslog.Template{Text: "{{.Code}}"})

	return ac
}

func main() {
    ac := makeAccessLog()
    defer ac.Close() // Close the underline file.

    app := iris.New()
    // Register the middleware (UseRouter to catch http errors too).
    app.UseRouter(ac.Handler)

    app.Get("/", indexHandler)

    app.Listen(":8080")
}

func indexHandler(ctx iris.Context) {
    ctx.WriteString("OK")
}
```

阅读更多示例：[_examples/日志记录/请求记录器](https://github.com/kataras/iris/tree/master/_examples/logging/request-logger)。

### [模型绑定和验证](https://www.iris-go.com/docs/#/?id=model-binding-and-validation)

若要将请求正文绑定到类型中，请使用模型绑定。我们目前支持 `JSON`、`JSONProtobuf` 、`Protobuf` 、`MsgPack`、`XML`、`YAML` 和标准形式值 (foo=bar&boo=baz) 的绑定。

```go
ReadJSON(outPtr interface{}) error
ReadJSONProtobuf(ptr proto.Message, opts ...ProtoUnmarshalOptions) error
ReadProtobuf(ptr proto.Message) error
ReadMsgPack(ptr interface{}) error
ReadXML(outPtr interface{}) error
ReadYAML(outPtr interface{}) error
ReadForm(formObject interface{}) error
ReadQuery(ptr interface{}) error
```

使用`ReadBody`时，Iris 尝试根据`Content-Type`标头推断。 如果您确定要绑定什么，则可以使用特定的`ReadXXX`方法，例如`ReadJSON`或`ReadProtobuf`等。

```go
ReadBody(ptr interface{}) error
```

明智的是，Iris没有内置的数据验证功能。但是，它确实允许您附加一个验证器，该验证器将自动调用诸如`ReadJSON`和`ReadXML`等，在此示例中，我们将学习如何使用 [go-playground/validator/v10](https://www.iris-go.com/docs/(https://github.com/go-playground/validator)) 进行请求正文验证。

请注意，您需要在要绑定的所有字段上设置相应的绑定标记。例如，从 JSON 绑定时，请设置 `json:"fieldname"`。

您还可以指定特定字段是必需的。 如果某个字段使用`validate:"required"`修饰，并且绑定时为空值，则会返回错误。

```go
package main

import (
    "fmt"

    "github.com/kataras/iris/v12"
    "github.com/go-playground/validator/v10"
)

func main() {
    app := iris.New()
    app.Validator = validator.New()

    userRouter := app.Party("/user")
    {
        userRouter.Get("/validation-errors", resolveErrorsDocumentation)
        userRouter.Post("/", postUser)
    }

    app.Listen(":8080")
}

// User contains user information.
type User struct {
    FirstName      string     `json:"fname" validate:"required"`
    LastName       string     `json:"lname" validate:"required"`
    Age            uint8      `json:"age" validate:"gte=0,lte=130"`
    Email          string     `json:"email" validate:"required,email"`
    FavouriteColor string     `json:"favColor" validate:"hexcolor|rgb|rgba"`
    Addresses      []*Address `json:"addresses" validate:"required,dive,required"`
}

// Address houses a users address information.
type Address struct {
    Street string `json:"street" validate:"required"`
    City   string `json:"city" validate:"required"`
    Planet string `json:"planet" validate:"required"`
    Phone  string `json:"phone" validate:"required"`
}

type validationError struct {
    ActualTag string `json:"tag"`
    Namespace string `json:"namespace"`
    Kind      string `json:"kind"`
    Type      string `json:"type"`
    Value     string `json:"value"`
    Param     string `json:"param"`
}

func wrapValidationErrors(errs validator.ValidationErrors) []validationError {
    validationErrors := make([]validationError, 0, len(errs))
    for _, validationErr := range errs {
        validationErrors = append(validationErrors, validationError{
            ActualTag: validationErr.ActualTag(),
            Namespace: validationErr.Namespace(),
            Kind:      validationErr.Kind().String(),
            Type:      validationErr.Type().String(),
            Value:     fmt.Sprintf("%v", validationErr.Value()),
            Param:     validationErr.Param(),
        })
    }

    return validationErrors
}

func postUser(ctx iris.Context) {
	var user User
	err := ctx.ReadJSON(&user)
	if err != nil {
		// 处理错误，下面你会找到正确的方法......
		if errs, ok := err.(validator.ValidationErrors); ok {
			// Wrap the errors with JSON format, the underline library returns the errors as interface.
			validationErrors := wrapValidationErrors(errs)

			// 触发 application/json + problem(问题描述) 响应并停止处理程序链.
			ctx.StopWithProblem(iris.StatusBadRequest, iris.NewProblem().
				Title("Validation error").
				Detail("One or more fields failed to be validated").
				Type("/user/validation-errors").
				Key("errors", validationErrors))

			return
		}

		// 这可能是一个内部 JSON 错误，此示例中不要提供更多信息.
		ctx.StopWithStatus(iris.StatusInternalServerError)
		return
	}

	ctx.JSON(iris.Map{"message": "OK"})
}

func resolveErrorsDocumentation(ctx iris.Context) {
    ctx.WriteString("A page that should document to web developers or users of the API on how to resolve the validation errors")
}
```

**示例请求**

```json
{
    "fname": "",
    "lname": "",
    "age": 45,
    "email": "mail@example.com",
    "favColor": "#000",
    "addresses": [{
        "street": "Eavesdown Docks",
        "planet": "Persphone",
        "phone": "none",
        "city": "Unknown"
    }]
}
```

**示例响应**

```json
{
    "title": "Validation error",
    "detail": "One or more fields failed to be validated",
    "type": "http://localhost:8080/user/validation-errors",
    "status": 400,
    "errors": [
        {
            "tag": "required",
            "namespace": "User.FirstName",
            "kind": "string",
            "type": "string",
            "value": "",
            "param": ""
        },
        {
            "tag": "required",
            "namespace": "User.LastName",
            "kind": "string",
            "type": "string",
            "value": "",
            "param": ""
        }
    ]
}
```

有关模型验证的更多信息，请访问：https://github.com/go-playground/validator/blob/master/_examples

### [绑定查询字符串](https://www.iris-go.com/docs/#/?id=bind-query-string)

`ReadQuery`方法只绑定查询参数而不绑定 post 数据，使用`ReadForm`代替绑定 post 数据。

```go
package main

import "github.com/kataras/iris/v12"

type Person struct {
    // 为空时返回: name is empty
    Name    string `url:"name,required"`
    // 为空时返回: Key: 'Person.Address' Error:Field validation for 'Address' failed on the 'required' tag
    Address string `url:"address,required" validate:"required"`
}

func main() {
    app := iris.Default()
    app.Any("/", index)
    app.Listen(":8080")
}

func index(ctx iris.Context) {
    var person Person
    if err := ctx.ReadQuery(&person); err!=nil {
        ctx.StopWithError(iris.StatusBadRequest, err)
        return
    }

    ctx.Application().Logger().Infof("Person: %#+v", person)
    ctx.WriteString("Success")
}
```

> 注意: 在当前版本(v10.10.0)中，只有当查询参数不全部为空时才会触发校验，否则将会跳过校验。

### [绑定任何](https://www.iris-go.com/docs/#/?id=bind-any)

将请求正文绑定到"ptr"，具体取决于客户端发送数据的内容类型，例如 JSON、XML、YAML、MessagePack、Protobuf、Form 和 URL Query。

```go
package main

import (
    "time"

    "github.com/kataras/iris/v12"
)

type Person struct {
        Name       string    `form:"name" json:"name" url:"name" msgpack:"name"` 
        Address    string    `form:"address" json:"address" url:"address" msgpack:"address"`
        Birthday   time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1" json:"birthday" url:"birthday" msgpack:"birthday"`
        CreateTime time.Time `form:"createTime" time_format:"unixNano" json:"create_time" url:"create_time" msgpack:"createTime"`
        UnixTime   time.Time `form:"unixTime" time_format:"unix" json:"unix_time" url:"unix_time" msgpack:"unixTime"`
}

func main() {
    app := iris.Default()
    app.Any("/", index)
    app.Listen(":8080")
}

func index(ctx iris.Context) {
    var person Person
    if err := ctx.ReadBody(&person); err!=nil {
        ctx.StopWithError(iris.StatusBadRequest, err)
        return
    }

    ctx.Application().Logger().Infof("Person: %#+v", person)
    ctx.WriteString("Success")
}
```

通过以下方式进行测试：

```sh
$ curl -X GET "localhost:8085/testing?name=kataras&address=xyz&birthday=1992-03-15&createTime=1562400033000000123&unixTime=1562400033"
```

### [绑定 URL 路径参数](https://www.iris-go.com/docs/#/?id=bind-url-path-parameters)

```go
package main

import "github.com/kataras/iris/v12"

type myParams struct {
    Name string   `param:"name"`
    Age  int      `param:"age"`
    Tail []string `param:"tail"`
}
// All parameters are required, as we already know,
// the router will fire 404 if name or int or tail are missing.

func main() {
    app := iris.Default()
    app.Get("/{name}/{age:int}/{tail:path}", func(ctx iris.Context) {
        var p myParams
        if err := ctx.ReadParams(&p); err != nil {
            ctx.StopWithError(iris.StatusInternalServerError, err)
            return
        }

        ctx.Writef("myParams: %#v", p)
    })
    app.Listen(":8088")
}
```

**请求**

```sh
$ curl -v http://localhost:8080/kataras/27/iris/web/framework
```

### [绑定标头](https://www.iris-go.com/docs/#/?id=bind-header)

```go
package main

import "github.com/kataras/iris/v12"


type myHeaders struct {
    RequestID      string `header:"X-Request-Id,required"`
    Authentication string `header:"Authentication,required"`
}

func main() {
    app := iris.Default()
    r.GET("/", func(ctx iris.Context) {
        var hs myHeaders
        if err := ctx.ReadHeaders(&hs); err != nil {
            ctx.StopWithError(iris.StatusInternalServerError, err)
            return
        }

        ctx.JSON(hs)
    })

    app.Listen(":8080")
}
```

**请求**

```sh
curl -H "x-request-id:373713f0-6b4b-42ea-ab9f-e2e04bc38e73" -H "authentication: Bearer my-token" \
http://localhost:8080
```

**响应**

```json
{
  "RequestID": "373713f0-6b4b-42ea-ab9f-e2e04bc38e73",
  "Authentication": "Bearer my-token"
}
```

### [绑定 HTML 复选框](https://www.iris-go.com/docs/#/?id=bind-html-checkboxes)

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()
    app.RegisterView(iris.HTML("./templates", ".html"))

    app.Get("/", showForm)
    app.Post("/", handleForm)

    app.Listen(":8080")
}

func showForm(ctx iris.Context) {
    ctx.View("form.html")
}

type formExample struct {
    Colors []string `form:"colors[]"` // or just "colors".
}

func handleForm(ctx iris.Context) {
    var form formExample
    err := ctx.ReadForm(&form)
    if err != nil {
        ctx.StopWithError(iris.StatusBadRequest, err)
        return
    }

    ctx.JSON(iris.Map{"Colors": form.Colors})
}
```

**templates/form.html**

```html
<form action="/" method="POST">
    <p>Check one or more colors</p>

    <label for="red">Red</label>
    <!-- name can be "colors" too -->
    <input type="checkbox" name="colors[]" value="red" id="red">
    <label for="green">Green</label>
    <input type="checkbox" name="colors[]" value="green" id="green">
    <label for="blue">Blue</label>
    <input type="checkbox" name="colors[]" value="blue" id="blue">
    <input type="submit">
</form>
```

**响应**

```json
{
  "Colors": [
    "red",
    "green",
    "blue"
  ]
}
```

### [JSON，JSONP，XML，Markdown，YAML 和 MsgPack rendering](https://www.iris-go.com/docs/#/?id=json-jsonp-xml-markdown-yaml-and-msgpack-rendering)

可以[在此处](https://github.com/kataras/iris/tree/master/_examples/response-writer/write-rest)找到详细示例。

```go
func main() {
    app := iris.New()

    // iris.Map is an alias of map[string]interface{}
    app.Get("/json", func(ctx iris.Context) {
        ctx.JSON(iris.Map{"message": "hello", "status": iris.StatusOK})
    })

    // Use Secure field to prevent json hijacking.
    // It prepends `"while(1),"` to the body when the data is array.
    app.Get("/json_secure", func(ctx iris.Context) {
        response := []string{"val1", "val2", "val3"}
        options := iris.JSON{Indent: "", Secure: true}
        ctx.JSON(response, options)

        // Will output: while(1);["val1","val2","val3"]
    })

    // Use ASCII field to generate ASCII-only JSON
    // with escaped non-ASCII characters.
    app.Get("/json_ascii", func(ctx iris.Context) {
        response := iris.Map{"lang": "GO-虹膜", "tag": "<br>"}
        options := iris.JSON{Indent: "    ", ASCII: true}
        ctx.JSON(response, options)

        /* Will output:
           {
               "lang": "GO-\u8679\u819c",
               "tag": "\u003cbr\u003e"
           }
        */
    })

    // Normally, JSON replaces special HTML characters with their unicode entities.
    // If you want to encode such characters literally,
    // you SHOULD set the UnescapeHTML field to true.
    app.Get("/json_raw", func(ctx iris.Context) {
        options := iris.JSON{UnescapeHTML: true}
        ctx.JSON(iris.Map{
            "html": "<b>Hello, world!</b>",
        }, options)

        // Will output: {"html":"<b>Hello, world!</b>"}
    })

    app.Get("/json_struct", func(ctx iris.Context) {
        // You also can use a struct.
        var msg struct {
            Name    string `json:"user"`
            Message string
            Number  int
        }
        msg.Name = "Mariah"
        msg.Message = "hello"
        msg.Number = 42
        // Note that msg.Name becomes "user" in the JSON.
        // Will output: {"user": "Mariah", "Message": "hello", "Number": 42}
        ctx.JSON(msg)
    })

    app.Get("/jsonp", func(ctx iris.Context) {
        ctx.JSONP(iris.Map{"hello": "jsonp"}, iris.JSONP{Callback: "callbackName"})
    })

    app.Get("/xml", func(ctx iris.Context) {
        ctx.XML(iris.Map{"message": "hello", "status": iris.StatusOK})
    })

    app.Get("/markdown", func(ctx iris.Context) {
        ctx.Markdown([]byte("# Hello Dynamic Markdown -- iris"))
    })

    app.Get("/yaml", func(ctx iris.Context) {
        ctx.YAML(iris.Map{"message": "hello", "status": iris.StatusOK})
    })

    app.Get("/msgpack", func(ctx iris.Context) {
        u := User{
            Firstname: "John",
            Lastname:  "Doe",
            City:      "Neither FBI knows!!!",
            Age:       25,
        }

        ctx.MsgPack(u)
    })

    // Render using jsoniter instead of the encoding/json:
    app.Listen(":8080", iris.WithOptimizations)
}
```

#### [原型布夫](https://www.iris-go.com/docs/#/?id=protobuf)

Iris 支持本机原型布，支持使用原版进行 JSON 编码和解码。`Protobuf`

```go
package main

import (
    "app/protos"

    "github.com/kataras/iris/v12"
)

func main() {
    app := iris.New()

    app.Get("/", send)
    app.Get("/json", sendAsJSON)
    app.Post("/read", read)
    app.Post("/read_json", readFromJSON)

    app.Listen(":8080")
}

func send(ctx iris.Context) {
    response := &protos.HelloReply{Message: "Hello, World!"}
    ctx.Protobuf(response)
}

func sendAsJSON(ctx iris.Context) {
    response := &protos.HelloReply{Message: "Hello, World!"}
    options := iris.JSON{
        Proto: iris.ProtoMarshalOptions{
            AllowPartial: true,
            Multiline:    true,
            Indent:       "    ",
        },
    }

    ctx.JSON(response, options)
}

func read(ctx iris.Context) {
    var request protos.HelloRequest

    err := ctx.ReadProtobuf(&request)
    if err != nil {
        ctx.StopWithError(iris.StatusBadRequest, err)
        return
    }

    ctx.Writef("HelloRequest.Name = %s", request.Name)
}

func readFromJSON(ctx iris.Context) {
    var request protos.HelloRequest

    err := ctx.ReadJSONProtobuf(&request)
    if err != nil {
        ctx.StopWithError(iris.StatusBadRequest, err)
        return
    }

    ctx.Writef("HelloRequest.Name = %s", request.Name)
}
```

### [提供静态文件](https://www.iris-go.com/docs/#/?id=serving-static-files)

```go
func main() {
    app := iris.New()
    app.Favicon("./resources/favicon.ico")
    app.HandleDir("/assets", iris.Dir("./assets"))

    app.Listen(":8080")
}
```

该方法接受的第三个可选参数：`HandleDir``DirOptions`

```go
type DirOptions struct {
    // Defaults to "/index.html", if request path is ending with **/*/$IndexName
    // then it redirects to **/*(/) which another handler is handling it,
    // that another handler, called index handler, is auto-registered by the framework
    // if end developer does not managed to handle it by hand.
    IndexName string
    // PushTargets filenames (map's value) to
    // be served without additional client's requests (HTTP/2 Push)
    // when a specific request path (map's key WITHOUT prefix)
    // is requested and it's not a directory (it's an `IndexFile`).
    //
    // Example:
    //     "/": {
    //         "favicon.ico",
    //         "js/main.js",
    //         "css/main.css",
    //     }
    PushTargets map[string][]string
    // PushTargetsRegexp like `PushTargets` but accepts regexp which
    // is compared against all files under a directory (recursively).
    // The `IndexName` should be set.
    //
    // Example:
    // "/": regexp.MustCompile("((.*).js|(.*).css|(.*).ico)$")
    // See `iris.MatchCommonAssets` too.
    PushTargetsRegexp map[string]*regexp.Regexp

    // Cache to enable in-memory cache and pre-compress files.
    Cache DirCacheOptions
    // When files should served under compression.
    Compress bool

    // List the files inside the current requested directory if `IndexName` not found.
    ShowList bool
    // If `ShowList` is true then this function will be used instead
    // of the default one to show the list of files of a current requested directory(dir).
    // See `DirListRich` package-level function too.
    DirList DirListFunc

    // Files downloaded and saved locally.
    Attachments Attachments

    // Optional validator that loops through each requested resource.
    AssetValidator func(ctx *context.Context, name string) bool
}
```

了解有关[文件服务器](https://github.com/kataras/iris/tree/master/_examples/file-server)的更多信息。

### [从上下文提供数据](https://www.iris-go.com/docs/#/?id=serving-data-from-context)

```go
SendFile(filename string, destinationName string) error
SendFileWithRate(src, destName string, limit float64, burst int) error
```

**用法**

强制将文件发送到客户端：

```go
func handler(ctx iris.Context) {
    src := "./files/first.zip"
    ctx.SendFile(src, "client.zip")
}
```

将下载速度限制为 ~50Kb/s，突发为 100KB：

```go
func handler(ctx iris.Context) {
    src := "./files/big.zip"
    // optionally, keep it empty to resolve the filename based on the "src".
    dest := "" 

    limit := 50.0 * iris.KB
    burst := 100 * iris.KB
    ctx.SendFileWithRate(src, dest, limit, burst)
}
ServeContent(content io.ReadSeeker, filename string, modtime time.Time)
ServeContentWithRate(content io.ReadSeeker, filename string, modtime time.Time, limit float64, burst int)

ServeFile(filename string) error
ServeFileWithRate(filename string, limit float64, burst int) error
```

**用法**

```go
func handler(ctx iris.Context) {
    ctx.ServeFile("./public/main.js")
}
```

### [模板呈现](https://www.iris-go.com/docs/#/?id=template-rendering)

Iris 支持 8 个开箱即用的模板引擎，开发人员仍然可以使用任何外部 golang 模板引擎，就像 .`Context.ResponseWriter()``io.Writer`

所有模板引擎共享一个通用API，即使用嵌入式资源，布局和特定于参与方的布局，模板功能，部分渲染等进行解析。

| #    | 名字   | 解析 器                                                |
| ---- | ---------- | :------------------------------------------------------ |
| 1    | HTML       | [html/template](https://pkg.go.dev/html/template)       |
| 2    | Blocks     | [kataras/blocks](https://github.com/kataras/blocks)     |
| 3    | Django     | [flosch/pongo2](https://github.com/flosch/pongo2)       |
| 4    | Pug        | [Joker/jade](https://github.com/Joker/jade)             |
| 5    | Handlebars | [aymerick/raymond](https://github.com/aymerick/raymond) |
| 6    | Amber      | [eknkc/amber](https://github.com/eknkc/amber)           |
| 7    | Jet        | [CloudyKit/jet](https://github.com/CloudyKit/jet)       |
| 8    | Ace        | [yosssi/ace](https://github.com/yosssi/ace)             |

[示例列表](https://github.com/kataras/iris/tree/master/_examples/view)。

[基准测试列表](https://dev.to/kataras/what-s-the-fastest-template-parser-in-go-4bal)。

可以按参与方注册视图引擎。要**注册**视图引擎，请使用如下所示的方法。`Application/Party.RegisterView(ViewEngine)`

从扩展名为".html"的"./views"文件夹中加载所有模板，并使用标准包解析它们。`html/template`

```go
// [app := iris.New...]
tmpl := iris.HTML("./views", ".html")
app.RegisterView(tmpl)
```

要**渲染或执行**视图，请使用主路由处理程序中的方法。`Context.View`

```go
ctx.View("hi.html")
```

要通过中间件或主处理程序将 Go 值与视图中的键值模式**绑定**，请使用该方法之前的方法。`Context.ViewData``Context.View`

绑定：与 .`{{.message}}``"Hello world!"`

```go
ctx.ViewData("message", "Hello world!")
```

根绑定：

```go
ctx.View("user-page.html", User{})

// root binding as {{.Name}}
```

要**添加模板函数**，请使用首选视图引擎的方法。`AddFunc`

```go
//       func name, input arguments, render value
tmpl.AddFunc("greet", func(s string) string {
    return "Greetings " + s + "!"
})
```

要**在每个请求上重新加载**，请调用视图引擎的方法。`Reload`

```go
tmpl.Reload(true)
```

要使用**嵌入式**模板而不依赖于本地文件系统，请使用 [go-bindata](https://github.com/go-bindata/go-bindata) 外部工具，并将其生成的函数传递给首选视图引擎的第一个输入参数。`AssetFile()`

```go
 tmpl := iris.HTML(AssetFile(), ".html")
```

示例代码：

```go
// file: main.go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()

    // Parse all templates from the "./views" folder
    // where extension is ".html" and parse them
    // using the standard `html/template` package.
    tmpl := iris.HTML("./views", ".html")
    // Set custom delimeters.
    tmpl.Delims("{{", "}}")
    // Enable re-build on local template files changes.
    tmpl.Reload(true)

    // Default template funcs are:
    //
    // - {{ urlpath "myNamedRoute" "pathParameter_ifNeeded" }}
    // - {{ render "header.html" }}
    // and partial relative path to current page:
    // - {{ render_r "header.html" }} 
    // - {{ yield }}
    // - {{ current }}
    // Register a custom template func:
    tmpl.AddFunc("greet", func(s string) string {
        return "Greetings " + s + "!"
    })

    // Register the view engine to the views,
    // this will load the templates.
    app.RegisterView(tmpl)

    // Method:    GET
    // Resource:  http://localhost:8080
    app.Get("/", func(ctx iris.Context) {
        // Bind: {{.message}} with "Hello world!"
        ctx.ViewData("message", "Hello world!")
        // Render template file: ./views/hi.html
        ctx.View("hi.html")
    })

    app.Listen(":8080")
}
<!-- file: ./views/hi.html -->
<html>
<head>
    <title>Hi Page</title>
</head>
<body>
    <h1>{{.message}}</h1>
    <strong>{{greet "to you"}}</strong>
</body>
</html>
```

在 [http://localhost:8080](http://localhost:8080/) 处打开浏览器选项卡。

**呈现的结果**将如下所示：

```html
<html>
<head>
    <title>Hi Page</title>
</head>
<body>
    <h1>Hello world!</h1>
    <strong>Greetings to you!</strong>
</body>
</html>
```

### [多模板](https://www.iris-go.com/docs/#/?id=multitemplate)

Iris 允许每个应用程序使用无限数量的已注册视图引擎。除此之外，您还可以**为每个参与方或通过中间件**注册一个视图引擎！

```go
// Register a view engine per group of routes.
adminGroup := app.Party("/admin")
adminGroup.RegisterView(iris.Blocks("./views/admin", ".html"))
```

#### [通过中间件](https://www.iris-go.com/docs/#/?id=through-middleware)

```go
func middleware(views iris.ViewEngine) iris.Handler {
    return func(ctx iris.Context) {
        ctx.ViewEngine(views)
        ctx.Next()
    }
}
```

**用法**

```go
// Register a view engine on-fly for the current chain of handlers.
views := iris.Blocks("./views/on-fly", ".html")
views.Load()

app.Get("/", setViews(views), onFly)
```

### [重定向](https://www.iris-go.com/docs/#/?id=redirects)

发出 HTTP 重定向很容易。支持内部和外部位置。我们所说的位置是指路径、子域、域和 e.t.c。

#### [从处理程序](https://www.iris-go.com/docs/#/?id=from-handler)

```go
app.Get("/", func(ctx iris.Context) {
    ctx.Redirect("https://golang.org/dl", iris.StatusMovedPermanently)
})
```

从 POST 发出 HTTP 重定向。

```go
app.Post("/", func(ctx iris.Context) {
    ctx.Redirect("/login", iris.StatusFound)
})
```

从处理程序发出本地路由器重定向，使用或如下所示。`Application.ServeHTTPC``Exec()`

```go
app.Get("/test", func(ctx iris.Context) {
    r := ctx.Request()
    r.URL.Path = "/test2"

    ctx.Application().ServeHTTPC(ctx)
    // OR
    // ctx.Exec("GET", "/test2")
})

app.Get("/test2", func(ctx iris.Context) {
    ctx.JSON(iris.Map{"hello": "world"})
})
```

#### [全球](https://www.iris-go.com/docs/#/?id=globally)

使用我们都喜欢的语法。

```go
import "github.com/kataras/iris/v12/middleware/rewrite"
func main() {
    app := iris.New()
    // [...routes]
    redirects := rewrite.Load("redirects.yml")
    app.WrapRouter(redirects)
    app.Listen(":80")
}
```

该文件如下所示：`"redirects.yml"`

```yaml
RedirectMatch:
  # Redirects /seo/* to /*
  - 301 /seo/(.*) /$1

  # Redirects /docs/v12* to /docs
  - 301 /docs/v12(.*) /docs

  # Redirects /old(.*) to /
  - 301 /old(.*) /

  # Redirects http or https://test.* to http or https://newtest.*
  - 301 ^(http|https)://test.(.*) $1://newtest.$2

  # Handles /*.json or .xml as *?format=json or xml,
  # without redirect. See /users route.
  # When Code is 0 then it does not redirect the request,
  # instead it changes the request URL
  # and leaves a route handle the request.
  - 0 /(.*).(json|xml) /$1?format=$2

# Redirects root domain to www.
# Creation of a www subdomain inside the Application is unnecessary,
# all requests are handled by the root Application itself.
PrimarySubdomain: www
```

完整的代码可以在[重写中间件示例](https://github.com/kataras/iris/tree/master/_examples/routing/rewrite)中找到。

### [定制中间件](https://www.iris-go.com/docs/#/?id=custom-middleware)

```go
func Logger() iris.Handler {
    return func(ctx iris.Context) {
        t := time.Now()

        // Set a shared variable between handlers
        ctx.Values().Set("framework", "iris")

        // before request

        ctx.Next()

        // after request
        latency := time.Since(t)
        log.Print(latency)

        // access the status we are sending
        status := ctx.GetStatusCode()
        log.Println(status)
    }
}

func main() {
    app := iris.New()
    app.Use(Logger())

    app.Get("/test", func(ctx iris.Context) {
        // retrieve a value set by the middleware.
        framework := ctx.Values().GetString("framework")

        // it would print: "iris"
        log.Println(framework)
    })

    app.Listen(":8080")
}
```

### [使用基本身份验证](https://www.iris-go.com/docs/#/?id=using-basic-authentication)

HTTP 基本身份验证是对 Web 资源实施访问控制的最简单技术，因为它不需要 Cookie、会话标识符或登录页面;相反，HTTP 基本身份验证使用 HTTP 标头中的标准字段。

基本身份验证中间件[包含在](https://github.com/kataras/iris/tree/master/middleware/basicauth) Iris 框架中，因此您无需单独安装它。

**1.** 导入中间件

```go
import "github.com/kataras/iris/v12/middleware/basicauth"
```

**2.** 配置中间件及其结构：`Options`

```go
opts := basicauth.Options{
    Allow: basicauth.AllowUsers(map[string]string{
        "username": "password",
    }),
    Realm:        "Authorization Required",
    ErrorHandler: basicauth.DefaultErrorHandler,
    // [...more options]
}
```

**3.** 初始化中间件：

```go
auth := basicauth.New(opts)
```

**3.1** 以上步骤与函数相同：`Default`

```go
auth := basicauth.Default(map[string]string{
    "username": "password",
})
```

**3.2** 使用自定义用户切片：

```go
// The struct value MUST contain a Username and Passwords fields
// or GetUsername() string and GetPassword() string methods.
type User struct {
    Username string
    Password string
}

// [...]
auth := basicauth.Default([]User{...})
```

**3.3** 从文件加载用户（可选），密码使用 [golang.org/x/crypto/bcrypt](https://golang.org/x/crypto/bcrypt) 包加密：

```go
auth := basicauth.Load("users.yml", basicauth.BCRYPT)
```

**3.3.1** 同样可以使用（推荐）：`Options`

```go
opts := basicauth.Options{
    Allow: basicauth.AllowUsersFile("users.yml", basicauth.BCRYPT),
    Realm: basicauth.DefaultRealm,
    // [...more options]
}

auth := basicauth.New(opts)
```

其中可能看起来像这样：`users.yml`

```yaml
- username: kataras
  password: $2a$10$Irg8k8HWkDlvL0YDBKLCYee6j6zzIFTplJcvZYKA.B8/clHPZn2Ey
  # encrypted of kataras_pass
  role: admin
- username: makis
  password: $2a$10$3GXzp3J5GhHThGisbpvpZuftbmzPivDMo94XPnkTnDe7254x7sJ3O
  # encrypted of makis_pass
  role: member
```

**4.** 注册中间件：

```go
// Register to all matched routes
// under a Party and its children.
app.Use(auth)

// OR/and register to all http error routes.
app.UseError(auth)

// OR register under a path prefix of a specific Party,
// including all http errors of this path prefix.
app.UseRouter(auth)

// OR register to a specific Route before its main handler.
app.Post("/protected", auth, routeHandler)
```

**5.** 检索用户名和密码：

```go
func routeHandler(ctx iris.Context) {
    username, password, _ := ctx.Request().BasicAuth()
    // [...]
}
```

**5.1** 检索 User 值（在 以下位置注册自定义用户结构切片时很有用）：`Options.AllowUsers`

```go
func routeHandler(ctx iris.Context) {
    user := ctx.User().(*iris.SimpleUser)
    // user.Username
    // user.Password
}
```

阅读[有关_examples/身份验证](https://github.com/kataras/iris/tree/master/_examples/auth)的更多授权和身份验证示例。

### [中间件内部的 Goroutines](https://www.iris-go.com/docs/#/?id=goroutines-inside-a-middleware)

在中间件或处理程序中启动新的 Goroutines 时，**不应**在其中使用原始上下文，必须使用只读副本。

```go
func main() {
    app := iris.Default()

    app.Get("/long_async", func(ctx iris.Context) {
        // create a clone to be used inside the goroutine
        ctxCopy := ctx.Clone()
        go func() {
            // simulate a long task with time.Sleep(). 5 seconds
            time.Sleep(5 * time.Second)

            // note that you are using the copied context "ctxCopy", IMPORTANT
            log.Printf("Done! in path: %s", ctxCopy.Path())
        }()
    })

    app.Get("/long_sync", func(ctx iris.Context) {
        // simulate a long task with time.Sleep(). 5 seconds
        time.Sleep(5 * time.Second)

        // since we are NOT using a goroutine, we do not have to copy the context
        log.Printf("Done! in path: %s", ctx.Path())
    })

    app.Listen(":8080")
}
```

### [自定义 HTTP 配置](https://www.iris-go.com/docs/#/?id=custom-http-configuration)

有关 http 服务器配置的 12 个以上示例可以在 [_examples/http-server](https://github.com/kataras/iris/tree/master/_examples/http-server) 文件夹中找到。

直接使用，如下所示：`http.ListenAndServe()`

```go
func main() {
    app := iris.New()
    // [...routes]
    if err := app.Build(); err!=nil{
        panic(err)
    }
    http.ListenAndServe(":8080", app)
}
```

请注意**，在将其**用作 .`Build``http.Handler`

另一个例子：

```go
func main() {
    app := iris.New()
    // [...routes]
    app.Build()

    srv := &http.Server{
        Addr:           ":8080",
        Handler:        app,
        ReadTimeout:    10 * time.Second,
        WriteTimeout:   10 * time.Second,
        MaxHeaderBytes: 1 << 20,
    }
    srv.ListenAndServe()
}
```

但是，您很少需要具有 Iris 的外部实例。您可以使用任何 tcp 侦听器、http 服务器或自定义函数通过方法进行侦听。`http.Server``Application.Run`

```go
app.Run(iris.Listener(l net.Listener)) // listen using a custom net.Listener
app.Run(iris.Server(srv *http.Server)) // listen using a custom http.Server
app.Run(iris.Addr(addr string)) // the app.Listen is a shortcut of this method.
app.Run(iris.TLS(addr string, certFileOrContents, keyFileOrContents string)) // listen TLS.
app.Run(iris.AutoTLS(addr, domain, email string)) // listen using letsencrypt (see below).

// and any custom function that returns an error:
app.Run(iris.Raw(f func() error))
```

### [套接字分片](https://www.iris-go.com/docs/#/?id=socket-sharding)

此选项允许在**多 CPU** 服务器上线性扩展服务器性能。有关详细信息[，请参阅 https://www.nginx.com/blog/socket-sharding-nginx-release-1-9-1/](https://www.nginx.com/blog/socket-sharding-nginx-release-1-9-1/)。使用配置器启用。`iris.WithSocketSharding`

*示例代码：*

```go
package main

import (
    "time"

    "github.com/kataras/iris/v12"
)

func main() {
    startup := time.Now()

    app := iris.New()
    app.Get("/", func(ctx iris.Context) {
        s := startup.Format(ctx.Application().ConfigurationReadOnly().GetTimeFormat())
        ctx.Writef("This server started at: %s\n", s)
    })

    app.Listen(":8080", iris.WithSocketSharding)
    // or app.Run(..., iris.WithSocketSharding)
}
```

### [支持让我们加密](https://www.iris-go.com/docs/#/?id=support-let39s-encrypt)

1 行 LetsEncrypt HTTPS 服务器的示例。

```go
package main

import (
    "log"

    "github.com/iris-gonic/autotls"
    "github.com/kataras/iris/v12"
)

func main() {
    app := iris.Default()

    // Ping handler
    app.Get("/ping", func(ctx iris.Context) {
        ctx.WriteString("pong")
    })

    app.Run(iris.AutoTLS(":443", "example.com example2.com", "mail@example.com"))
}
```

自定义 TLS 示例（您也可以绑定自动证书管理器）：

```go
app.Run(
    iris.TLS(":443", "", "", func(su *iris.Supervisor) {
        su.Server.TLSConfig = &tls.Config{
            /* your custom fields */
        },
    }),
)
```

> 所有方法（如：Addr、TLS、AutoTLS、Server、Listener 和 e.t.c接受可变参数输入参数，以在构建状态下配置 http 服务器实例。`iris.Runner``func(*iris.Supervisor)`

### [使用 Iris 运行多个服务](https://www.iris-go.com/docs/#/?id=run-multiple-service-using-iris)

```go
package main

import (
    "log"
    "net/http"
    "time"

    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/middleware/recover"

    "golang.org/x/sync/errgroup"
)

var g errgroup.Group

func startApp1() error {
    app := iris.New().SetName("app1")
    app.Use(recover.New())
    app.Get("/", func(ctx iris.Context) {
        app.Get("/", func(ctx iris.Context) {
            ctx.JSON(iris.Map{
                "code":  iris.StatusOK,
                "message": "Welcome server 1",
            })
        })
    })

    app.Build()
   return app.Listen(":8080")
}

func startApp2() error {
    app := iris.New().SetName("app2")
    app.Use(recover.New())
    app.Get("/", func(ctx iris.Context) {
        ctx.JSON(iris.Map{
            "code":  iris.StatusOK,
            "message": "Welcome server 2",
        })
    })

    return app.Listen(":8081")
}

func main() {
    g.Go(startApp1)
    g.Go(startApp2)

    if err := g.Wait(); err != nil {
        log.Fatal(err)
    }
}
```

通过软件包管理多个 Iris 实例。在此处阅读更多[内容](https://github.com/kataras/iris/blob/master/apps/README.md)。`apps`

### [正常关机或重新启动](https://www.iris-go.com/docs/#/?id=graceful-shutdown-or-restart)

有几种方法可用于执行正常关机或重新启动。您可以使用专门为此构建的第三方包，也可以使用该方法。可以[在此处](https://github.com/kataras/iris/tree/master/_examples/http-server/graceful-shutdown)找到示例。`app.Shutdown(context.Context)`

使用以下命令在 CTRL/CMD+C 上注册事件：`iris.RegisterOnInterrupt`

```go
idleConnsClosed := make(chan struct{})
iris.RegisterOnInterrupt(func() {
    timeout := 10 * time.Second
    ctx, cancel := stdContext.WithTimeout(stdContext.Background(), timeout)
    defer cancel()
    // close all hosts.
    app.Shutdown(ctx)
    close(idleConnsClosed)
})

// [...]
app.Listen(":8080", iris.WithoutInterruptHandler, iris.WithoutServerError(iris.ErrServerClosed))
<-idleConnsClosed
```

### [使用模板构建单个二进制文件](https://www.iris-go.com/docs/#/?id=build-a-single-binary-with-templates)

您可以使用 [go-bindata][[https://github.com/go-bindata/go-bindata\] 生成的](https://github.com/go-bindata/go-bindata]'s)函数将服务器构建为包含模板的单个二进制文件。`AssetFile`

```sh
$ go get -u github.com/go-bindata/go-bindata/...
$ go-bindata -fs -prefix "templates" ./templates/...
$ go run .
```

示例代码：

```go
func main() {
    app := iris.New()

    tmpl := iris.HTML(AssetFile(), ".html")
    tmpl.Layout("layouts/layout.html")
    tmpl.AddFunc("greet", func(s string) string {
        return "Greetings " + s + "!"
    })
    app.RegisterView(tmpl)

    // [...]
}
```

在[_examples/视图中](https://github.com/kataras/iris/tree/master/_examples/view)查看完整示例。

### [尝试将主体绑定到不同的结构中](https://www.iris-go.com/docs/#/?id=try-to-bind-body-into-different-structs)

绑定请求正文的正常方法会消耗，并且不能多次调用它们，**除非**将配置器传递给 。`ctx.Request().Body``iris.WithoutBodyConsumptionOnUnmarshal``app.Run/Listen`

```go
package main

import "github.com/kataras/iris/v12"

func main() {
    app := iris.New()

    app.Post("/", logAllBody, logJSON, logFormValues, func(ctx iris.Context) {
        // body, err := ioutil.ReadAll(ctx.Request().Body) once or
        body, err := ctx.GetBody() // as many times as you need.
        if err != nil {
            ctx.StopWithError(iris.StatusInternalServerError, err)
            return
        }

        if len(body) == 0 {
            ctx.WriteString(`The body was empty.`)
        } else {
            ctx.WriteString("OK body is still:\n")
            ctx.Write(body)
        }
    })

    app.Listen(":8080", iris.WithoutBodyConsumptionOnUnmarshal)
}

func logAllBody(ctx iris.Context) {
    body, err := ctx.GetBody()
    if err == nil && len(body) > 0 {
        ctx.Application().Logger().Infof("logAllBody: %s", string(body))
    }

    ctx.Next()
}

func logJSON(ctx iris.Context) {
    var p interface{}
    if err := ctx.ReadJSON(&p); err == nil {
        ctx.Application().Logger().Infof("logJSON: %#+v", p)
    }

    ctx.Next()
}

func logFormValues(ctx iris.Context) {
    values := ctx.FormValues()
    if values != nil {
        ctx.Application().Logger().Infof("logFormValues: %v", values)
    }

    ctx.Next()
}
```

可以使用 将结构绑定到基于客户端的内容类型的请求。您还可以使用[内容协商](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation)。下面是一个完整的示例：`ReadBody`

```go
package main

import (
    "github.com/kataras/iris/v12"
)

func main() {
    app := newApp()
    // See main_test.go for usage.
    app.Listen(":8080")
}

func newApp() *iris.Application {
    app := iris.New()
    // To automatically decompress using gzip:
    // app.Use(iris.GzipReader)

    app.Use(setAllowedResponses)

    app.Post("/", readBody)

    return app
}

type payload struct {
    Message string `json:"message" xml:"message" msgpack:"message" yaml:"Message" url:"message" form:"message"`
}

func readBody(ctx iris.Context) {
    var p payload

    // Bind request body to "p" depending on the content-type that client sends the data,
    // e.g. JSON, XML, YAML, MessagePack, Protobuf, Form and URL Query.
    err := ctx.ReadBody(&p)
    if err != nil {
        ctx.StopWithProblem(iris.StatusBadRequest,
            iris.NewProblem().Title("Parser issue").Detail(err.Error()))
        return
    }

    // For the sake of the example, log the received payload.
    ctx.Application().Logger().Infof("Received: %#+v", p)

    // Send back the payload depending on the accept content type and accept-encoding of the client,
    // e.g. JSON, XML and so on.
    ctx.Negotiate(p)
}

func setAllowedResponses(ctx iris.Context) {
    // Indicate that the Server can send JSON, XML, YAML and MessagePack for this request.
    ctx.Negotiation().JSON().XML().YAML().MsgPack()
    // Add more, allowed by the server format of responses, mime types here...

    // If client is missing an "Accept: " header then default it to JSON.
    ctx.Negotiation().Accept.JSON()

    ctx.Next()
}
```

### [HTTP2 服务器推送](https://www.iris-go.com/docs/#/?id=http2-server-push)

完整的示例代码可以在[_examples/response-writer/http2push](https://github.com/kataras/iris/tree/master/_examples/response-writer/http2push)上找到。

服务器推送允许服务器先发制人地将网站资产"推送"到客户端，而无需用户明确要求它们。谨慎使用时，我们可以发送我们知道用户将需要他们请求的页面的内容。

```go
package main

import (
    "net/http"

    "github.com/kataras/iris/v12"
)

func main() {
    app := iris.New()
    app.Get("/", pushHandler)
    app.Get("/main.js", simpleAssetHandler)

    app.Run(iris.TLS("127.0.0.1:443", "mycert.crt", "mykey.key"))
    // $ openssl req -new -newkey rsa:4096 -x509 -sha256 \
    // -days 365 -nodes -out mycert.crt -keyout mykey.key
}

func pushHandler(ctx iris.Context) {
    // The target must either be an absolute path (like "/path") or an absolute
    // URL that contains a valid host and the same scheme as the parent request.
    // If the target is a path, it will inherit the scheme and host of the
    // parent request.
    target := "/main.js"

    if pusher, ok := ctx.ResponseWriter().Naive().(http.Pusher); ok {
        err := pusher.Push(target, nil)
        if err != nil {
            if err == iris.ErrPushNotSupported {
                ctx.StopWithText(iris.StatusHTTPVersionNotSupported, "HTTP/2 push not supported.")
            } else {
                ctx.StopWithError(iris.StatusInternalServerError, err)
            }
            return
        }
    }

    ctx.HTML(`<html><body><script src="%s"></script></body></html>`, target)
}

func simpleAssetHandler(ctx iris.Context) {
    ctx.ServeFile("./public/main.js")
}
```

### [设置并获取饼干](https://www.iris-go.com/docs/#/?id=set-and-get-a-cookie)

安全 Cookie、编码和解码、会话（和会话缩放）、Flash 消息等可以在[_examples/Cookie](https://github.com/kataras/iris/tree/master/_examples/cookies) 和[_examples/会话](https://github.com/kataras/iris/tree/master/_examples/sessions)目录中找到。

```go
import "github.com/kataras/iris/v12"

func main() {
    app := iris.Default()

    app.Get("/cookie", func(ctx iris.Context) {
        value := ctx.GetCookie("my_cookie")

        if value == "" {
            value = "NotSet"
            ctx.SetCookieKV("my_cookie", value)
            // Alternatively: ctx.SetCookie(&http.Cookie{...})
            ctx.SetCookie("", "test", 3600, "/", "localhost", false, true)
        }

        ctx.Writef("Cookie value: %s \n", cookie)
    })

    app.Listen(":8080")
}
```

如果要设置自定义路径：

```go
ctx.SetCookieKV(name, value, iris.CookiePath("/custom/path/cookie/will/be/stored"))
```

如果希望仅对当前请求路径可见：

```go
ctx.SetCookieKV(name, value, iris.CookieCleanPath /* or iris.CookiePath("") */)
```

更多：

- `iris.CookieAllowReclaim`
- `iris.CookieAllowSubdomains`
- `iris.CookieSecure`
- `iris.CookieHTTPOnly`
- `iris.CookieSameSite`
- `iris.CookiePath`
- `iris.CookieCleanPath`
- `iris.CookieExpires`
- `iris.CookieEncoding`

您也可以在中间件中为整个请求添加 Cookie 选项：

```go
func setCookieOptions(ctx iris.Context) {
    ctx.AddCookieOptions(iris.CookieHTTPOnly(true), iris.CookieExpires(1*time.Hour))
    ctx.Next()
}
```

## [JSON 网络令牌](https://www.iris-go.com/docs/#/?id=json-web-tokens)

JSON Web Token （JWT） 是一种开放标准 （[RFC 7519](https://tools.ietf.org/html/rfc7519)），它定义了一种紧凑且独立的方式，用于将信息作为 JSON 对象在各方之间安全地传输。此信息可以进行验证和信任，因为它是经过数字签名的。JWT 可以使用密钥（使用 HMAC 算法）或使用 RSA 或 ECDSA 的公钥/私钥对进行签名。

### [何时应使用 JSON Web 令牌？](https://www.iris-go.com/docs/#/?id=when-should-you-use-json-web-tokens)

以下是 JSON Web 令牌有用的一些方案：

**授权**：这是使用 JWT 的最常见方案。用户登录后，每个后续请求都将包含 JWT，允许用户访问该令牌允许的路由、服务和资源。单点登录是当今广泛使用 JWT 的一项功能，因为它的开销很小，并且能够跨不同域轻松使用。

**信息交换**：JSON Web令牌是在各方之间安全传输信息的好方法。由于 JWT 可以签名（例如，使用公钥/私钥对），因此您可以确定发送方就是他们所说的人。此外，由于签名是使用标头和有效负载计算的，因此您还可以验证内容是否未被篡改。

> 阅读更多关于智威汤逊的信息，网址：https://jwt.io/introduction/

### [将 JWT 与虹膜配合使用](https://www.iris-go.com/docs/#/?id=using-jwt-with-iris)

Iris JWT [中间件](https://github.com/kataras/iris/tree/master/middleware/jwt)在设计时充分考虑了安全性、性能和简单性，可保护您的令牌免受[其他库中可能出现的关键漏洞的影响](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/)。它基于 [kataras/jwt](https://github.com/kataras/jwt) 包。

```go
package main

import (
    "time"

    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/middleware/jwt"
)

var (
    sigKey = []byte("signature_hmac_secret_shared_key")
    encKey = []byte("GCM_AES_256_secret_shared_key_32")
)

type fooClaims struct {
    Foo string `json:"foo"`
}

func main() {
    app := iris.New()

    signer := jwt.NewSigner(jwt.HS256, sigKey, 10*time.Minute)
    // Enable payload encryption with:
    // signer.WithEncryption(encKey, nil)
    app.Get("/", generateToken(signer))

    verifier := jwt.NewVerifier(jwt.HS256, sigKey)
    // Enable server-side token block feature (even before its expiration time):
    verifier.WithDefaultBlocklist()
    // Enable payload decryption with:
    // verifier.WithDecryption(encKey, nil)
    verifyMiddleware := verifier.Verify(func() interface{} {
        return new(fooClaims)
    })

    protectedAPI := app.Party("/protected")
    // Register the verify middleware to allow access only to authorized clients.
    protectedAPI.Use(verifyMiddleware)
    // ^ or UseRouter(verifyMiddleware) to disallow unauthorized http error handlers too.

    protectedAPI.Get("/", protected)
    // Invalidate the token through server-side, even if it's not expired yet.
    protectedAPI.Get("/logout", logout)

    // http://localhost:8080
    // http://localhost:8080/protected?token=$token (or Authorization: Bearer $token)
    // http://localhost:8080/protected/logout?token=$token
    // http://localhost:8080/protected?token=$token (401)
    app.Listen(":8080")
}

func generateToken(signer *jwt.Signer) iris.Handler {
    return func(ctx iris.Context) {
        claims := fooClaims{Foo: "bar"}

        token, err := signer.Sign(claims)
        if err != nil {
            ctx.StopWithStatus(iris.StatusInternalServerError)
            return
        }

        ctx.Write(token)
    }
}

func protected(ctx iris.Context) {
    // Get the verified and decoded claims.
    claims := jwt.Get(ctx).(*fooClaims)

    // Optionally, get token information if you want to work with them.
    // Just an example on how you can retrieve all the standard claims (set by signer's max age, "exp").
    standardClaims := jwt.GetVerifiedToken(ctx).StandardClaims
    expiresAtString := standardClaims.ExpiresAt().Format(ctx.Application().ConfigurationReadOnly().GetTimeFormat())
    timeLeft := standardClaims.Timeleft()

    ctx.Writef("foo=%s\nexpires at: %s\ntime left: %s\n", claims.Foo, expiresAtString, timeLeft)
}

func logout(ctx iris.Context) {
    err := ctx.Logout()
    if err != nil {
        ctx.WriteString(err.Error())
    } else {
        ctx.Writef("token invalidated, a new token is required to access the protected API")
    }
}
```

> 有关刷新令牌、黑名单等的信息，[请访问：_examples/auth/jwt](https://github.com/kataras/iris/tree/master/_examples/auth/jwt)。

## [测试](https://www.iris-go.com/docs/#/?id=testing)

Iris为httpexpect提供了令人难以置信的支持，[httpexpect](https://github.com/gavv/httpexpect)是Web应用程序的测试框架。该子包为 Iris + httpexpect 提供了帮助程序。`iris/httptest`

如果你更喜欢Go的标准[net/http/httptest](https://golang.org/pkg/net/http/httptest/)软件包，你仍然可以使用它。Iris每个http Web框架都与任何外部测试工具兼容，最后它是HTTP。

### [测试基本身份验证](https://www.iris-go.com/docs/#/?id=testing-basic-authentication)

在第一个示例中，我们将使用子包来测试基本身份验证。`iris/httptest`

**1.** 源文件如下所示：`main.go`

```go
package main

import (
    "github.com/kataras/iris/v12"
    "github.com/kataras/iris/v12/middleware/basicauth"
)

func newApp() *iris.Application {
    app := iris.New()

    opts := basicauth.Options{
        Allow: basicauth.AllowUsers(map[string]string{"myusername": "mypassword"}),
    }

    authentication := basicauth.New(opts) // or just: basicauth.Default(map...)

    app.Get("/", func(ctx iris.Context) { ctx.Redirect("/admin") })

    // to party

    needAuth := app.Party("/admin", authentication)
    {
        //http://localhost:8080/admin
        needAuth.Get("/", h)
        // http://localhost:8080/admin/profile
        needAuth.Get("/profile", h)

        // http://localhost:8080/admin/settings
        needAuth.Get("/settings", h)
    }

    return app
}

func h(ctx iris.Context) {
    // username, password, _ := ctx.Request().BasicAuth()
    // third parameter it will be always true because the middleware
    // makes sure for that, otherwise this handler will not be executed.
    // OR:

    user := ctx.User().(*iris.SimpleUser)
    ctx.Writef("%s %s:%s", ctx.Path(), user.Username, user.Password)
    // ctx.Writef("%s %s:%s", ctx.Path(), username, password)
}

func main() {
    app := newApp()
    app.Listen(":8080")
}
```

**2.**现在，创建一个文件并复制粘贴以下内容。`main_test.go`

```go
package main

import (
    "testing"

    "github.com/kataras/iris/v12/httptest"
)

func TestNewApp(t *testing.T) {
    app := newApp()
    e := httptest.New(t, app)

    // redirects to /admin without basic auth
    e.GET("/").Expect().Status(httptest.StatusUnauthorized)
    // without basic auth
    e.GET("/admin").Expect().Status(httptest.StatusUnauthorized)

    // with valid basic auth
    e.GET("/admin").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin myusername:mypassword")
    e.GET("/admin/profile").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin/profile myusername:mypassword")
    e.GET("/admin/settings").WithBasicAuth("myusername", "mypassword").Expect().
        Status(httptest.StatusOK).Body().Equal("/admin/settings myusername:mypassword")

    // with invalid basic auth
    e.GET("/admin/settings").WithBasicAuth("invalidusername", "invalidpassword").
        Expect().Status(httptest.StatusUnauthorized)

}
```

**3.** 打开命令行并执行：

```bash
$ go test -v
```

### [测试饼干](https://www.iris-go.com/docs/#/?id=testing-cookies)

```go
package main

import (
    "fmt"
    "testing"

    "github.com/kataras/iris/v12/httptest"
)

func TestCookiesBasic(t *testing.T) {
    app := newApp()
    e := httptest.New(t, app, httptest.URL("http://example.com"))

    cookieName, cookieValue := "my_cookie_name", "my_cookie_value"

    // Test Set A Cookie.
    t1 := e.GET(fmt.Sprintf("/cookies/%s/%s", cookieName, cookieValue)).
        Expect().Status(httptest.StatusOK)
    // Validate cookie's existence, it should be available now.
    t1.Cookie(cookieName).Value().Equal(cookieValue)
    t1.Body().Contains(cookieValue)

    path := fmt.Sprintf("/cookies/%s", cookieName)

    // Test Retrieve A Cookie.
    t2 := e.GET(path).Expect().Status(httptest.StatusOK)
    t2.Body().Equal(cookieValue)

    // Test Remove A Cookie.
    t3 := e.DELETE(path).Expect().Status(httptest.StatusOK)
    t3.Body().Contains(cookieName)

    t4 := e.GET(path).Expect().Status(httptest.StatusOK)
    t4.Cookies().Empty()
    t4.Body().Empty()
}
$ go test -v -run=TestCookiesBasic$
```

Iris Web框架本身使用这个包来测试自己。在[_examples存储库目录中](https://github.com/kataras/iris/tree/master/_examples)，您还可以找到一些有用的测试。有关更多信息，请查看并阅读 [httpexpect 的文档](https://github.com/gavv/httpexpect)。

## [本地化](https://www.iris-go.com/docs/#/?id=localization)

### [介绍](https://www.iris-go.com/docs/#/?id=introduction)

本地化功能提供了一种检索各种语言字符串的便捷方法，使您可以轻松地在应用程序中支持多种语言。语言字符串存储在目录中的文件中。在此目录中，应用程序支持的每种语言都应该有一个子目录：`./locales`

```bash
│   main.go
└───locales
    ├───el-GR
    │       home.yml
    ├───en-US
    │       home.yml
    └───zh-CN
            home.yml
```

应用程序的默认语言是第一个注册语言。

```go
app := iris.New()

// First parameter: Glob filpath patern,
// Second variadic parameter: Optional language tags,
// the first one is the default/fallback one.
app.I18n.Load("./locales/*/*", "en-US", "el-GR", "zh-CN")
```

或者，如果您通过以下方式加载所有语言：

```go
app.I18n.Load("./locales/*/*")
// Then set the default language using:
app.I18n.SetDefault("en-US")
```

### [加载嵌入式区域设置](https://www.iris-go.com/docs/#/?id=load-embedded-locales)

您可能希望在应用程序可执行文件中嵌入带有 go-bindata 工具的区域设置。

1. 安装一个go-bindata工具，例如

   `$ go get -u github.com/go-bindata/go-bindata/...`

2. 将本地文件嵌入到应用程序中

   `$ go-bindata -o locales.go ./locales/...`

3. 使用该方法初始化和加载语言`LoadAssets`

   ^ 和 函数由 生成`AssetNames``Asset``go-bindata`

```go
ap.I18n.LoadAssets(AssetNames, Asset, "en-US", "el-GR", "zh-CN")
```

### [定义翻译](https://www.iris-go.com/docs/#/?id=defining-translations)

区域设置文件可以写在YAML（推荐），JSON，TOML或INI表单上。

每个文件都应包含密钥。键也可以有子键（我们称它们为"部分"）。

每个键的值应是其翻译文本（或**模板**）或/及其复数键值的形式或包含的。`string``map`

Iris i18n 模块支持开箱即用的**复数化**，见下文。

### [Fmt Style](https://www.iris-go.com/docs/#/?id=fmt-style)

```yaml
hi: "Hi %s!"
ctx.Tr("Hi", "John")
// Outputs: Hi John!
```

### [模板](https://www.iris-go.com/docs/#/?id=template)

```yaml
hi: "Hi {{.Name}}!"
ctx.Tr("Hi", iris.Map{"Name": "John"})
// Outputs: Hi John!
```

### [多元化](https://www.iris-go.com/docs/#/?id=pluralization)

Iris i18n 支持复数变量。要定义每个区域设置变量，必须定义键的新部分。`Vars`

变量可接受的键是：

- `one`
- `"=x"`其中 x 是一个数字
- `"<x"`
- `other`
- `format`

例：

```yaml
Vars:
  - Minutes:
      one: "minute"
      other: "minutes"
  - Houses:
      one: "house"
      other: "houses"
```

然后，每条消息都可以使用此变量，方法如下：

```yaml
# Using variables in raw string
YouLate: "You are %[1]d ${Minutes} late."
# [x] is the argument position,
# variables always have priority other fmt-style arguments,
# that's why we see [1] for houses and [2] for the string argument.
HouseCount: "%[2]s has %[1]d ${Houses}."
ctx.Tr("YouLate", 1)
// Outputs: You are 1 minute late.
ctx.Tr("YouLate", 10)
// Outputs: You are 10 minutes late.

ctx.Tr("HouseCount", 2, "John")
// Outputs: John has 2 houses.
```

您可以根据给定的复数计数选择显示的消息。

除了变量，每条消息也可以有它的复数形式！

可接受的密钥：

- `zero`
- `one`
- `two`
- `"=x"`
- `"<x"`
- `">x"`
- `other`

让我们创建一个简单的复数功能消息，它也可以使用我们上面创建的Minutes变量。

```yaml
FreeDay:
  "=3": "You have three days and %[2]d ${Minutes} off." # "FreeDay" 3, 15
  one:  "You have a day off." # "FreeDay", 1
  other: "You have %[1]d free days." # "FreeDay", 5
ctx.Tr("FreeDay", 3, 15)
// Outputs: You have three days and 15 minutes off.
ctx.Tr("FreeDay", 1)
// Outputs: You have a day off.
ctx.Tr("FreeDay", 5)
// Outputs: You have 5 free days.
```

让我们继续使用一个更高级的示例，使用模板文本 + 函数 + 复数 + 变量。

```yaml
Vars:
  - Houses:
      one: "house"
      other: "houses"
  - Gender:
      "=1": "She"
      "=2": "He"

VarTemplatePlural:
  one: "${Gender} is awesome!"
  other: "other (${Gender}) has %[3]d ${Houses}."
  "=5": "{{call .InlineJoin .Names}} are awesome."
const (
    female = iota + 1
    male
)

ctx.Tr("VarTemplatePlural", iris.Map{
    "PluralCount": 5,
    "Names":       []string{"John", "Peter"},
    "InlineJoin": func(arr []string) string {
        return strings.Join(arr, ", ")
    },
})
// Outputs: John, Peter are awesome

ctx.Tr("VarTemplatePlural", 1, female)
// Outputs: She is awesome!

ctx.Tr("VarTemplatePlural", 2, female, 5)
// Outputs: other (She) has 5 houses.
```

### [部分](https://www.iris-go.com/docs/#/?id=sections)

如果密钥不是保留密钥（例如一个，两个...），则它充当子部分。这些部分由点字符 （） 分隔。`.`

```yaml
Welcome:
  Message: "Welcome {{.Name}}"
ctx.Tr("Welcome.Message", iris.Map{"Name": "John"})
// Outputs: Welcome John
```

### [确定当前区域设置](https://www.iris-go.com/docs/#/?id=determining-the-current-locale)

您可以使用该方法确定当前区域设置或检查区域设置是否为给定值：`context.GetLocale`

```go
func(ctx iris.Context) {
    locale := ctx.GetLocale()
    // [...]
}
```

**区域设置**界面如下所示。

```go
// Locale is the interface which returns from a `Localizer.GetLocale` metod.
// It serves the transltions based on "key" or format. See `GetMessage`.
type Locale interface {
    // Index returns the current locale index from the languages list.
    Index() int
    // Tag returns the full language Tag attached tothis Locale,
    // it should be uniue across different Locales.
    Tag() *language.Tag
    // Language should return the exact languagecode of this `Locale`
    //that the user provided on `New` function.
    //
    // Same as `Tag().String()` but it's static.
    Language() string
    // GetMessage should return translated text based n the given "key".
    GetMessage(key string, args ...interface{}) string
}
```

### [检索翻译](https://www.iris-go.com/docs/#/?id=retrieving-translation)

使用方法作为快捷方式来获取此请求的已翻译文本。`context.Tr`

```go
func(ctx iris.Context) {
    text := ctx.Tr("hi", "name")
    // [...]
}
```

### [内部视图](https://www.iris-go.com/docs/#/?id=inside-views)

```go
func(ctx iris.Context) {
    ctx.View("index.html", iris.Map{
        "tr": ctx.Tr,
    })
}
```

### [例](https://github.com/kataras/iris/tree/master/_examples/i18n)

```go
package main

import (
    "github.com/kataras/iris/v12"
)

func newApp() *iris.Application {
    app := iris.New()

    // Configure i18n.
    // First parameter: Glob filpath patern,
    // Second variadic parameter: Optional language tags, the first one is the default/fallback one.
    app.I18n.Load("./locales/*/*.ini", "en-US", "el-GR", "zh-CN")
    // app.I18n.LoadAssets for go-bindata.

    // Default values:
    // app.I18n.URLParameter = "lang"
    // app.I18n.Subdomain = true
    //
    // Set to false to disallow path (local) redirects,
    // see https://github.com/kataras/iris/issues/1369.
    // app.I18n.PathRedirect = true

    app.Get("/", func(ctx iris.Context) {
        hi := ctx.Tr("hi", "iris")

        locale := ctx.GetLocale()

        ctx.Writef("From the language %s translated output: %s", locale.Language(), hi)
    })

    app.Get("/some-path", func(ctx iris.Context) {
        ctx.Writef("%s", ctx.Tr("hi", "iris"))
    })

    app.Get("/other", func(ctx iris.Context) {
        language := ctx.GetLocale().Language()

        fromFirstFileValue := ctx.Tr("key1")
        fromSecondFileValue := ctx.Tr("key2")
        ctx.Writef("From the language: %s, translated output:\n%s=%s\n%s=%s",
            language, "key1", fromFirstFileValue,
            "key2", fromSecondFileValue)
    })

    // using in inside your views:
    view := iris.HTML("./views", ".html")
    app.RegisterView(view)

    app.Get("/templates", func(ctx iris.Context) {
        ctx.View("index.html", iris.Map{
            "tr": ctx.Tr, // word, arguments... {call .tr "hi" "iris"}}
        })

        // Note that,
        // Iris automatically adds a "tr" global template function as well,
        // the only difference is the way you call it inside your templates and
        // that it accepts a language code as its first argument.
    })
    //

    return app
}

func main() {
    app := newApp()

    // go to http://localhost:8080/el-gr/some-path
    // ^ (by path prefix)
    //
    // or http://el.mydomain.com8080/some-path
    // ^ (by subdomain - test locally with the hosts file)
    //
    // or http://localhost:8080/zh-CN/templates
    // ^ (by path prefix with uppercase)
    //
    // or http://localhost:8080/some-path?lang=el-GR
    // ^ (by url parameter)
    //
    // or http://localhost:8080 (default is en-US)
    // or http://localhost:8080/?lang=zh-CN
    //
    // go to http://localhost:8080/other?lang=el-GR
    // or http://localhost:8080/other (default is en-US)
    // or http://localhost:8080/other?lang=en-US
    //
    // or use cookies to set the language.
    app.Listen(":8080", iris.WithSitemap("http://localhost:8080"))
}
```

### [网站地图](https://www.iris-go.com/docs/#/?id=sitemap)

站点地图翻译会自动按路径前缀（如果为 true）或按子域（如果为 true）或按网址查询参数（如果为空）设置为每个路线。`app.I18n.PathRedirect``app.I18n.Subdomain``app.I18n.URLParameter`

阅读更多： https://support.google.com/webmasters/answer/189077?hl=en

```bash
GET http://localhost:8080/sitemap.xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">
    <url>
        <loc>http://localhost:8080/</loc>
        <xhtml:link rel="alternate" hreflang="en-US" href="http://localhost:8080/"></xhtml:link>
        <xhtml:link rel="alternate" hreflang="el-GR" href="http://localhost:8080/el-GR/"></xhtml:link>
        <xhtml:link rel="alternate" hreflang="zh-CN" href="http://localhost:8080/zh-CN/"></xhtml:link>
    </url>
    <url>
        <loc>http://localhost:8080/some-path</loc>
        <xhtml:link rel="alternate" hreflang="en-US" href="http://localhost:8080/some-path"></xhtml:link>
        <xhtml:link rel="alternate" hreflang="el-GR" href="http://localhost:8080/el-GR/some-path"></xhtml:link>
        <xhtml:link rel="alternate" hreflang="zh-CN" href="http://localhost:8080/zh-CN/some-path"></xhtml:link>
    </url>
    <url>
        <loc>http://localhost:8080/other</loc>
        <xhtml:link rel="alternate" hreflang="en-US" href="http://localhost:8080/other"></xhtml:link>
        <xhtml:link rel="alternate" hreflang="el-GR" href="http://localhost:8080/el-GR/other"></xhtml:link>
        <xhtml:link rel="alternate" hreflang="zh-CN" href="http://localhost:8080/zh-CN/other"></xhtml:link>
    </url>
    <url>
        <loc>http://localhost:8080/templates</loc>
        <xhtml:link rel="alternate" hreflang="en-US" href="http://localhost:8080/templates"></xhtml:link>
        <xhtml:link rel="alternate" hreflang="el-GR" href="http://localhost:8080/el-GR/templates"></xhtml:link>
        <xhtml:link rel="alternate" hreflang="zh-CN" href="http://localhost:8080/zh-CN/templates"></xhtml:link>
    </url>
</urlset>
```

这就是Iris的所有基础知识。本文档涵盖了足够的初学者。想要成为专家和认证虹膜开发人员，了解 MVC、i18n、依赖注入、gRPC、lambda 函数、websocket、最佳实践等？[立即索取虹膜电子书](https://www.iris-go.com/#ebookDonateForm)，参与鸢尾花的开发！