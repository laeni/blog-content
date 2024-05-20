---
title: 'OpenAPI 规范'
tags: OpenAPI, API, 接口, 规范
date: '2024-04-13'
updated: '2024-04-13'
hide: true
---

本文档为 OpenAPI 规范 3.1.0 版本的中文翻译版，原始文档参见[github](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md)。

本文档中，当“必须（MUST）”、“不得（MUST NOT）”、“需要（REQUIRED）”、“最好（SHALL）”、“最好不（SHALL NOT）”、“应该（SHOULD）”、“不应该（SHOULD）”、“推荐（RECOMMENDED）”、“不推荐（NOT RECOMMENDED）”、“可以（MAY）”和“可选（OPTIONAL）”这些关键词以全大写字母出现时，应按照[BCP 14](https://tools.ietf.org/html/bcp14)[RFC2119](https://tools.ietf.org/html/rfc2119)[RFC8174](https://tools.ietf.org/html/rfc8174) 中的说明进行解释。

本文档根据[Apache 许可证 2.0 版](https://www.apache.org/licenses/LICENSE-2.0.html)进行许可。

# 介绍

OpenAPI 规范（OAS）为 HTTP API 定义了一个与语言无关的标准接口，使得人和计算机都可以在不使用源代码、文档或监听网络通信的情况下具备发现和理解服务的功能。正确定义后，使用者可以使用最少的实现逻辑来理解远程服务并与之交互。

文档生成工具可以使用 OpenAPI 定义来显示 API，代码生成工具可以生成各种编程语言的服务器和客户端，测试工具以及许多其他工具也可以使用 OpenAPI 定义。

## 定义

## OpenAPI 文档

OpenAPI 文档是用于定义或描述 API 或 API 元素的自包含或复合资源。OpenAPI 文档必须至少包含一个[paths](#pathsObject#todo)字段、一个[components](#oasComponents#todo)字段或一个[webhooks](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasWebhooks)字段。OpenAPI 文档使用并符合 OpenAPI 规范。

## Path 模板

Path 模板是指使用大括号 ({}) 包围的模板表达式，用来在 URL 路径中标记一个可以被路径参数替换的部分。

路径中的每个模板表达式必须（MUST）对应一个路径参数，这些路径参数包含在每个[路径项](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#path-item-object)本身或路径项的每个[操作](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operation-object)中。一个例外是，如果路径项为空（例如由 ACL 约束），则不需要匹配的路径参数。

这些路径参数的值不得（MUST NOT）包含[RFC3986](https://tools.ietf.org/html/rfc3986#section-3)描述的任何未转义的“通用语法”字符：正斜杠（`/`）、问号（`?`）和井号（`#`）。 

## 媒体类型

媒体类型定义分布在多种资源中。媒体类型定义应该符合[RFC6838](https://tools.ietf.org/html/rfc6838) 。

可能的媒体类型定义的一些示例： 

```
text/plain; charset=utf-8
application/json
application/vnd.github+json
application/vnd.github.v3+json
application/vnd.github.v3.raw+json
application/vnd.github.v3.text+json
application/vnd.github.v3.html+json
application/vnd.github.v3.full+json
application/vnd.github.v3.diff
application/vnd.github.v3.patch
```

## HTTP 状态码

HTTP 状态代码用于指示已执行操作的状态。可用的状态代码由[RFC7231](https://tools.ietf.org/html/rfc7231#section-6)定义，已注册的状态代码在[IANA 状态代码注册表](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)列中。

# 规范

## 版本

OpenAPI 规范使用`major`.`minor`.`patch`版本控制方案。版本字符串的`major`.`minor`部分（例如`3.1`）应指定 OAS 功能集。*`.patch`*版本用于修订本文档中的错误或提供说明，而不是功能集。支持 OAS 3.1 的工具应该与所有 OAS 3.1.* 版本兼容。工具不应考虑补丁版本，例如对工具来说`3.1.0`和`3.1.1`没有任何区别。

有时，OAS 的`minor`版本可能不会向后兼容，这种情况通常认为相对于收益而言影响较小。

与 OAS 3.\*.\* 兼容的 OpenAPI 文档包含一个必需的[`openapi`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasVersion)字段，用于指定它使用的 OAS 版本。 

## 格式

符合 OpenAPI 规范的 OpenAPI 文档本身就是一个 JSON 对象，可以用 JSON 或 YAML 格式表示。

例如，如果一个字段有一个数组值，将使用 JSON 数组表示： 

```json
{
   "field":[1, 2, 3]
}
```

规范中的所有字段名称都**区分大小写**。这包括在Map中的所有键字段，除非明确指出键不**区分大小写**。

该模式公开了两种类型的字段：固定字段（字段名为明确声明）和模式化字段（字段名为正则表达式）。

模式字段在包含的对象中必须具有唯一的名称。

为了能在 YAML 和 JSON 格式之间转换，建议使用 YAML [1.2](https://yaml.org/spec/1.2/spec.html) 版，并且需要满足以下约束： 

- 标签必须限制在[JSON 模式规则集](https://yaml.org/spec/1.2/spec.html#id2803231)允许的范围内。
- YAML 中使用的键必须为[YAML Failsafe 模式规则集](https://yaml.org/spec/1.2/spec.html#id2802346)中定义的标量字符串。

**注意：**虽然 API 可以由 OpenAPI 文档以 YAML 或 JSON 格式定义，但 API 请求和响应正文以及其他内容不需要是 JSON 或 YAML。

---

https://wenku.baidu.com/view/e6de622fb868a98271fe910ef12d2af90242a819.html

### 文档结构

OpenAPI 文档可以由单个文档组成，也可以根据作者的判断分为多个相互关联的部分。在后一种情况下，[`Reference Objects`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)和[`Schema Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject)`$ref`使用关键字。

建议将根 OpenAPI 文档命名为：`openapi.json`或者`openapi.yaml`. 

### 

### 数据类型 

OAS 中的数据类型基于[JSON Schema Specification Draft 2020-12](https://tools.ietf.org/html/draft-bhutton-json-schema-00#section-4.2.1)支持的类型。注意`integer`作为一种类型也受支持，它被定义为不带分数或指数部分的 JSON 数字。模型是使用[Schema Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject)定义的，它是 JSON Schema Specification Draft 2020-12 的超集。

所定义的[正如JSON Schema Validation vocabulary](https://tools.ietf.org/html/draft-bhutton-json-schema-validation-00#section-7.3)，数据类型可以有一个可选的修饰符属性：`format`. OAS 定义了额外的格式来提供原始数据类型的详细信息。

OAS 定义的格式是： 

| [`type`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#dataTypes) | [`format`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#dataTypeFormat) | 评论                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------ |
| `integer`                                                    | `int32`                                                      | 带符号的 32 位           |
| `integer`                                                    | `int64`                                                      | 有符号的 64 位（又名长） |
| `number`                                                     | `float`                                                      |                          |
| `number`                                                     | `double`                                                     |                          |
| `string`                                                     | `password`                                                   | 提示 UI 模糊输入。       |

### 

### 富文本格式

在整个规范中`description`字段被标记为支持 CommonMark 降价格式。在 OpenAPI 工具呈现富文本的地方，它必须至少支持[CommonMark 0.27](https://spec.commonmark.org/0.27/)描述的降价语法。工具可以选择忽略一些 CommonMark 特性来解决安全问题。

### 

### URI 中的相对引用 

除非另有说明，所有作为 URI 的属性都可以是[RFC3986](https://tools.ietf.org/html/rfc3986#section-4.2)定义的相对引用。

相对引用，包括那些在[`Reference Objects`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)，[`PathItem Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject)`$ref`领域，[`Link Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#linkObject)`operationRef`领域和[`Example Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#exampleObject)`externalValue`字段，根据[RFC3986](https://tools.ietf.org/html/rfc3986#section-5.2)使用引用文档作为基本 URI 进行解析。

如果 URI 包含片段标识符，则应根据引用文档的片段解析机制解析该片段。解释为 JSON 指针[如果引用文档的表示是 JSON 或 YAML，则片段标识符应该根据RFC6901](https://tools.ietf.org/html/rfc6901)。

中的相对引用[`Schema Objects`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject)，包括任何显示为`$id`值，使用最近的父母`$id`所述[作为 Base URI，如JSON Schema Specification Draft 2020-12](https://tools.ietf.org/html/draft-bhutton-json-schema-00#section-8.2)。如果没有父模式包含`$id`, 那么 Base URI 必须根据[RFC3986](https://tools.ietf.org/html/rfc3986#section-5.1)来确定。

### 

### URL 中的相对引用 

除非另有说明，所有作为 URL 的属性都可以是[RFC3986](https://tools.ietf.org/html/rfc3986#section-4.2)定义的相对引用。除非另有说明，否则相对引用使用定义在[`Server Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverObject)作为基本 URL。请注意，这些本身可能与引用文档有关。

### 

### 图式 

在下面的描述中，如果一个字段没有明确**要求**或用 MUST 或 SHALL 描述，则可以认为它是 OPTIONAL。

#### 

#### OpenAPI 对象 

的根对象[这是OpenAPI 文档](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasDocument)。

##### 

##### 固定字段 

| 字段名称       | 类型                                                         | 描述                                                         |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 开火           | `string`                                                     | **必需的**。此字符串必须是[版本号](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#versions)OpenAPI 文档使用的 OpenAPI 规范的 。这`openapi`工具应该使用该字段来解释 OpenAPI 文档。这 *无关* 与 API[`info.version`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#infoVersion)细绳。 |
| 信息           | [信息对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#infoObject) | **必需的**。提供有关 API 的元数据。元数据可以根据需要由工具使用。 |
| jsonSchema方言 | `string`                                                     | 的默认值`$schema`中的关键字[模式对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject)此 OAS 文档中包含的 。这必须采用 URI 的形式。 |
| 服务器         | [[服务器对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverObject)] | 一组服务器对象，它提供到目标服务器的连接信息。如果`servers`未提供属性，或者是一个空数组，默认值将是一个[服务器对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverObject)，其[url](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverUrl)值为`/`. |
| 路径           | [路径对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathsObject) | API 的可用路径和操作。                                       |
| 网络钩子       | Map[`string`,[路径项对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)]] | 传入的 webhooks 可以作为此 API 的一部分接收，并且 API 消费者可以选择实施。密切相关的`callbacks`功能，本节描述由 API 调用以外的方式发起的请求，例如由带外注册发起的请求。键名是引用每个 webhook 的唯一字符串，而（可选引用的）路径项对象描述了可能由 API 提供者发起的请求和预期的响应。一个[例子。](https://github.com/OAI/OpenAPI-Specification/blob/main/examples/v3.1/webhook-example.yaml)  有 |
| 成分           | [组件对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#componentsObject) | 用于保存文档的各种模式的元素。                               |
| 安全           | [[安全需求对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#securityRequirementObject)] | 可以在 API 中使用哪些安全机制的声明。值列表包括可以使用的替代安全要求对象。只需满足其中一个安全要求对象即可授权请求。个别操作可以覆盖此定义。为了使安全性成为可选的，一个空的安全要求（`{}`）可以包含在数组中。 |
| 标签           | [[标记对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#tagObject)] | 带有附加元数据的文档使用的标签列表。标签的顺序可用于通过解析工具反映它们的顺序。使用的所有标签都[并非操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject)必须声明。未声明的标签可以随机组织或根据工具的逻辑组织。列表中的每个标签名称必须是唯一的。 |
| 外部文档       | [外部文档对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#externalDocumentationObject) | 额外的外部文档。                                             |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

#### 

#### 信息对象 

该对象提供有关 API 的元数据。如果需要，元数据可以由客户使用，并且可以为了方便而在编辑或文档生成工具中呈现。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 标题     | `string`                                                     | **必需的**。API 的标题。                                     |
| 概括     | `string`                                                     | API 的简短摘要。                                             |
| 描述     | `string`                                                     | API 的描述。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 服务条款 | `string`                                                     | API 服务条款的 URL。这必须是 URL 的形式。                    |
| 接触     | [联系对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#contactObject) | 公开的 API 的联系信息。                                      |
| 执照     | [许可证对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#licenseObject) | 公开的 API 的许可证信息。                                    |
| 版本     | `string`                                                     | **必需的**。OpenAPI 文档的版本（不同于[OpenAPI 规范版本](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasVersion)或 API 实现版本）。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 信息对象示例 

```
{
  "title": "Sample Pet Store App",
  "summary": "A pet store manager.",
  "description": "This is a sample server for a pet store.",
  "termsOfService": "https://example.com/terms/",
  "contact": {
    "name": "API Support",
    "url": "https://www.example.com/support",
    "email": "support@example.com"
  },
  "license": {
    "name": "Apache 2.0",
    "url": "https://www.apache.org/licenses/LICENSE-2.0.html"
  },
  "version": "1.0.1"
}
title: Sample Pet Store App
summary: A pet store manager.
description: This is a sample server for a pet store.
termsOfService: https://example.com/terms/
contact:
  name: API Support
  url: https://www.example.com/support
  email: support@example.com
license:
  name: Apache 2.0
  url: https://www.apache.org/licenses/LICENSE-2.0.html
version: 1.0.1
```

#### 

#### 联系对象 

公开的 API 的联系信息。

##### 

##### 固定字段 

| 字段名称 | 类型     | 描述                                                    |
| -------- | -------- | ------------------------------------------------------- |
| 姓名     | `string` | 联系人/组织的识别名称。                                 |
| 网址     | `string` | 指向联系信息的 URL。这必须是 URL 的形式。               |
| 电子邮件 | `string` | 联系人/组织的电子邮件地址。这必须是电子邮件地址的形式。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 联系人对象示例 

```
{
  "name": "API Support",
  "url": "https://www.example.com/support",
  "email": "support@example.com"
}
name: API Support
url: https://www.example.com/support
email: support@example.com
```

#### 

#### 许可证对象 

公开的 API 的许可证信息。

##### 

##### 固定字段 

| 字段名称 | 类型     | 描述                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| 姓名     | `string` | **必需的**。用于 API 的许可证名称。                          |
| 标识符   | `string` | 许可证表达式[SPDX](https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60)API 的 。这`identifier`字段是互斥的`url`场地。 |
| 网址     | `string` | 用于 API 的许可证的 URL。这必须是 URL 的形式。这`url`字段是互斥的`identifier`场地。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 许可证对象示例 

```
{
  "name": "Apache 2.0",
  "identifier": "Apache-2.0"
}
name: Apache 2.0
identifier: Apache-2.0
```

#### 

#### 服务器对象 

表示服务器的对象。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 网址     | `string`                                                     | **必需的**。目标主机的 URL。此 URL 支持服务器变量并且可以是相对的，以指示主机位置与提供 OpenAPI 文档的位置相关。当一个变量被命名为时，将进行变量替换`{`括号`}`. |
| 描述     | `string`                                                     | 可选字符串，描述 URL 指定的主机。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 变量     | Map[`string`,[服务器变量对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverVariableObject)] | 变量名称与其值之间的映射。该值用于替换服务器的 URL 模板。    |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 服务器对象示例 

单个服务器将被描述为： 

```
{
  "url": "https://development.gigantic-server.com/v1",
  "description": "Development server"
}
url: https://development.gigantic-server.com/v1
description: Development server
```

下面显示了如何描述多个服务器，例如，在 OpenAPI 对象的[`servers`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasServers): 

```
{
  "servers":[
    {
      "url": "https://development.gigantic-server.com/v1",
      "description": "Development server"
    },
    {
      "url": "https://staging.gigantic-server.com/v1",
      "description": "Staging server"
    },
    {
      "url": "https://api.gigantic-server.com/v1",
      "description": "Production server"
    }
]
}
servers:
- url: https://development.gigantic-server.com/v1
  description: Development server
- url: https://staging.gigantic-server.com/v1
  description: Staging server
- url: https://api.gigantic-server.com/v1
  description: Production server
```

下面显示了如何将变量用于服务器配置： 

```
{
  "servers":[
    {
      "url": "https://{username}.gigantic-server.com:{port}/{basePath}",
      "description": "The production API server",
      "variables": {
        "username": {
          "default": "demo",
          "description": "this value is assigned by the service provider, in this example`gigantic-server.com`"
        },
        "port": {
          "enum":[
            "8443",
            "443"
        ],
          "default": "8443"
        },
        "basePath": {
          "default": "v2"
        }
      }
    }
]
}
servers:
- url: https://{username}.gigantic-server.com:{port}/{basePath}
  description: The production API server
  variables:
    username:
      # note! no enum here means it is an open value
      default: demo
      description: this value is assigned by the service provider, in this example`gigantic-server.com`
    port:
      enum:
        - '8443'
        - '443'
      default: '8443'
    basePath:
      # open meaning there is the opportunity to use special base paths as assigned by the provider, default is`v2`
      default: v2
```

#### 服务器变量对象 

表示用于服务器 URL 模板替换的服务器变量的对象。

##### 

##### 固定字段 

| 字段名称 | 类型       | 描述                                                         |
| -------- | ---------- | ------------------------------------------------------------ |
| 枚举     | [`string`] | 如果替换选项来自有限集合，则要使用的字符串值的枚举。该数组不能为空。 |
| 默认     | `string`   | **必需的**。用于替换的默认值，如果 *未* 提供替代值，则应发送该默认值。请注意，此行为不同于[模式对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject)对默认值的处理，因为在那些情况下参数值是可选的。如果[`enum`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverVariableEnum)已定义，该值必须存在于枚举值中。 |
| 描述     | `string`   | 服务器变量的可选描述。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

#### 

#### 组件对象 

为 OAS 的不同方面保存一组可重用对象。components 对象中定义的所有对象对 API 都没有影响，除非从 components 对象外部的属性中明确引用它们。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 图式     | Map[`string`,[模式对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject)] | 一个对象来保存可重用的[Schema Objects](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject) 。 |
| 反应     | Map[`string`,[响应对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#responseObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[Response Objects](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#responseObject) 。 |
| 参数     | Map[`string`,[参数对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[参数对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterObject)。 |
| 例子     | Map[`string`,[示例对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#exampleObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[示例对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#exampleObject)。 |
| 请求体   | Map[`string`,[请求正文对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#requestBodyObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[Request Body Objects](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#requestBodyObject) 。 |
| 标题     | Map[`string`,[标题对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#headerObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[Header Objects](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#headerObject) 。 |
| 安全计划 | Map[`string`,[安全方案对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#securitySchemeObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[安全方案对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#securitySchemeObject)。 |
| 链接     | Map[`string`,[链接对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#linkObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[链接对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#linkObject)。 |
| 回调     | Map[`string`,[回调对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#callbackObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[回调对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#callbackObject)。 |
| 路径项   | Map[`string`,[路径项对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 一个对象来保存可重用的[Path Item Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject) 。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

上面声明的所有固定字段都是必须使用与正则表达式匹配的键的对象：`^[a-zA-Z0-9\.\-_]+$`. 

字段名称示例： 

```
User
User_1
User_Name
user-name
my.org.User
```

##### 

##### 组件对象示例 

```
"components": {
  "schemas": {
    "GeneralError": {
      "type": "object",
      "properties": {
        "code": {
          "type": "integer",
          "format": "int32"
        },
        "message": {
          "type": "string"
        }
      }
    },
    "Category": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64"
        },
        "name": {
          "type": "string"
        }
      }
    },
    "Tag": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64"
        },
        "name": {
          "type": "string"
        }
      }
    }
  },
  "parameters": {
    "skipParam": {
      "name": "skip",
      "in": "query",
      "description": "number of items to skip",
      "required": true,
      "schema": {
        "type": "integer",
        "format": "int32"
      }
    },
    "limitParam": {
      "name": "limit",
      "in": "query",
      "description": "max records to return",
      "required": true,
      "schema" : {
        "type": "integer",
        "format": "int32"
      }
    }
  },
  "responses": {
    "NotFound": {
      "description": "Entity not found."
    },
    "IllegalInput": {
      "description": "Illegal input for operation."
    },
    "GeneralError": {
      "description": "General Error",
      "content": {
        "application/json": {
          "schema": {
            "$ref": "#/components/schemas/GeneralError"
          }
        }
      }
    }
  },
  "securitySchemes": {
    "api_key": {
      "type": "apiKey",
      "name": "api_key",
      "in": "header"
    },
    "petstore_auth": {
      "type": "oauth2",
      "flows": {
        "implicit": {
          "authorizationUrl": "https://example.org/api/oauth/dialog",
          "scopes": {
            "write:pets": "modify pets in your account",
            "read:pets": "read your pets"
          }
        }
      }
    }
  }
}
components:
  schemas:
    GeneralError:
      type: object
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
    Category:
      type: object
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
    Tag:
      type: object
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
  parameters:
    skipParam:
      name: skip
      in: query
      description: number of items to skip
      required: true
      schema:
        type: integer
        format: int32
    limitParam:
      name: limit
      in: query
      description: max records to return
      required: true
      schema:
        type: integer
        format: int32
  responses:
    NotFound:
      description: Entity not found.
    IllegalInput:
      description: Illegal input for operation.
    GeneralError:
      description: General Error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/GeneralError'
  securitySchemes:
    api_key:
      type: apiKey
      name: api_key
      in: header
    petstore_auth:
      type: oauth2
      flows: 
        implicit:
          authorizationUrl: https://example.org/api/oauth/dialog
          scopes:
            write:pets: modify pets in your account
            read:pets: read your pets
```

#### 

#### 路径对象 

保存各个端点及其操作的相对路径。路径附加到来自[`Server Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverObject)为了构建完整的 URL。由于[访问控制列表 (ACL) 约束](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#securityFiltering)，路径可能为空。

##### 

##### 图案字段 

| 字段模式 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| /{小路}  | [路径项对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject) | 单个端点的相对路径。字段名称必须以正斜杠 （`/`）.   路径**附加**（无相对 URL 解析）到来自[`Server Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverObject)的`url`字段以构建完整的 URL。[Path 模板](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathTemplating)  允许 。匹配 URL 时，具体的（非模板化的）路径将在它们的模板化路径之前被匹配。具有相同层次结构但不同模板名称的模板路径不得存在，因为它们是相同的。在不明确匹配的情况下，由工具决定使用哪一个。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### Path 模板匹配 

假设路径如下，具体定义，`/pets/mine`, 如果使用，将首先匹配： 

```
  /pets/{petId}
  /pets/mine
```

以下路径被视为相同且无效： 

```
  /pets/{petId}
  /pets/{name}
```

以下可能会导致不明确的解决方案： 

```
  /{entity}/me
  /books/{id}
```

##### 

##### 路径对象示例 

```
{
  "/pets": {
    "get": {
      "description": "Returns all pets from the system that the user has access to",
      "responses": {
        "200": {          
          "description": "A list of pets.",
          "content": {
            "application/json": {
              "schema": {
                "type": "array",
                "items": {
                  "$ref": "#/components/schemas/pet"
                }
              }
            }
          }
        }
      }
    }
  }
}
/pets:
  get:
    description: Returns all pets from the system that the user has access to
    responses:
      '200':
        description: A list of pets.
        content:
          application/json:
            schema:
              type: array
              items:
                $ref: '#/components/schemas/pet'
```

#### 

#### 路径项对象 

描述单个路径上可用的操作。由于[ACL 约束](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#securityFiltering)，路径项可以为空。路径本身仍然暴露给文档查看者，但他们不知道哪些操作和参数可用。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| $ref     | `string`                                                     | 允许引用此路径项的定义。引用的结构必须采用[Path Item Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject)的形式。如果 Path Item Object 字段同时出现在定义的对象和引用的对象中，则行为未定义。的规则[请参阅解析Relative References](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#relativeReferencesURI)。 |
| 概括     | `string`                                                     | 一个可选的字符串摘要，旨在应用于此路径中的所有操作。         |
| 描述     | `string`                                                     | 一个可选的字符串描述，旨在应用于此路径中的所有操作。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 得到     | [操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject) | 此路径上的 GET 操作的定义。                                  |
| 放       | [操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject) | 此路径上 PUT 操作的定义。                                    |
| 邮政     | [操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject) | 此路径上的 POST 操作的定义。                                 |
| 删除     | [操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject) | 此路径上的 DELETE 操作的定义。                               |
| 选项     | [操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject) | 此路径上的 OPTIONS 操作的定义。                              |
| 头       | [操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject) | 此路径上的 HEAD 操作的定义。                                 |
| 修补     | [操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject) | 此路径上 PATCH 操作的定义。                                  |
| 痕迹     | [操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject) | 此路径上的 TRACE 操作的定义。                                |
| 服务器   | [[服务器对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverObject)] | 替代`server`数组以服务此路径中的所有操作。                   |
| 参数     | [[参数对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 适用于此路径下描述的所有操作的参数列表。这些参数可以在操作级别覆盖，但不能在那里删除。该列表不得包含重复的参数。的组合定义[唯一参数由名称](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterName)和[位置](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterIn)。该列表可以使用[引用对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)链接到在[OpenAPI 对象的 components/parameters](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#componentsParameters)中定义的参数。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 路径项对象示例 

```
{
  "get": {
    "description": "Returns pets based on ID",
    "summary": "Find pets by ID",
    "operationId": "getPetsById",
    "responses": {
      "200": {
        "description": "pet response",
        "content": {
          "*/*": {
            "schema": {
              "type": "array",
              "items": {
                "$ref": "#/components/schemas/Pet"
              }
            }
          }
        }
      },
      "default": {
        "description": "error payload",
        "content": {
          "text/html": {
            "schema": {
              "$ref": "#/components/schemas/ErrorModel"
            }
          }
        }
      }
    }
  },
  "parameters":[
    {
      "name": "id",
      "in": "path",
      "description": "ID of pet to use",
      "required": true,
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "style": "simple"
    }
]
}
get:
  description: Returns pets based on ID
  summary: Find pets by ID
  operationId: getPetsById
  responses:
    '200':
      description: pet response
      content:
        '*/*' :
          schema:
            type: array
            items:
              $ref: '#/components/schemas/Pet'
    default:
      description: error payload
      content:
        'text/html':
          schema:
            $ref: '#/components/schemas/ErrorModel'
parameters:
- name: id
  in: path
  description: ID of pet to use
  required: true
  schema:
    type: array
    items:
      type: string  
  style: simple
```

#### 

#### 操作对象 

描述路径上的单个 API 操作。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 标签     | [`string`]                                                   | API 文档控制的标签列表。标签可用于按资源或任何其他限定符对操作进行逻辑分组。 |
| 概括     | `string`                                                     | 操作的简短摘要。                                             |
| 描述     | `string`                                                     | 对操作行为的详细解释。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 外部文档 | [外部文档对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#externalDocumentationObject) | 此操作的附加外部文档。                                       |
| 操作编号 | `string`                                                     | 用于标识操作的唯一字符串。在 API 中描述的所有操作中，id 必须是唯一的。operationId 值**区分大小写**。工具和库可以使用 operationId 来唯一标识一个操作，因此，建议遵循通用的编程命名约定。 |
| 参数     | [[参数对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 适用于此操作的参数列表。中定义了参数[如果已在Path Item](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemParameters)，则新定义将覆盖它但永远不能删除它。该列表不得包含重复的参数。的组合定义[唯一参数由名称](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterName)和[位置](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterIn)。该列表可以使用[引用对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)链接到在[OpenAPI 对象的 components/parameters](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#componentsParameters)中定义的参数。 |
| 请求体   | [请求正文对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#requestBodyObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject) | 适用于此操作的请求正文。这`requestBody`在 HTTP 方法中完全受支持，其中 HTTP 1.1 规范[RFC7231](https://tools.ietf.org/html/rfc7231#section-4.3.1)明确定义了请求主体的语义。在 HTTP 规范模糊的其他情况下（例如[GET](https://tools.ietf.org/html/rfc7231#section-4.3.1) 、[HEAD](https://tools.ietf.org/html/rfc7231#section-4.3.2)和[DELETE](https://tools.ietf.org/html/rfc7231#section-4.3.5) ），`requestBody`允许但没有明确定义的语义，如果可能应该避免。 |
| 回应     | [响应对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#responsesObject) | 执行此操作返回的可能响应列表。                               |
| 回调     | Map[`string`,[回调对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#callbackObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 与父操作相关的可能带外回调的映射。键是回调对象的唯一标识符。映射中的每个值都是一个[回调对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#callbackObject)，它描述了可能由 API 提供者发起的请求和预期的响应。 |
| 弃用     | `boolean`                                                    | 声明此操作已弃用。消费者应该避免使用声明的操作。默认值为`false`. |
| 安全     | [[安全需求对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#securityRequirementObject)] | 该操作可使用哪些安全机制的声明。值列表包括可以使用的替代安全要求对象。只需满足其中一个安全要求对象即可授权请求。为了使安全性成为可选的，一个空的安全要求（`{}`）可以包含在数组中。此定义覆盖任何已声明的顶级[`security`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasSecurity).  要删除顶级安全声明，可以使用空数组。 |
| 服务器   | [[服务器对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverObject)] | 替代`server`数组来服务这个操作。如果有替代方案`server`object 在 Path Item Object 或 Root 级别指定，它将被此值覆盖。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 操作对象示例 

```
{
  "tags":[
    "pet"
],
  "summary": "Updates a pet in the store with form data",
  "operationId": "updatePetWithForm",
  "parameters":[
    {
      "name": "petId",
      "in": "path",
      "description": "ID of pet that needs to be updated",
      "required": true,
      "schema": {
        "type": "string"
      }
    }
],
  "requestBody": {
    "content": {
      "application/x-www-form-urlencoded": {
        "schema": {
          "type": "object",
          "properties": {
            "name": { 
              "description": "Updated name of the pet",
              "type": "string"
            },
            "status": {
              "description": "Updated status of the pet",
              "type": "string"
            }
          },
          "required":["status"] 
        }
      }
    }
  },
  "responses": {
    "200": {
      "description": "Pet updated.",
      "content": {
        "application/json": {},
        "application/xml": {}
      }
    },
    "405": {
      "description": "Method Not Allowed",
      "content": {
        "application/json": {},
        "application/xml": {}
      }
    }
  },
  "security":[
    {
      "petstore_auth":[
        "write:pets",
        "read:pets"
    ]
    }
]
}
tags:
- pet
summary: Updates a pet in the store with form data
operationId: updatePetWithForm
parameters:
- name: petId
  in: path
  description: ID of pet that needs to be updated
  required: true
  schema:
    type: string
requestBody:
  content:
    'application/x-www-form-urlencoded':
      schema:
       type: object
       properties:
          name: 
            description: Updated name of the pet
            type: string
          status:
            description: Updated status of the pet
            type: string
       required:
         - status
responses:
  '200':
    description: Pet updated.
    content: 
      'application/json': {}
      'application/xml': {}
  '405':
    description: Method Not Allowed
    content: 
      'application/json': {}
      'application/xml': {}
security:
- petstore_auth:
  - write:pets
  - read:pets
```

#### 

#### 外部文档对象 

允许引用外部资源以获取扩展文档。

##### 

##### 固定字段 

| 字段名称 | 类型     | 描述                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| 描述     | `string` | 目标文档的描述。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 网址     | `string` | **必需的**。目标文档的 URL。这必须是 URL 的形式。            |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 外部文档对象示例 

```
{
  "description": "Find more info here",
  "url": "https://example.com"
}
description: Find more info here
url: https://example.com
```

####

#### 参数对象 

描述单个操作参数。

的组合定义[唯一参数由名称](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterName)和[位置](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterIn)。

##### 

##### 参数位置 

有四个可能的参数位置由`in`场地： 

- path - 与[Path Templating](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathTemplating)一起使用，其中参数值实际上是操作 URL 的一部分。这不包括 API 的主机或基本路径。例如，在`/items/{itemId}`，路径参数为`itemId`. 
- query - 附加到 URL 的参数。例如，在`/items?id=###`，查询参数为`id`. 
- header - 预期作为请求一部分的自定义标头。请注意，[RFC7230](https://tools.ietf.org/html/rfc7230#page-22)声明标头名称不区分大小写。
- cookie - 用于将特定的 cookie 值传递给 API。

##### 

##### 固定字段 

| 字段名称 | 类型      | 描述                                                         |
| -------- | --------- | ------------------------------------------------------------ |
| 姓名     | `string`  | **必需的**。参数的名称。参数名称 *区分大小写* 。如果[`in`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterIn)是`"path"`，这`name`字段必须对应于[中路径](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathsPath)字段[路径对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathsObject)中出现的模板表达式。请参阅[Path 模板。](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathTemplating)  有关详细信息，如果[`in`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterIn)是`"header"`和`name`领域是`"Accept"`,`"Content-Type"`或者`"Authorization"`，参数定义应被忽略。对于所有其他情况，`name`对应于所使用的参数名称[`in`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterIn)财产。 |
| 在       | `string`  | **必需的**。参数的位置。可能的值是`"query"`,`"header"`,`"path"`或者`"cookie"`. |
| 描述     | `string`  | 参数的简要说明。这可能包含使用示例。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 必需的   | `boolean` | 确定此参数是否是必需的。如果[参数位置](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterIn)是`"path"`，此属性是**必需的**，它的值必须是`true`.  否则，可以包含该属性，其默认值为`false`. |
| 弃用     | `boolean` | 指定参数已弃用并且应该停止使用。默认值为`false`.             |
| 允许空值 | `boolean` | 设置传递空值参数的能力。这仅适用于`query`参数并允许发送具有空值的参数。默认值为`false`.  如果[`style`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterStyle)被使用，如果行为是`n/a`（不能序列化），值`allowEmptyValue`应被忽略。不推荐使用此属性，因为它可能会在以后的修订中被删除。 |

参数的序列化规则以两种方式之一指定。对于更简单的场景，[`schema`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterSchema)和[`style`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterStyle)可以描述参数的结构和语法。

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 风格     | `string`                                                     | 描述如何根据参数值的类型对参数值进行序列化。默认值（基于值`in`）： 为了`query` -`form`;  为了`path` -`simple`;  为了`header` -`simple`;  为了`cookie` -`form`. |
| 爆炸     | `boolean`                                                    | 当这是真的时，类型的参数值`array`或者`object`为数组的每个值或映射的键值对生成单独的参数。对于其他类型的参数，此属性无效。什么时候[`style`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterStyle)是`form`，默认值为`true`.  对于所有其他样式，默认值为`false`. |
| 允许保留 | `boolean`                                                    | 确定参数值是否应该允许保留字符，如[RFC3986所定义](https://tools.ietf.org/html/rfc3986#section-2.2)`:/?#[]@!$&'()*+,;=`无需百分比编码即可包含在内。此属性仅适用于具有`in`的价值`query`.  默认值为`false`. |
| 图式     | [架构对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject) | 定义用于参数的类型的模式。                                   |
| 例子     | 任何                                                         | 参数潜在值的示例。该示例应该匹配指定的架构和编码属性（如果存在）。这`example`字段是互斥的`examples`场地。此外，如果引用一个`schema`包含一个例子，`example`值应 *覆盖* 模式提供的示例。为了表示不能自然地用 JSON 或 YAML 表示的媒体类型的示例，字符串值可以包含示例，并在必要时进行转义。 |
| 例子     | Map[`string`,[示例对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#exampleObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 参数潜在值的示例。每个示例都应该包含参数编码中指定的正确格式的值。这`examples`字段是互斥的`example`场地。此外，如果引用一个`schema`包含一个例子，`examples`值应 *覆盖* 模式提供的示例。 |

对于更复杂的场景，[`content`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterContent)属性可以定义参数的媒体类型和架构。参数必须包含`schema`财产，或`content`财产，但不是两者。什么时候`example`或者`examples`与提供一起`schema`对象，示例必须遵循参数的规定序列化策略。

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 内容     | Map[`string`,[媒体类型对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#mediaTypeObject)] | 包含参数表示的映射。键是媒体类型，值描述它。Map必须只包含一个条目。 |

##### 

##### 样式值 

为了支持序列化简单参数的常用方法，一组`style`值被定义。

| `style`  | [`type`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#dataTypes) | `in`             | 评论                                                         |
| -------- | ------------------------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| 矩阵     | `primitive`,`array`,`object`                                 | `path`           | 定义的路径样式参数[RFC6570](https://tools.ietf.org/html/rfc6570#section-3.2.7) |
| 标签     | `primitive`,`array`,`object`                                 | `path`           | 定义的标签样式参数[RFC6570](https://tools.ietf.org/html/rfc6570#section-3.2.5) |
| 形式     | `primitive`,`array`,`object`                                 | `query`,`cookie` | 定义的表单样式参数[RFC6570](https://tools.ietf.org/html/rfc6570#section-3.2.8)。这个选项取代`collectionFormat`与`csv`（什么时候`explode`是错误的）或`multi`（什么时候`explode`是真实的）来自 OpenAPI 2.0 的值。 |
| 简单的   | `array`                                                      | `path`,`header`  | 定义的简单样式参数[RFC6570](https://tools.ietf.org/html/rfc6570#section-3.2.2)。这个选项取代`collectionFormat`与`csv`来自 OpenAPI 2.0 的值。 |
| 空格分隔 | `array`,`object`                                             | `query`          | 空格分隔的数组或对象值。这个选项取代`collectionFormat`等于`ssv`来自 OpenAPI 2.0。 |
| 管道分隔 | `array`,`object`                                             | `query`          | 管道分隔的数组或对象值。这个选项取代`collectionFormat`等于`pipes`来自 OpenAPI 2.0。 |
| 深度对象 | `object`                                                     | `query`          | 提供一种使用表单参数呈现嵌套对象的简单方法。                 |

##### 

##### 样式示例 

假设一个名为`color`具有以下值之一： 

```
   string -> "blue"
   array ->["blue","black","brown"]
   object -> { "R": 100, "G": 200, "B": 150 }
```

下表显示了每个值的渲染差异示例。

| [`style`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#styleValues) | `explode` | `empty` | `string`   | `array`                        | `object`                            |
| ------------------------------------------------------------ | --------- | ------- | ---------- | ------------------------------ | ----------------------------------- |
| 矩阵                                                         | 错误的    | ;颜色   | ;颜色=蓝色 | ;颜色=蓝色、黑色、棕色         | ;颜色=R,100,G,200,B,150             |
| 矩阵                                                         | 真的      | ;颜色   | ;颜色=蓝色 | ;颜色=蓝色;颜色=黑色;颜色=棕色 | ;R=100;G=200;B=150                  |
| 标签                                                         | 错误的    | .       | 。蓝色的   | .blue.black.brown              | .R.100.G.200.B.150                  |
| 标签                                                         | 真的      | .       | 。蓝色的   | .blue.black.brown              | .R=100.G=200.B=150                  |
| 形式                                                         | 错误的    | 颜色=   | 颜色=蓝色  | 颜色=蓝色、黑色、棕色          | 颜色=R,100,G,200,B,150              |
| 形式                                                         | 真的      | 颜色=   | 颜色=蓝色  | 颜色=蓝色&颜色=黑色&颜色=棕色  | R=100&G=200&B=150                   |
| 简单的                                                       | 错误的    | 不适用  | 蓝色的     | 蓝色，黑色，棕色               | R,100,G,200,B,150                   |
| 简单的                                                       | 真的      | 不适用  | 蓝色的     | 蓝色，黑色，棕色               | R=100,G=200,B=150                   |
| 空格分隔                                                     | 错误的    | 不适用  | 不适用     | 蓝色%20黑色%20棕色             | R%20100%20G%20200%20B%20150         |
| 管道分隔                                                     | 错误的    | 不适用  | 不适用     | 蓝色\|黑色\|棕色               | R\|100\|G\|200\|B\|150              |
| 深度对象                                                     | 真的      | 不适用  | 不适用     | 不适用                         | 颜色[R]=100&颜色[G]=200&颜色[B]=150 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 参数对象示例 

带有 64 位整数数组的标头参数： 

```
{
  "name": "token",
  "in": "header",
  "description": "token to be passed as a header",
  "required": true,
  "schema": {
    "type": "array",
    "items": {
      "type": "integer",
      "format": "int64"
    }
  },
  "style": "simple"
}
name: token
in: header
description: token to be passed as a header
required: true
schema:
  type: array
  items:
    type: integer
    format: int64
style: simple
```

字符串值的路径参数： 

```
{
  "name": "username",
  "in": "path",
  "description": "username to fetch",
  "required": true,
  "schema": {
    "type": "string"
  }
}
name: username
in: path
description: username to fetch
required: true
schema:
  type: string
```

字符串值的可选查询参数，通过重复查询参数允许多个值： 

```
{
  "name": "id",
  "in": "query",
  "description": "ID of the object to fetch",
  "required": false,
  "schema": {
    "type": "array",
    "items": {
      "type": "string"
    }
  },
  "style": "form",
  "explode": true
}
name: id
in: query
description: ID of the object to fetch
required: false
schema:
  type: array
  items:
    type: string
style: form
explode: true
```

一个自由形式的查询参数，允许特定类型的未定义参数： 

```
{
  "in": "query",
  "name": "freeForm",
  "schema": {
    "type": "object",
    "additionalProperties": {
      "type": "integer"
    },
  },
  "style": "form"
}
in: query
name: freeForm
schema:
  type: object
  additionalProperties:
    type: integer
style: form
```

一个复杂的参数使用`content`定义序列化： 

```
{
  "in": "query",
  "name": "coordinates",
  "content": {
    "application/json": {
      "schema": {
        "type": "object",
        "required":[
          "lat",
          "long"
      ],
        "properties": {
          "lat": {
            "type": "number"
          },
          "long": {
            "type": "number"
          }
        }
      }
    }
  }
}
in: query
name: coordinates
content:
  application/json:
    schema:
      type: object
      required:
        - lat
        - long
      properties:
        lat:
          type: number
        long:
          type: number
```

#### 

#### 请求正文对象 

描述单个请求正文。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 描述     | `string`                                                     | 请求正文的简要说明。这可能包含使用示例。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 内容     | Map[`string`,[媒体类型对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#mediaTypeObject)] | **必需的**。请求正文的内容。键是媒体类型或[媒体类型范围](https://tools.ietf.org/html/rfc7231#appendix-D)，值描述它。对于匹配多个键的请求，只有最具体的键适用。例如 text/plain 覆盖 text/* |
| 必需的   | `boolean`                                                    | 确定请求中是否需要请求正文。默认为`false`.                   |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 请求正文示例 

具有引用模型定义的请求正文。

```
{
  "description": "user to add to the system",
  "content": {
    "application/json": {
      "schema": {
        "$ref": "#/components/schemas/User"
      },
      "examples": {
          "user" : {
            "summary": "User Example", 
            "externalValue": "https://foo.bar/examples/user-example.json"
          } 
        }
    },
    "application/xml": {
      "schema": {
        "$ref": "#/components/schemas/User"
      },
      "examples": {
          "user" : {
            "summary": "User example in XML",
            "externalValue": "https://foo.bar/examples/user-example.xml"
          }
        }
    },
    "text/plain": {
      "examples": {
        "user" : {
            "summary": "User example in Plain text",
            "externalValue": "https://foo.bar/examples/user-example.txt" 
        }
      } 
    },
    "*/*": {
      "examples": {
        "user" : {
            "summary": "User example in other format",
            "externalValue": "https://foo.bar/examples/user-example.whatever"
        }
      }
    }
  }
}
description: user to add to the system
content: 
  'application/json':
    schema:
      $ref: '#/components/schemas/User'
    examples:
      user:
        summary: User Example
        externalValue: 'https://foo.bar/examples/user-example.json'
  'application/xml':
    schema:
      $ref: '#/components/schemas/User'
    examples:
      user:
        summary: User example in XML
        externalValue: 'https://foo.bar/examples/user-example.xml'
  'text/plain':
    examples:
      user:
        summary: User example in Plain text
        externalValue: 'https://foo.bar/examples/user-example.txt'
  '*/*':
    examples:
      user: 
        summary: User example in other format
        externalValue: 'https://foo.bar/examples/user-example.whatever'
```

一个主体参数，它是一个字符串值数组： 

```
{
  "description": "user to add to the system",
  "required": true,
  "content": {
    "text/plain": {
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      }
    }
  }
}
description: user to add to the system
required: true
content:
  text/plain:
    schema:
      type: array
      items:
        type: string
```

#### 

#### 媒体类型对象 

每个媒体类型对象都为由其键标识的媒体类型提供模式和示例。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 图式     | [架构对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject) | 定义请求、响应或参数内容的模式。                             |
| 例子     | 任何                                                         | 媒体类型示例。示例对象应该采用媒体类型指定的正确格式。这`example`字段是互斥的`examples`场地。此外，如果引用一个`schema`其中包含一个示例，`example`值应 *覆盖* 模式提供的示例。 |
| 例子     | Map[`string`,[示例对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#exampleObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 媒体类型的例子。每个示例对象应该匹配媒体类型和指定的模式（如果存在）。这`examples`字段是互斥的`example`场地。此外，如果引用一个`schema`其中包含一个示例，`examples`值应 *覆盖* 模式提供的示例。 |
| 编码     | Map[`string`,[编码对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingObject)] | 属性名称与其编码信息之间的映射。作为属性名称的键必须作为属性存在于模式中。编码对象应仅适用于`requestBody`媒体类型为`multipart`或者`application/x-www-form-urlencoded`. |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 媒体类型示例 

```
{
  "application/json": {
    "schema": {
         "$ref": "#/components/schemas/Pet"
    },
    "examples": {
      "cat" : {
        "summary": "An example of a cat",
        "value": 
          {
            "name": "Fluffy",
            "petType": "Cat",
            "color": "White",
            "gender": "male",
            "breed": "Persian"
          }
      },
      "dog": {
        "summary": "An example of a dog with a cat's name",
        "value" :  { 
          "name": "Puma",
          "petType": "Dog",
          "color": "Black",
          "gender": "Female",
          "breed": "Mixed"
        },
      "frog": {
          "$ref": "#/components/examples/frog-example"
        }
      }
    }
  }
}
application/json: 
  schema:
    $ref: "#/components/schemas/Pet"
  examples:
    cat:
      summary: An example of a cat
      value:
        name: Fluffy
        petType: Cat
        color: White
        gender: male
        breed: Persian
    dog:
      summary: An example of a dog with a cat's name
      value:
        name: Puma
        petType: Dog
        color: Black
        gender: Female
        breed: Mixed
    frog:
      $ref: "#/components/examples/frog-example"
```

##### 

##### 文件上传的注意事项 

对比2.0规范，`file`OpenAPI 中的输入/输出内容使用与任何其他模式类型相同的语义进行描述。

与 3.0 规范相比，`format`关键字对架构的内容编码没有影响。JSON Schema 提供了一个`contentEncoding`关键字，可用于指定`Content-Encoding`对于模式。这`contentEncoding`中定义的所有编码[关键字支持RFC4648](https://tools.ietf.org/html/rfc4648)，包括“base64”和“base64url”，以及[RFC2045](https://tools.ietf.org/html/rfc2045#section-6.7)中的“quoted-printable” 。由指定的编码`contentEncoding`关键字独立于由`Content-Type`请求或响应中的标头或多部分主体的元数据——当两者都存在时，编码在`contentEncoding`首先应用，然后应用中指定的编码`Content-Type`标头。

JSON Schema 还提供了一个`contentMediaType`关键词。但是，当媒体类型已经由媒体类型对象的键或由`contentType`的字段[编码对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingObject)，`contentMediaType`如果存在，则应忽略关键字。

例子： 

以二进制（八位字节流）传输的内容可以省略`schema`: 

```
# a PNG image as a binary file:
content:
    image/png: {}
# an arbitrary binary file:
content:
    application/octet-stream: {}
```

使用base64编码传输的二进制内容： 

```
content:
    image/png:
        schema:
            type: string
            contentMediaType: image/png
            contentEncoding: base64
```

请注意，`Content-Type`遗迹`image/png`，描述有效载荷的语义。JSON 模式`type`和`contentEncoding`字段说明有效载荷以文本形式传输。JSON 模式`contentMediaType`在技术上是多余的，但可以由可能不知道 OpenAPI 上下文的 JSON Schema 工具使用。

这些示例适用于文件上传的输入有效负载或响应有效负载。

A`requestBody`用于提交文件`POST`操作可能类似于以下示例： 

```
requestBody:
  content:
    application/octet-stream: {}
```

此外，可以指定特定的媒体类型： 

```
# multiple, specific media types may be specified:
requestBody:
  content:
    # a binary file of type png or jpeg
    image/jpeg: {}
    image/png: {}
```

要上传多个文件，一个`multipart`必须使用媒体类型： 

```
requestBody:
  content:
    multipart/form-data:
      schema:
        properties:
          # The property name 'file' will be used for all files.
          file:
            type: array
            items: {}
```

如上一节所示`multipart/form-data`下面，空模式`items`表示媒体类型`application/octet-stream`. 

##### 

##### 支持 x-www-form-urlencoded 请求体 

使用表单 url 编码提交内容[要通过RFC1866](https://tools.ietf.org/html/rfc1866)，请执行以下操作   可以使用定义： 

```
requestBody:
  content:
    application/x-www-form-urlencoded:
      schema:
        type: object
        properties:
          id:
            type: string
            format: uuid
          address:
            # complex types are stringified to support RFC 1866
            type: object
            properties: {}
```

在这个例子中，内容在`requestBody`必须按照[RFC1866](https://tools.ietf.org/html/rfc1866/)传递给服务器时 进行字符串化。除此之外`address`字段复杂对象将被字符串化。

在传递复杂对象时`application/x-www-form-urlencoded`内容类型，此类属性的默认序列化策略在[`Encoding Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingObject)的[`style`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingStyle)财产作为`form`. 

##### 

##### 特别注意事项`multipart`内容 

常用的是`multipart/form-data`作为一个`Content-Type`将请求主体传输到操作时。与 2.0 相比，`schema`需要在使用时定义操作的输入参数`multipart`内容。这支持复杂的结构以及多个文件上传的支持机制。

在一个`multipart/form-data`请求正文、每个架构属性或架构数组属性的每个元素在有效负载中采用一个部分，其中包含[RFC7578](https://tools.ietf.org/html/rfc7578)定义的内部标头。a的每个属性的序列化策略`multipart/form-data`请求正文可以在关联的[`Encoding Object`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingObject). 

路过时`multipart`types, boundaries 可以用来分隔正在传输的内容的各个部分——因此，以下默认`Content-Type`s 被定义为`multipart`: 

- 如果该属性是原始值或原始值数组，则默认的 Content-Type 是`text/plain`
- 如果属性很复杂，或者是复杂值的数组，则默认的 Content-Type 是`application/json`
- 如果财产是`type: string`与`contentEncoding`, 默认的 Content-Type 是`application/octet-stream`

根据 JSON Schema 规范，`contentMediaType`没有`contentEncoding`现在被视为`contentEncoding: identity`在场。虽然对于嵌入文本文档很有用，例如`text/html`转换成 JSON 字符串，它对`multipart/form-data`部分，因为它只会导致文档被视为`text/plain`而不是它的实际媒体类型。使用不带编码对象`contentMediaType`如果不`contentEncoding`是必须的。

例子： 

```
requestBody:
  content:
    multipart/form-data:
      schema:
        type: object
        properties:
          id:
            type: string
            format: uuid
          address:
            # default Content-Type for objects is`application/json`
            type: object
            properties: {}
          profileImage:
            # Content-Type for application-level encoded resource is`text/plain`
            type: string
            contentMediaType: image/png
            contentEncoding: base64
          children:
            # default Content-Type for arrays is based on the _inner_ type （`text/plain` here)
            type: array
            items:
              type: string
          addresses:
            # default Content-Type for arrays is based on the _inner_ type (object shown, so`application/json` in this example)
            type: array
            items:
              type: object
              $ref: '#/components/schemas/Address'
```

一个`encoding`引入属性是为了让您控制部分的序列化`multipart`请求机构。该属性 *仅* 适用于`multipart`和`application/x-www-form-urlencoded`请求机构。

#### 

#### 编码对象 

应用于单个架构属性的单个编码定义。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 内容类型 | `string`                                                     | 用于编码特定属性的 Content-Type。默认值取决于属性类型：for`object` -`application/json`;  为了`array`– 默认是根据内部类型定义的；  对于所有其他情况，默认值为`application/octet-stream`.  该值可以是特定的媒体类型（例如`application/json`）, 通配符媒体类型 (例如`image/*`）, 或两种类型的逗号分隔列表。 |
| 标题     | Map[`string`,[标题对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#headerObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 例如，允许将附加信息作为标题提供的Map`Content-Disposition`.`Content-Type`单独描述，在本节中应被忽略。如果请求主体媒体类型不是`multipart`. |
| 风格     | `string`                                                     | 描述特定属性值将如何根据其类型进行序列化。请参阅[参数对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterObject)  有关详细信息，[`style`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterStyle)财产。行为遵循与以下相同的值`query`参数，包括默认值。如果请求主体媒体类型不是，则应忽略此属性`application/x-www-form-urlencoded`或者`multipart/form-data`.  如果显式定义了一个值，则该值[`contentType`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingContentType)（隐式或显式）应被忽略。 |
| 爆炸     | `boolean`                                                    | 当这是真的时，类型的属性值`array`或者`object`为数组的每个值或映射的键值对生成单独的参数。对于其他类型的属性，此属性无效。什么时候[`style`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingStyle)是`form`，默认值为`true`.  对于所有其他样式，默认值为`false`.  如果请求主体媒体类型不是，则应忽略此属性`application/x-www-form-urlencoded`或者`multipart/form-data`.  如果显式定义了一个值，则该值[`contentType`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingContentType)（隐式或显式）应被忽略。 |
| 允许保留 | `boolean`                                                    | 确定参数值是否应该允许保留字符，如[RFC3986所定义](https://tools.ietf.org/html/rfc3986#section-2.2)`:/?#[]@!$&'()*+,;=`无需百分比编码即可包含在内。默认值为`false`.  如果请求主体媒体类型不是，则应忽略此属性`application/x-www-form-urlencoded`或者`multipart/form-data`.  如果显式定义了一个值，则该值[`contentType`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#encodingContentType)（隐式或显式）应被忽略。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 编码对象示例 

```
requestBody:
  content:
    multipart/form-data:
      schema:
        type: object
        properties:
          id:
            # default is text/plain
            type: string
            format: uuid
          address:
            # default is application/json
            type: object
            properties: {}
          historyMetadata:
            # need to declare XML format!
            description: metadata in XML format
            type: object
            properties: {}
          profileImage: {}
      encoding:
        historyMetadata:
          # require XML Content-Type in utf-8 encoding
          contentType: application/xml; charset=utf-8
        profileImage:
          # only accept png/jpeg
          contentType: image/png, image/jpeg
          headers:
            X-Rate-Limit-Limit:
              description: The number of allowed requests in the current period
              schema:
                type: integer
```

#### 

#### 响应对象 

操作的预期响应的容器。容器将 HTTP 响应代码映射到预期的响应。

文档不一定涵盖所有可能的 HTTP 响应代码，因为它们可能无法提前获知。但是，文档应涵盖成功的操作响应和任何已知错误。

这`default`可以用作所有 HTTP 代码的默认响应对象 未单独涵盖的`Responses Object`. 

这`Responses Object`必须包含至少一个响应代码，如果只有一个 提供了响应代码，它应该是成功操作的响应 称呼。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 默认     | [响应对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#responseObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject) | 除了为特定 HTTP 响应代码声明的响应之外的响应文档。使用此字段来涵盖未声明的响应。 |

##### 

##### 图案字段 

| 字段模式                                                     | 类型                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [HTTP 状态码](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#httpCodes) | [响应对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#responseObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject) | 任何[HTTP 状态代码](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#httpCodes)都可以用作属性名称，但每个代码只能使用一个属性，以描述该 HTTP 状态代码的预期响应。为了 JSON 和 YAML 之间的兼容性，此字段必须用引号引起来（例如，“200”）。要定义一系列响应代码，此字段可以包含大写通配符`X`.  例如，`2XX`表示之间的所有响应代码`[200-299]`.  仅允许以下范围定义：`1XX`,`2XX`,`3XX`,`4XX`，和`5XX`.  如果使用显式代码定义响应，则显式代码定义优先于该代码的范围定义。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 响应对象示例 

成功操作的 200 响应和其他默认响应（暗示错误）： 

```
{
  "200": {
    "description": "a pet to be returned",
    "content": {
      "application/json": {
        "schema": {
          "$ref": "#/components/schemas/Pet"
        }
      }
    }
  },
  "default": {
    "description": "Unexpected error",
    "content": {
      "application/json": {
        "schema": {
          "$ref": "#/components/schemas/ErrorModel"
        }
      }
    }
  }
}
'200':
  description: a pet to be returned
  content: 
    application/json:
      schema:
        $ref: '#/components/schemas/Pet'
default:
  description: Unexpected error
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/ErrorModel'
```

#### 

#### 响应对象 

描述来自 API 操作的单个响应，包括设计时、静态`links`到基于响应的操作。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 描述     | `string`                                                     | **必需的**。响应的描述。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 标题     | Map[`string`,[标题对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#headerObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 将标头名称映射到其定义。[RFC7230](https://tools.ietf.org/html/rfc7230#page-22)声明标头名称不区分大小写。如果响应标头是用名称定义的`"Content-Type"`, 它应被忽略。 |
| 内容     | Map[`string`,[媒体类型对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#mediaTypeObject)] | 包含潜在响应负载描述的Map。键是媒体类型或[媒体类型范围](https://tools.ietf.org/html/rfc7231#appendix-D)，值描述它。对于匹配多个键的响应，只有最具体的键适用。例如 text/plain 覆盖 text/* |
| 链接     | Map[`string`,[链接对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#linkObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject)] | 可以从响应中遵循的操作链接图。映射的键是链接的短名称，遵循[组件对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#componentsObject)名称的命名约束。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 响应对象示例 

复杂类型数组的响应： 

```
{
  "description": "A complex object array response",
  "content": {
    "application/json": {
      "schema": {
        "type": "array",
        "items": {
          "$ref": "#/components/schemas/VeryComplexType"
        }
      }
    }
  }
}
description: A complex object array response
content: 
  application/json:
    schema: 
      type: array
      items:
        $ref: '#/components/schemas/VeryComplexType'
```

字符串类型的响应： 

```
{
  "description": "A simple string response",
  "content": {
    "text/plain": {
      "schema": {
        "type": "string"
      }
    }
  }

}
description: A simple string response
content:
  text/plain:
    schema:
      type: string
```

带有标题的纯文本响应： 

```
{
  "description": "A simple string response",
  "content": {
    "text/plain": {
      "schema": {
        "type": "string",
        "example": "whoa!"
      }
    }
  },
  "headers": {
    "X-Rate-Limit-Limit": {
      "description": "The number of allowed requests in the current period",
      "schema": {
        "type": "integer"
      }
    },
    "X-Rate-Limit-Remaining": {
      "description": "The number of remaining requests in the current period",
      "schema": {
        "type": "integer"
      }
    },
    "X-Rate-Limit-Reset": {
      "description": "The number of seconds left in the current period",
      "schema": {
        "type": "integer"
      }
    }
  }
}
description: A simple string response
content:
  text/plain:
    schema:
      type: string
    example: 'whoa!'
headers:
  X-Rate-Limit-Limit:
    description: The number of allowed requests in the current period
    schema:
      type: integer
  X-Rate-Limit-Remaining:
    description: The number of remaining requests in the current period
    schema:
      type: integer
  X-Rate-Limit-Reset:
    description: The number of seconds left in the current period
    schema:
      type: integer
```

没有返回值的响应： 

```
{
  "description": "object created"
}
description: object created
```

#### 

#### 回调对象 

与父操作相关的可能带外回调的映射。映射中的每个值都是一个[路径项对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject)，它描述了一组可能由 API 提供者发起的请求和预期的响应。用于标识路径项对象的键值是一个表达式，在运行时计算，它标识用于回调操作的 URL。

要独立于另一个 API 调用来描述来自 API 提供者的传入请求，请使用[`webhooks`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasWebhooks)场地。

##### 

##### 图案字段 

| 字段模式 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| {表达}   | [路径项对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject)\|[参考对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#referenceObject) | 用于定义回调请求和预期响应的路径项对象或对对象的引用。一个[完整的例子](https://github.com/OAI/OpenAPI-Specification/blob/main/examples/v3.0/callback-example.yaml)是可用的。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 关键表达 

的键[标识路径项对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject)是一个[运行时表达式](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#runtimeExpression)，可以在运行时 HTTP 请求/响应的上下文中对其进行评估，以标识要用于回调请求的 URL。一个简单的例子可能是`$request.body#/url`.   但是，使用[运行时表达式](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#runtimeExpression)可以访问完整的 HTTP 消息。的正文的任何部分。[这包括访问 JSON 指针RFC6901](https://tools.ietf.org/html/rfc6901)可以引用 

例如，给定以下 HTTP 请求： 

```
POST /subscribe/myevent?queryUrl=https://clientdomain.com/stillrunning HTTP/1.1
Host: example.org
Content-Type: application/json
Content-Length: 187

{
  "failedUrl" : "https://clientdomain.com/failed",
  "successUrls" :[
    "https://clientdomain.com/fast",
    "https://clientdomain.com/medium",
    "https://clientdomain.com/slow"
] 
}

201 Created
Location: https://example.org/subscription/1
```

以下示例显示了各种表达式的计算方式，假设回调操作有一个名为`eventType`和一个名为的查询参数`queryUrl`. 

| 表达                         | 价值                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| $网址                        | [https://example.org/subscribe/myevent?queryUrl=https://clientdomain.com/stillrunning](https://example.org/subscribe/myevent?queryUrl=https://clientdomain.com/stillrunning) |
| $方法                        | 邮政                                                         |
| $request.path.eventType      | 我的事件                                                     |
| $request.query.queryUrl      | [https://clientdomain.com/stillrunning](https://clientdomain.com/stillrunning) |
| $request.header.content-类型 | 应用程序/json                                                |
| $request.body#/failedUrl     | [https://clientdomain.com/failed](https://clientdomain.com/failed) |
| $request.body#/successUrls/2 | [https://clientdomain.com/medium](https://clientdomain.com/medium) |
| $response.header.Location    | [https://example.org/subscription/1](https://example.org/subscription/1) |

##### 

##### 回调对象示例 

以下示例使用用户提供的`queryUrl`查询字符串参数来定义回调 URL。这是一个示例，说明如何使用回调对象来描述与订阅操作一起使用的 WebHook 回调，以启用对 WebHook 的注册。

```
myCallback:
  '{$request.query.queryUrl}':
    post:
      requestBody:
        description: Callback payload
        content:
          'application/json':
            schema:
              $ref: '#/components/schemas/SomePayload'
      responses:
        '200':
          description: callback successfully processed
```

以下示例显示了一个回调，其中服务器是硬编码的，但查询字符串参数是从`id`和`email`请求正文中的属性。

```
transactionCallback:
  'http://notificationServer.com?transactionId={$request.body#/id}&email={$request.body#/email}':
    post:
      requestBody:
        description: Callback payload
        content:
          'application/json':
            schema:
              $ref: '#/components/schemas/SomePayload'
      responses:
        '200':
          description: callback successfully processed
```

#### 

#### 示例对象 

##### 

##### 固定字段 

| 字段名称 | 类型     | 描述                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| 概括     | `string` | 示例的简短描述。                                             |
| 描述     | `string` | 示例的详细说明。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 价值     | 任何     | 嵌入式文字示例。这`value`领域和`externalValue`领域是互斥的。要表示无法在 JSON 或 YAML 中自然表示的媒体类型的示例，请使用字符串值来包含示例，并在必要时转义。 |
| 外部值   | `string` | 指向文字示例的 URI。这提供了引用无法轻松包含在 JSON 或 YAML 文档中的示例的功能。这`value`领域和`externalValue`领域是互斥的。的规则[请参阅解析Relative References](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#relativeReferencesURI)。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

在所有情况下，示例值都应与类型模式兼容 其相关价值。工具实现可以选择 自动验证兼容性，如果不兼容则拒绝示例值。

##### 

##### 示例对象示例 

在请求正文中： 

```
requestBody:
  content:
    'application/json':
      schema:
        $ref: '#/components/schemas/Address'
      examples: 
        foo:
          summary: A foo example
          value: {"foo": "bar"}
        bar:
          summary: A bar example
          value: {"bar": "baz"}
    'application/xml':
      examples: 
        xmlExample:
          summary: This is an example in XML
          externalValue: 'https://example.org/examples/address-example.xml'
    'text/plain':
      examples:
        textExample: 
          summary: This is a text example
          externalValue: 'https://foo.bar/examples/address-example.txt'
```

在一个参数中： 

```
parameters:
  - name: 'zipCode'
    in: 'query'
    schema:
      type: 'string'
      format: 'zip-code'
    examples:
      zip-example: 
        $ref: '#/components/examples/zip-example'
```

在回应中： 

```
responses:
  '200':
    description: your car appointment has been booked
    content: 
      application/json:
        schema:
          $ref: '#/components/schemas/SuccessResponse'
        examples:
          confirmation-success:
            $ref: '#/components/examples/confirmation-success'
```

#### 

#### 链接对象 

这`Link object`表示响应的可能设计时链接。链接的存在并不能保证调用者能够成功调用它，而是它提供了响应和其他操作之间的已知关系和遍历机制。

）不同 *与动态* 提供的链接**链接（即响应负载中**，OAS 链接机制不需要运行时响应中的链接信息。

为了计算链接并提供执行它们的指令，[运行时表达式](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#runtimeExpression)用于访问操作中的值并在调用链接操作时将它们用作参数。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 操作参考 | `string`                                                     | 对 OAS 操作的相对或绝对 URI 引用。该字段与`operationId`字段，并且必须指向一个[操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject)。相对的`operationRef`值可以用于[定位现有的操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject)在 OpenAPI 定义中 的规则[。请参阅解析Relative References](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#relativeReferencesURI)。 |
| 操作编号 | `string`                                                     | 、可解析的 OAS 操作的名称 *现有的* ，用唯一的定义`operationId`.  该字段与`operationRef`场地。 |
| 参数     | Map[`string`, 任何 \|[{表达式}](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#runtimeExpression)] | 表示要传递给指定的操作的参数的映射`operationId`或通过识别`operationRef`.   键是要使用的参数名称，而值可以是常量或要计算并传递给链接操作的表达式。来限定参数名称[可以使用参数位置](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterIn)`[{in}.]{name}`对于在不同位置使用相同参数名称的操作（例如 path.id）。 |
| 请求体   | 任何\|[{表达}](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#runtimeExpression) | 文字值或[{expression} 。](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#runtimeExpression)调用目标操作时用作请求主体的 |
| 描述     | `string`                                                     | 链接的描述。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 服务器   | [服务器对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#serverObject) | 目标操作要使用的服务器对象。                                 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

链接操作必须使用`operationRef`或者`operationId`. 在一个情况下`operationId`, 它必须是唯一的并且在 OAS 文档的范围内被解析。由于名称冲突的可能性，`operationRef`语法优先 对于带有外部引用的 OpenAPI 文档。

##### 

##### 例子 

从请求操作计算链接，其中`$request.path.id`用于将请求参数传递给链接操作。

```
paths:
  /users/{id}:
    parameters:
    - name: id
      in: path
      required: true
      description: the user identifier, as userId 
      schema:
        type: string
    get:
      responses:
        '200':
          description: the user being returned
          content:
            application/json:
              schema:
                type: object
                properties:
                  uuid: # the unique user id
                    type: string
                    format: uuid
          links:
            address:
              # the target link operationId
              operationId: getUserAddress
              parameters:
                # get the`id` field from the request path parameter named`id`
                userId: $request.path.id
  # the path item of the linked operation
  /users/{userid}/address:
    parameters:
    - name: userid
      in: path
      required: true
      description: the user identifier, as userId 
      schema:
        type: string
    # linked operation
    get:
      operationId: getUserAddress
      responses:
        '200':
          description: the user's address
```

当运行时表达式无法计算时，不会将任何参数值传递给目标操作。

来自响应主体的值可用于驱动链接操作。

```
links:
  address:
    operationId: getUserAddressByUUID
    parameters:
      # get the`uuid` field from the`uuid` field in the response body
      userUuid: $response.body#/uuid
```

客户自行决定是否访问所有链接。既不保证权限也不保证成功调用该链接的能力 仅仅因为关系的存在。

##### 

##### OperationRef 示例 

作为参考`operationId`可能不可能（`operationId`是可选的  中的字段[Operation Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject)），也可以通过亲戚进行引用`operationRef`: 

```
links:
  UserRepositories:
    # returns array of '#/components/schemas/repository'
    operationRef: '#/paths/~12.0~1repositories~1{username}/get'
    parameters:
      username: $response.body#/username
```

或绝对`operationRef`: 

```
links:
  UserRepositories:
    # returns array of '#/components/schemas/repository'
    operationRef: 'https://na2.gigantic-server.com/#/paths/~12.0~1repositories~1{username}/get'
    parameters:
      username: $response.body#/username
```

请注意，在使用`operationRef`, *转义的正斜杠* 是必要的   使用 JSON 引用。

##### 

##### 运行时表达式 

运行时表达式允许根据仅在实际 API 调用中的 HTTP 消息中可用的信息来定义值。使用此机制[链接对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#linkObject)和[回调对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#callbackObject)。

运行时表达式由以下[ABNF](https://tools.ietf.org/html/rfc5234)语法定义 

```
      expression = ( "$url" / "$method" / "$statusCode" / "$request." source / "$response." source )
      source = ( header-reference / query-reference / path-reference / body-reference )
      header-reference = "header." token
      query-reference = "query." name  
      path-reference = "path." name
      body-reference = "body"["#" json-pointer]
      json-pointer    = *( "/" reference-token )
      reference-token = *( unescaped / escaped )
      unescaped       = %x00-2E / %x30-7D / %x7F-10FFFF
         ; %x2F ('/') and %x7E ('~') are excluded from 'unescaped'
      escaped         = "~" ( "0" / "1" )
        ; representing '~' and '/', respectively
      name = *( CHAR )
      token = 1*tchar
      tchar = "!" / "#" / "$" / "%" / "&" / "'" / "*" / "+" / "-" / "." /
        "^" / "_" / "`" / "|" / "~" / DIGIT / ALPHA
```

这里，`json-pointer`取自[RFC6901](https://tools.ietf.org/html/rfc6901) ，`char`来自[RFC7159](https://tools.ietf.org/html/rfc7159#section-7)和`token`来自[RFC7230](https://tools.ietf.org/html/rfc7230#section-3.2.6) 。

这`name`标识符区分大小写，而`token`不是。

下表提供了运行时表达式的示例以及它们在值中的使用示例： 

##### 

##### 例子 

| 源位置         | 示例表达式                 | 笔记                                                         |
| -------------- | -------------------------- | ------------------------------------------------------------ |
| 方法           | `$method`                  | 的允许值`$method`将是那些用于 HTTP 操作的。                  |
| 请求的媒体类型 | `$request.header.accept`   |                                                              |
| 请求参数       | `$request.path.id`         | 请求参数必须在`parameters`父操作的一部分，否则无法对其进行评估。这包括请求标头。 |
| 请求正文属性   | `$request.body#/user/uuid` | 在接受有效载荷的操作中，可以参考`requestBody`或整个身体。    |
| 请求网址       | `$url`                     |                                                              |
| 响应值         | `$response.body#/status`   | 在返回有效负载的操作中，可以引用部分响应主体或整个主体。     |
| 响应头         | `$response.header.Server`  | 仅单个标头值可用                                             |

运行时表达式保留引用值的类型。可以通过将表达式包围在表达式中来将表达式嵌入到字符串值中`{}`大括号。

#### 

#### 标头对象 

的结构，[Header 对象遵循Parameter 对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterObject)但有以下变化： 

1. `name`不得指定，它在相应的`headers`Map。
2. `in`不得指定，它隐含在`header`. 
3. 受位置影响的所有特征必须适用于`header`（例如，[`style`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#parameterStyle)). 

##### 

##### 标头对象示例 

一个简单的标题类型`integer`: 

```
{
  "description": "The number of allowed requests in the current period",
  "schema": {
    "type": "integer"
  }
}
description: The number of allowed requests in the current period
schema:
  type: integer
```

#### 

#### 标签对象 

使用的单个标记[将元数据添加到Operation Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject)。在操作对象实例中定义的每个标签都不是强制性的。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 姓名     | `string`                                                     | **必需的**。标记的名称。                                     |
| 描述     | `string`                                                     | 标签的描述。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 外部文档 | [外部文档对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#externalDocumentationObject) | 此标签的附加外部文档。                                       |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 标记对象示例 

```
{
	"name": "pet",
	"description": "Pets operations"
}
name: pet
description: Pets operations
```

#### 

#### 参考对象 

一个简单的对象，允许在内部和外部引用 OpenAPI 文档中的其他组件。

这`$ref`字符串值包含 URI[RFC3986](https://tools.ietf.org/html/rfc3986) ，它标识被引用值的位置。

的规则[请参阅解析Relative References](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#relativeReferencesURI)。

##### 

##### 固定字段 

| 字段名称 | 类型     | 描述                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| $ref     | `string` | **必需的**。参考标识符。这必须采用 URI 的形式。              |
| 概括     | `string` | 一个简短的摘要，默认情况下应该覆盖引用组件的摘要。如果引用的对象类型不允许`summary`字段，则该字段无效。 |
| 描述     | `string` | 默认情况下应该覆盖引用组件的描述。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。如果引用的对象类型不允许`description`字段，则该字段无效。 |

该对象不能用附加属性扩展，任何添加的属性都应被忽略。

请注意，对附加属性的这种限制是引用对象和[`Schema Objects`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject)包含一个`$ref`关键词。

##### 

##### 参考对象示例 

```
{
	"$ref": "#/components/schemas/Pet"
}
$ref: '#/components/schemas/Pet'
```

##### 

##### 相对架构文档示例 

```
{
  "$ref": "Pet.json"
}
$ref: Pet.yaml
```

##### 

##### 具有嵌入式架构示例的相关文档 

```
{
  "$ref": "definitions.json#/Pet"
}
$ref: definitions.yaml#/Pet
```

#### 

#### 架构对象 

模式对象允许定义输入和输出数据类型。这些类型可以是对象，也可以是基元和数组。的超集[该对象是JSON Schema Specification Draft 2020-12](https://tools.ietf.org/html/draft-bhutton-json-schema-00)。

有关属性的更多信息，请参阅[JSON Schema Core](https://tools.ietf.org/html/draft-bhutton-json-schema-00)和[JSON Schema Validation](https://tools.ietf.org/html/draft-bhutton-json-schema-validation-00) 。

除非另有说明，否则属性定义遵循 JSON Schema 的定义，并且不添加任何额外的语义。在 JSON Schema 指示行为由应用程序定义的地方（例如注释），OAS 还将语义定义推迟到使用 OpenAPI 文档的应用程序。

##### 

##### 特性 

 OpenAPI Schema Object[方言](https://tools.ietf.org/html/draft-bhutton-json-schema-00#section-4.3.3)被定义为需要[词汇表](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#baseVocabulary)除了 JSON Schema 草案 2020-12 通用元模式中指定的词汇表之外，[OAS基础](https://tools.ietf.org/html/draft-bhutton-json-schema-00#section-8)。

此版本规范的 OpenAPI 架构对象方言由 URI 标识`https://spec.openapis.org/oas/3.1/dialect/base`（  “OAS 方言架构 ID”）。

以下属性取自 JSON Schema 规范，但它们的定义已由 OAS 扩展： 

- 描述 -[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。
- format - 有关更多详细信息，请参阅[数据类型格式](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#dataTypeFormat)。在依赖 JSON Schema 定义的格式的同时，OAS 提供了一些额外的预定义格式。

除了包含 OAS 方言的 JSON 架构属性之外，架构对象还支持来自任何其他词汇表的关键字，或完全任意的属性。

OpenAPI 规范的基本词汇表由以下关键字组成： 

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 判别器   | [鉴别器对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#discriminatorObject) | 添加对多态性的支持。鉴别器是一个对象名称，用于区分可能满足有效载荷描述的其他模式。请参阅[组合和继承。](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaComposition)  有关详细信息， |
| XML      | [XML对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#xmlObject) | 这可能仅用于属性模式。它对根模式没有影响。添加额外的元数据来描述此属性的 XML 表示。 |
| 外部文档 | [外部文档对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#externalDocumentationObject) | 此架构的其他外部文档。                                       |
| 例子     | 任何                                                         | 一个自由格式的属性，用于包含此模式的实例示例。为了表示无法在 JSON 或 YAML 中自然表示的示例，可以使用字符串值来包含示例，并在必要时进行转义。**：**弃用`example`属性已被弃用，取而代之的是 JSON 模式`examples`关键词。用于`example`不鼓励，本规范的更高版本可能会删除它。 |

该对象可以使用[Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)进行扩展，但如前所述，附加属性可以省略`x-`此对象中的前缀。

###### 

###### 组合与继承（多态）

OpenAPI 规范允许使用`allOf`JSON Schema 的属性，实际上提供模型组合。`allOf`采用一组独立验证 *但* 共同组成单个对象的对象定义。

虽然组合提供了模型可扩展性，但并不意味着模型之间存在层次结构。为了支持多态性，OpenAPI 规范添加了`discriminator`场地。使用时，`discriminator`将是决定哪个模式定义验证模型结构的属性的名称。因此，`discriminator`字段必须是必填字段。有两种方法可以为继承实例定义鉴别器的值。

- 使用架构名称。
- 通过使用新值覆盖属性来覆盖架构名称。如果存在新值，则它优先于架构名称。因此，没有给定 id 的内联模式定义 *不能* 用于多态性。

###### 

###### XML 建模 

属性[xml](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaXml)将 JSON 定义转换为 XML 时，允许额外的定义。包含[XML 对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#xmlObject)有关可用选项的附加信息。

###### 

###### 指定模式方言 

重要的是工具能够确定任何给定资源希望使用哪种方言或元模式进行处理：JSON 模式核心、JSON 模式验证、OpenAPI 模式方言或一些自定义元模式。

这`$schema`关键字可以存在于任何根模式对象中，如果存在，则必须用于确定在处理模式时应使用哪种方言。这允许使用符合其他 JSON 架构草案而不是默认的 2020-12 草案支持的架构对象。工具必须支持[OAS 方言模式 id](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#dialectSchemaId) ，并且可以支持额外的值`$schema`. 

允许使用不同的默认值`$schema`OAS 文档中包含的所有模式对象的值，一个`jsonSchemaDialect`中设置[值可以在OpenAPI 对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasObject)。如果未设置此默认值，则 OAS 方言架构 ID 必须用于这些架构对象。的价值`$schema`在模式对象中总是覆盖任何默认值。

当从非 OAS 文档的外部资源（例如，裸 JSON 模式资源）引用模式对象时，则`$schema`该资源中模式的关键字必须遵循[JSON 模式规则](https://tools.ietf.org/html/draft-bhutton-json-schema-00#section-8.1.1)。

##### 

##### 架构对象示例 

###### 

###### 原始样本 

```
{
  "type": "string",
  "format": "email"
}
type: string
format: email
```

###### 

###### 简单模型 

```
{
  "type": "object",
  "required":[
    "name"
],
  "properties": {
    "name": {
      "type": "string"
    },
    "address": {
      "$ref": "#/components/schemas/Address"
    },
    "age": {
      "type": "integer",
      "format": "int32",
      "minimum": 0
    }
  }
}
type: object
required:
- name
properties:
  name:
    type: string
  address:
    $ref: '#/components/schemas/Address'
  age:
    type: integer
    format: int32
    minimum: 0
```

###### 

###### 具有Map/字典属性的模型 

对于简单的字符串到字符串映射： 

```
{
  "type": "object",
  "additionalProperties": {
    "type": "string"
  }
}
type: object
additionalProperties:
  type: string
```

对于字符串到模型的映射： 

```
{
  "type": "object",
  "additionalProperties": {
    "$ref": "#/components/schemas/ComplexModel"
  }
}
type: object
additionalProperties:
  $ref: '#/components/schemas/ComplexModel'
```

###### 

###### 示例模型 

```
{
  "type": "object",
  "properties": {
    "id": {
      "type": "integer",
      "format": "int64"
    },
    "name": {
      "type": "string"
    }
  },
  "required":[
    "name"
],
  "example": {
    "name": "Puma",
    "id": 1
  }
}
type: object
properties:
  id:
    type: integer
    format: int64
  name:
    type: string
required:
- name
example:
  name: Puma
  id: 1
```

###### 

###### 具有组合的模型 

```
{
  "components": {
    "schemas": {
      "ErrorModel": {
        "type": "object",
        "required":[
          "message",
          "code"
      ],
        "properties": {
          "message": {
            "type": "string"
          },
          "code": {
            "type": "integer",
            "minimum": 100,
            "maximum": 600
          }
        }
      },
      "ExtendedErrorModel": {
        "allOf":[
          {
            "$ref": "#/components/schemas/ErrorModel"
          },
          {
            "type": "object",
            "required":[
              "rootCause"
          ],
            "properties": {
              "rootCause": {
                "type": "string"
              }
            }
          }
      ]
      }
    }
  }
}
components:
  schemas:
    ErrorModel:
      type: object
      required:
      - message
      - code
      properties:
        message:
          type: string
        code:
          type: integer
          minimum: 100
          maximum: 600
    ExtendedErrorModel:
      allOf:
      - $ref: '#/components/schemas/ErrorModel'
      - type: object
        required:
        - rootCause
        properties:
          rootCause:
            type: string
```

###### 

###### 支持多态性的模型 

```
{
  "components": {
    "schemas": {
      "Pet": {
        "type": "object",
        "discriminator": {
          "propertyName": "petType"
        },
        "properties": {
          "name": {
            "type": "string"
          },
          "petType": {
            "type": "string"
          }
        },
        "required":[
          "name",
          "petType"
      ]
      },
      "Cat": {
        "description": "A representation of a cat. Note that`Cat` will be used as the discriminator value.",
        "allOf":[
          {
            "$ref": "#/components/schemas/Pet"
          },
          {
            "type": "object",
            "properties": {
              "huntingSkill": {
                "type": "string",
                "description": "The measured skill for hunting",
                "default": "lazy",
                "enum":[
                  "clueless",
                  "lazy",
                  "adventurous",
                  "aggressive"
              ]
              }
            },
            "required":[
              "huntingSkill"
          ]
          }
      ]
      },
      "Dog": {
        "description": "A representation of a dog. Note that`Dog` will be used as the discriminator value.",
        "allOf":[
          {
            "$ref": "#/components/schemas/Pet"
          },
          {
            "type": "object",
            "properties": {
              "packSize": {
                "type": "integer",
                "format": "int32",
                "description": "the size of the pack the dog is from",
                "default": 0,
                "minimum": 0
              }
            },
            "required":[
              "packSize"
          ]
          }
      ]
      }
    }
  }
}
components:
  schemas:
    Pet:
      type: object
      discriminator:
        propertyName: petType
      properties:
        name:
          type: string
        petType:
          type: string
      required:
      - name
      - petType
    Cat:  ## "Cat" will be used as the discriminator value
      description: A representation of a cat
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        properties:
          huntingSkill:
            type: string
            description: The measured skill for hunting
            enum:
            - clueless
            - lazy
            - adventurous
            - aggressive
        required:
        - huntingSkill
    Dog:  ## "Dog" will be used as the discriminator value
      description: A representation of a dog
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        properties:
          packSize:
            type: integer
            format: int32
            description: the size of the pack the dog is from
            default: 0
            minimum: 0
        required:
        - packSize
```

#### 

#### 鉴别器对象 

当请求主体或响应有效负载可能是许多不同模式之一时，一个`discriminator`对象可用于帮助序列化、反序列化和验证。鉴别器是模式中的特定对象，用于根据与其关联的值通知文档的消费者替代模式。

使用鉴别器时，*内联模式。* 将不考虑 

##### 

##### 固定字段 

| 字段名称 | 类型                   | 描述                                                 |
| -------- | ---------------------- | ---------------------------------------------------- |
| 财产名称 | `string`               | **必需的**。负载中将保存鉴别器值的属性名称。         |
| 映射     | Map[`string`,`string`] | 一个对象，用于保存负载值和架构名称或引用之间的映射。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

鉴别器对象仅在使用复合关键字之一时才合法`oneOf`,`anyOf`,`allOf`. 

在 OAS 3.0 中，响应负载可以被描述为任意数量类型中的一种： 

```
MyResponseType:
  oneOf:
  - $ref: '#/components/schemas/Cat'
  - $ref: '#/components/schemas/Dog'
  - $ref: '#/components/schemas/Lizard'
```

这意味着有效载荷 *必须* 通过验证与描述的模式之一完全匹配`Cat`,`Dog`，或者`Lizard`.  在这种情况下，鉴别器可以充当“提示”以快捷验证和选择匹配模式，这可能是一项成本高昂的操作，具体取决于模式的复杂性。然后我们可以准确描述哪个字段告诉我们要使用哪个模式： 

```
MyResponseType:
  oneOf:
  - $ref: '#/components/schemas/Cat'
  - $ref: '#/components/schemas/Dog'
  - $ref: '#/components/schemas/Lizard'
  discriminator:
    propertyName: petType
```

现在的期望是一个有名字的财产`petType` *必须* 出现在响应有效负载中，并且该值将对应于 OAS 文档中定义的模式的名称。因此响应有效载荷： 

```
{
  "id": 12345,
  "petType": "Cat"
}
```

将表明`Cat`模式与此有效负载结合使用。

在鉴别器字段的值与模式名称不匹配或隐式映射不可能的情况下，一个可选的`mapping`可以使用定义： 

```
MyResponseType:
  oneOf:
  - $ref: '#/components/schemas/Cat'
  - $ref: '#/components/schemas/Dog'
  - $ref: '#/components/schemas/Lizard'
  - $ref: 'https://gigantic-server.com/schemas/Monster/schema.json'
  discriminator:
    propertyName: petType
    mapping:
      dog: '#/components/schemas/Dog'
      monster: 'https://gigantic-server.com/schemas/Monster/schema.json'
```

判别器 *值* 这里的`dog`将映射到模式`#/components/schemas/Dog`，而不是默认（隐式）值`Dog`.   如果鉴别器 *值* 与隐式或显式映射不匹配，则无法确定模式并且验证应该失败。映射键必须是字符串值，但工具可以将响应值转换为字符串以进行比较。

当与`anyOf`构造，鉴别器的使用可以避免多个模式可能满足单个有效载荷的歧义。

在这两个`oneOf`和`anyOf`用例，必须明确列出所有可能的模式。为了避免冗余，鉴别器可以添加到父模式定义中，并且所有包含父模式的模式都在`allOf`构造可以用作替代模式。

例如： 

```
components:
  schemas:
    Pet:
      type: object
      required:
      - petType
      properties:
        petType:
          type: string
      discriminator:
        propertyName: petType
        mapping:
          dog: Dog
    Cat:
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        # all other properties specific to a`Cat`
        properties:
          name:
            type: string
    Dog:
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        # all other properties specific to a`Dog`
        properties:
          bark:
            type: string
    Lizard:
      allOf:
      - $ref: '#/components/schemas/Pet'
      - type: object
        # all other properties specific to a`Lizard`
        properties:
          lovesRocks:
            type: boolean
```

像这样的有效载荷： 

```
{
  "petType": "Cat",
  "name": "misty"
}
```

将表明`Cat`模式被使用。同样这个模式： 

```
{
  "petType": "dog",
  "bark": "soft"
}
```

将映射到`Dog`因为在定义`mapping`元素。

#### 

#### XML对象 

允许更精细调整的 XML 模型定义的元数据对象。

推断XML 元素名称 *使用数组时，不会*（对于单数/复数形式）并且`name`属性应该用于添加该信息。请参阅预期行为的示例。

##### 

##### 固定字段 

| 字段名称 | 类型      | 描述                                                         |
| -------- | --------- | ------------------------------------------------------------ |
| 姓名     | `string`  | 替换用于描述的架构属性的元素/属性的名称。当定义在`items`，它将影响列表中各个 XML 元素的名称。一起定义时`type`存在`array`（外`items`）, 它将影响包装元素并且仅当`wrapped`是`true`.  如果`wrapped`是`false`, 它将被忽略。 |
| 命名空间 | `string`  | 命名空间定义的 URI。这必须是绝对 URI 的形式。                |
| 字首     | `string`  | 用于名称的[前缀](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#xmlName)。 |
| 属性     | `boolean` | 声明属性定义是否转换为属性而不是元素。默认值为`false`.       |
| 包裹     | `boolean` | 可以仅用于数组定义。表示数组是否被包装（例如，`<books><book/><book/></books>`）或展开 （`<book/><book/>`）.  默认值为`false`.  该定义仅在同时定义时生效`type`存在`array`（外`items`）. |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### XML 对象示例 

XML 对象定义的示例包含在[示例的模式对象的属性定义中。](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#schemaObject)带有它的 XML 表示 

###### 

###### 无 XML 元素 

基本字符串属性： 

```
{
    "animals": {
        "type": "string"
    }
}
animals:
  type: string
<animals>...</animals>
```

基本字符串数组属性 ([`wrapped`](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#xmlWrapped)是`false`默认情况下）： 

```
{
    "animals": {
        "type": "array",
        "items": {
            "type": "string"
        }
    }
}
animals:
  type: array
  items:
    type: string
<animals>...</animals>
<animals>...</animals>
<animals>...</animals>
```

###### 

###### XML 名称替换 

```
{
  "animals": {
    "type": "string",
    "xml": {
      "name": "animal"
    }
  }
}
animals:
  type: string
  xml:
    name: animal
<animal>...</animal>
```

###### 

###### XML 属性、前缀和命名空间 

在此示例中，显示了完整的模型定义。

```
{
  "Person": {
    "type": "object",
    "properties": {
      "id": {
        "type": "integer",
        "format": "int32",
        "xml": {
          "attribute": true
        }
      },
      "name": {
        "type": "string",
        "xml": {
          "namespace": "https://example.com/schema/sample",
          "prefix": "sample"
        }
      }
    }
  }
}
Person:
  type: object
  properties:
    id:
      type: integer
      format: int32
      xml:
        attribute: true
    name:
      type: string
      xml:
        namespace: https://example.com/schema/sample
        prefix: sample
<Person id="123">
    <sample:name xmlns:sample="https://example.com/schema/sample">example</sample:name>
</Person>
```

###### 

###### XML 数组 

更改元素名称： 

```
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string",
      "xml": {
        "name": "animal"
      }
    }
  }
}
animals:
  type: array
  items:
    type: string
    xml:
      name: animal
<animal>value</animal>
<animal>value</animal>
```

外部的`name`属性对 XML 没有影响： 

```
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string",
      "xml": {
        "name": "animal"
      }
    },
    "xml": {
      "name": "aliens"
    }
  }
}
animals:
  type: array
  items:
    type: string
    xml:
      name: animal
  xml:
    name: aliens
<animal>value</animal>
<animal>value</animal>
```

即使数组被包装，如果没有显式定义名称，内部和外部也会使用相同的名称： 

```
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string"
    },
    "xml": {
      "wrapped": true
    }
  }
}
animals:
  type: array
  items:
    type: string
  xml:
    wrapped: true
<animals>
  <animals>value</animals>
  <animals>value</animals>
</animals>
```

为了克服上面示例中的命名问题，可以使用以下定义： 

```
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string",
      "xml": {
        "name": "animal"
      }
    },
    "xml": {
      "wrapped": true
    }
  }
}
animals:
  type: array
  items:
    type: string
    xml:
      name: animal
  xml:
    wrapped: true
<animals>
  <animal>value</animal>
  <animal>value</animal>
</animals>
```

影响内部和外部名称： 

```
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string",
      "xml": {
        "name": "animal"
      }
    },
    "xml": {
      "name": "aliens",
      "wrapped": true
    }
  }
}
animals:
  type: array
  items:
    type: string
    xml:
      name: animal
  xml:
    name: aliens
    wrapped: true
<aliens>
  <animal>value</animal>
  <animal>value</animal>
</aliens>
```

如果我们改变外部元素而不是内部元素： 

```
{
  "animals": {
    "type": "array",
    "items": {
      "type": "string"
    },
    "xml": {
      "name": "aliens",
      "wrapped": true
    }
  }
}
animals:
  type: array
  items:
    type: string
  xml:
    name: aliens
    wrapped: true
<aliens>
  <aliens>value</aliens>
  <aliens>value</aliens>
</aliens>
```

#### 

#### 安全方案对象 

定义操作可以使用的安全方案。

支持的方案是 HTTP 身份验证、API 密钥（作为标头、cookie 参数或作为查询参数）、双向 TLS（使用客户端证书）、OAuth2 的常见流程（隐式、密码、客户端凭据和授权代码）作为 中定义[在RFC6749](https://tools.ietf.org/html/rfc6749)和[OpenID Connect Discovery](https://tools.ietf.org/html/draft-ietf-oauth-discovery-06)。请注意，自 2020 年起，隐式流程将被[OAuth 2.0 Security Best Current Practice](https://tools.ietf.org/html/draft-ietf-oauth-security-topics)弃用。对于大多数用例，建议使用 PKCE 的授权代码授予流程。

##### 

##### 固定字段 

| 字段名称         | 类型                                                         | 适用于                | 描述                                                         |
| ---------------- | ------------------------------------------------------------ | --------------------- | ------------------------------------------------------------ |
| 类型             | `string`                                                     | 任何                  | **必需的**。安全方案的类型。有效值为`"apiKey"`,`"http"`,`"mutualTLS"`,`"oauth2"`,`"openIdConnect"`. |
| 描述             | `string`                                                     | 任何                  | 安全方案的描述。[CommonMark 语法](https://spec.commonmark.org/)可以用于富文本表示。 |
| 姓名             | `string`                                                     | `apiKey`              | **必需的**。要使用的标头、查询或 cookie 参数的名称。         |
| 在               | `string`                                                     | `apiKey`              | **必需的**。API 密钥的位置。有效值为`"query"`,`"header"`或者`"cookie"`. |
| 方案             | `string`                                                     | `http`                | **必需的**。中使用的 HTTP 授权方案的名称[在 RFC7235 中定义的授权标头](https://tools.ietf.org/html/rfc7235#section-5.1)。使用的值应该在[IANA 身份验证方案注册表](https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml)中注册。 |
| 承载格式         | `string`                                                     | `http` （`"bearer"`） | 提示客户端确定不记名令牌的格式。不记名令牌通常由授权服务器生成，因此此信息主要用于文档目的。 |
| 流动             | [OAuth 流对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oauthFlowsObject) | `oauth2`              | **必需的**。包含支持的流类型的配置信息的对象。               |
| openIdConnectUrl | `string`                                                     | `openIdConnect`       | **必需的**。用于发现 OAuth2 配置值的 OpenId Connect URL。这必须是 URL 的形式。OpenID Connect 标准要求使用 TLS。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### 安全方案对象示例 

###### 

###### 基本身份验证示例 

```
{
  "type": "http",
  "scheme": "basic"
}
type: http
scheme: basic
```

###### 

###### API 密钥示例 

```
{
  "type": "apiKey",
  "name": "api_key",
  "in": "header"
}
type: apiKey
name: api_key
in: header
```

###### 

###### JWT 不记名样本 

```
{
  "type": "http",
  "scheme": "bearer",
  "bearerFormat": "JWT",
}
type: http
scheme: bearer
bearerFormat: JWT
```

###### 

###### 隐式 OAuth2 示例 

```
{
  "type": "oauth2",
  "flows": {
    "implicit": {
      "authorizationUrl": "https://example.com/api/oauth/dialog",
      "scopes": {
        "write:pets": "modify pets in your account",
        "read:pets": "read your pets"
      }
    }
  }
}
type: oauth2
flows: 
  implicit:
    authorizationUrl: https://example.com/api/oauth/dialog
    scopes:
      write:pets: modify pets in your account
      read:pets: read your pets
```

#### 

#### OAuth 流对象 

允许配置支持的 OAuth 流程。

##### 

##### 固定字段 

| 字段名称 | 类型                                                         | 描述                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 含蓄的   | [OAuth 流对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oauthFlowObject) | OAuth 隐式流程的配置                                         |
| 密码     | [OAuth 流对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oauthFlowObject) | OAuth 资源所有者密码流程的配置                               |
| 客户凭证 | [OAuth 流对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oauthFlowObject) | OAuth 客户端凭证流的配置。以前叫`application`在 OpenAPI 2.0 中。 |
| 授权码   | [OAuth 流对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oauthFlowObject) | OAuth 授权代码流的配置。以前叫`accessCode`在 OpenAPI 2.0 中。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

#### 

#### OAuth 流对象 

受支持的 OAuth 流程的配置详细信息 

##### 

##### 固定字段 

| 字段名称 | 类型                   | 适用于                                                       | 描述                                                         |
| -------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 授权网址 | `string`               | `oauth2` （`"implicit"`,`"authorizationCode"`）              | **必需的**。用于此流的授权 URL。这必须是 URL 的形式。OAuth2 标准要求使用 TLS。 |
| 令牌网址 | `string`               | `oauth2` （`"password"`,`"clientCredentials"`,`"authorizationCode"`） | **必需的**。用于此流的令牌 URL。这必须是 URL 的形式。OAuth2 标准要求使用 TLS。 |
| 刷新网址 | `string`               | `oauth2`                                                     | 用于获取刷新令牌的 URL。这必须是 URL 的形式。OAuth2 标准要求使用 TLS。 |
| 范围     | Map[`string`,`string`] | `oauth2`                                                     | **必需的**。OAuth2 安全方案的可用范围。作用域名称和它的简短描述之间的映射。Map可能是空的。 |

来扩展[这个对象可以用Specification Extensions](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#specificationExtensions)。

##### 

##### OAuth 流对象示例 

```
{
  "type": "oauth2",
  "flows": {
    "implicit": {
      "authorizationUrl": "https://example.com/api/oauth/dialog",
      "scopes": {
        "write:pets": "modify pets in your account",
        "read:pets": "read your pets"
      }
    },
    "authorizationCode": {
      "authorizationUrl": "https://example.com/api/oauth/dialog",
      "tokenUrl": "https://example.com/api/oauth/token",
      "scopes": {
        "write:pets": "modify pets in your account",
        "read:pets": "read your pets"
      }
    }
  }
}
type: oauth2
flows: 
  implicit:
    authorizationUrl: https://example.com/api/oauth/dialog
    scopes:
      write:pets: modify pets in your account
      read:pets: read your pets
  authorizationCode:
    authorizationUrl: https://example.com/api/oauth/dialog
    tokenUrl: https://example.com/api/oauth/token
    scopes:
      write:pets: modify pets in your account
      read:pets: read your pets 
```

#### 

#### 安全需求对象 

列出执行此操作所需的安全方案。用于每个属性的名称必须对应于[安全方案](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#componentsSecuritySchemes)下的[组件对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#componentsObject)中声明的安全方案。

包含多个方案的安全要求对象要求必须满足所有方案才能授权请求。这支持需要多个查询参数或 HTTP 标头来传达安全信息的场景。

上定义了安全要求对象列表时[当在OpenAPI 对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#oasObject)或[操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operationObject)，只需满足列表中的一个安全要求对象即可授权请求。

##### 

##### 图案字段 

| 字段模式 | 类型       | 描述                                                         |
| -------- | ---------- | ------------------------------------------------------------ |
| {姓名}   | [`string`] | 中声明的安全方案[安全方案](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#componentsSecuritySchemes)下的[每个名称必须对应于在组件对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#componentsObject)。如果安全方案是类型`"oauth2"`或者`"openIdConnect"`，则该值是执行所需范围名称的列表，如果授权不需要指定范围，则该列表可以为空。对于其他安全方案类型，数组可以包含执行所需的角色名称列表，但不会以其他方式定义或带内交换。 |

##### 

##### 安全要求对象示例 

###### 

###### 非 OAuth2 安全要求 

```
{
  "api_key":[]
}
api_key:[]
```

###### 

###### OAuth2 安全要求 

```
{
  "petstore_auth":[
    "write:pets",
    "read:pets"
]
}
petstore_auth:
- write:pets
- read:pets
```

###### 

###### 可选的 OAuth2 安全性 

中定义[可选的 OAuth2 安全性将在OpenAPI 对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#openapi-object)或[操作对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#operation-object)： 

```
{
  "security":[
    {},
    {
      "petstore_auth":[
        "write:pets",
        "read:pets"
    ]
    }
]
}
security:
  - {}
  - petstore_auth:
    - write:pets
    - read:pets
```

### 

### 规范扩展 

虽然 OpenAPI 规范试图适应大多数用例，但可以添加额外的数据以在某些点扩展规范。

扩展属性被实现为总是以`"x-"`. 

| 字段模式 | 类型 | 描述                                                         |
| -------- | ---- | ------------------------------------------------------------ |
| ^x-      | 任何 | 允许对 OpenAPI 模式进行扩展。字段名称必须以`x-`，例如，`x-internal-id`.  字段名称开头`x-oai-`和`x-oas-`保留给[OpenAPI Initiative](https://www.openapis.org/)定义的用途。该值可以是`null`、原始数据、数组或对象。 |

可用工具可能支持也可能不支持扩展，但也可以扩展这些扩展以添加请求的支持（如果工具是内部的或开源的）。

### 

### 安全过滤 

OpenAPI 规范中的一些对象可以被声明并保持为空，或者被完全删除，即使它们本质上是 API 文档的核心。

原因是允许对文档进行额外的访问控制层。虽然不是规范本身的一部分，但某些库可以选择允许基于某种形式的身份验证/授权访问文档的部分内容。

这两个例子： 

1. 路径[对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathsObject)可能存在但为空。这可能违反直觉，但这可能会告诉查看者他们到达了正确的位置，但无法访问任何文档。他们仍然至少可以访问可能[信息对象。](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#infoObject)包含有关身份验证的附加信息的 
2. 路径[项目对象](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathItemObject)可以是空的。在这种情况下，查看者将意识到该路径存在，但无法看到其任何操作或参数。这不同于从[Paths Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.1.0.md#pathsObject)中隐藏路径本身，因为用户会知道它的存在。这允许文档提供者精细地控制查看者可以看到的内容。

## 

## 附录 A：修订历史 

| 版本      | 日期       | 笔记                                   |
| --------- | ---------- | -------------------------------------- |
| 3.1.0     | 2021-02-15 | OpenAPI 规范 3.1.0 发布                |
| 3.1.0-rc1 | 2020-10-08 | 3.1规范的rc1                           |
| 3.1.0-rc0 | 2020-06-18 | 3.1规范的rc0                           |
| 3.0.3     | 2020-02-20 | OpenAPI 规范 3.0.3 补丁发布            |
| 3.0.2     | 2018-10-08 | OpenAPI 规范 3.0.2 补丁发布            |
| 3.0.1     | 2017-12-06 | OpenAPI 规范 3.0.1 补丁发布            |
| 3.0.0     | 2017-07-26 | OpenAPI 规范 3.0.0 发布                |
| 3.0.0-rc2 | 2017-06-16 | 3.0规范的rc2                           |
| 3.0.0-rc1 | 2017-04-27 | 3.0规范的rc1                           |
| 3.0.0-rc0 | 2017-02-28 | 3.0 规范的实施者草案                   |
| 2.0       | 2015-12-31 | 向 OpenAPI Initiative 捐赠 Swagger 2.0 |
| 2.0       | 2014-09-08 | 发布 Swagger 2.0                       |
| 1.2       | 2014-03-14 | 正式文件的初始发布。                   |
| 1.1       | 2012-08-22 | 发布 Swagger 1.1                       |
| 1.0       | 2011-08-10 | 首次发布 Swagger 规范                  |

