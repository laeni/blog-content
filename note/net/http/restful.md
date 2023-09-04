---
title: HTTP接口响应格式定义参考（restful风格）
author: 'Laeni'
tags: HTTP, API, 规范
date: '2022-02-22'
updated: '2022-03-22'
---

做的好的情况下，一般需要根据不同的类型返回不同的错误响应，比如直接通过浏览器地址栏打开时可以返回`HTML`，通过`js`等`API`访问时可以`JSON`或者`XML`等。不过这里只考虑全部返回`JSON`格式，因为目前大部分系统为前后分离的，所有`API`一般都不直接通过浏览器访问。

此外，正常和异常状态下返回格式应该是不一样的，所以这里只是针对异常情况。如果出现异常，那一般情况下需要定位该问题并解决，所以异常情况下不仅要返回相关的错误信息，而且格式尽量一致，以便统一识别和处理（客户端或者服务端）。而正常情况下应该是根据具体API的业务特性来决定，比如返回结果可以是`JSON`字符串、一个普通字符串或者二进制的文件流等等，甚至还可以什么都不返回。

## 错误响应参考示例

### Spring

```json
{
    "timestamp": "2022-03-22T01:03:40.475+0000",
    "status": 400,
    "error": "Bad Request",
    // [可选]详细的错误信息
    "errors": [
        {
            "codes": [
                "NotBlank.signAddRequest.apiAcc",
                "NotBlank.apiAcc",
                "NotBlank.java.lang.String",
                "NotBlank"
            ],
            "arguments": [
                {
                    "codes": [
                        "signAddRequest.apiAcc",
                        "apiAcc"
                    ],
                    "arguments": null,
                    "defaultMessage": "apiAcc",
                    "code": "apiAcc"
                }
            ],
            "defaultMessage": "apiAcc不能为空!",
            "objectName": "signAddRequest",
            "field": "apiAcc",
            "rejectedValue": null,
            "bindingFailure": false,
            "code": "NotBlank"
        },
        { /* ... */ }
    ],
    "message": "Validation failed for object='signAddRequest'. Error count: 4",
    "path": "/api/xxx"
}
```

### Iris

#### 最简单的错误

```
直接以纯文本方式返回错误描述
```

#### 格式化

```json
{
    // 同 Spring 的 message
    "detail": "One or more fields failed to be validated",
    "errors": [
        { /* 自定义错误 */ },
        { /* ... */ }
    ],
    "status": 400,
    // 同 Spring 的 error
    "title": "Validation error",
    "type": "http://127.0.0.1:8888/user/validation-errors"
}
```

### 微信支付错误信息

```json
{
    "code": "PARAM_ERROR",
    "message": "参数错误",
    "detail": {
        // 指示错误参数的位置。当错误参数位于请求body的JSON时，填写指向参数的[JSON Pointer](https://datatracker.ietf.org/doc/html/rfc6901) 。当错误参数位于请求的url或者querystring时，填写参数的变量名。
        "field": "/amount/currency",
        // 错误的值
        "value": "XYZ",
        // 具体错误原因
        "issue": "Currency code is invalid",
        "location" :"body | path | query | header"
    }
}
```

## 最终结果参考

```json
{
    // 实际上该字段是可以不要的,因为http协议肯定能拿到状态码。而把它加到响应中应该是为了更方便，比如：1.方便客户端获取 2.服务器打印响应日志时也方便查看
    // 这应该也是Spring默认JSON格式的错误响应带有该字段的原因。
    "status": 400,
    // [可选]少数code与status一一对应,但有时候可能需要进行扩展,比如 403 未授权时可以通过code区分具体缺少哪些授权
    "code": "PARAM_ERROR",
    // 主要是给前端时可能用得到,比如公共的错误提示时
    "error": "参数错误",
    // 该值和status存在的目的相似,但主要是给人看。此外,由于Nginx层面可能会更改转发路径而导致用户看到的和后端服务的path不一致，所以这里填写后端服务路径会更容易排查问题
    "path": "/api/xxx",
    // 该值和path存在的目的相似,主要是给人看. 注意,这里建议格式化为常见的格式,并且时区采用东八区
    "timestamp": "2022-03-22 01:03:40",
    // [可选]具体的错误信息,主要是给人看. 这里格式不固定是因为对于不同类型的错误，详细程度可能不一样，比如参数错误时可以精确列出具体错误的地方并加以说明，而请求过多被限流等情况一般只需要一句详细描述即可。
    "detail": {
        "field": "/amount/currency",
        // 拒绝值,即请求参数中的原始值
        "rejectedValue": "XYZ",
        "issue": "Currency code is invalid",
        "location" :"body | path | query | header"
    }
}
```

## 参考文档

1. [[简书]restful风格API](https://www.jianshu.com/p/73d2415956bd)
2. [微信支付-开发者文档](https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay2_0.shtml#part-6)
3. [API 设计中的最佳实践](https://swagger.io/resources/articles/best-practices-in-api-design/)



