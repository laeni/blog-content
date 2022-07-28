---
title: API接口设计参考
date: '2021-04-01'
tags: ["API", "接口"]
---

## 参数位置

- 业务无关的不要放在Body中,从设计角度讲放在头中可能会更合适

- 请求和响应最好都要包含唯一标识,目的是为了方便定位问题，位置可以是头中，也可以是消息体中。而消息体的相同率越高，则越需要唯一标识的存在。

## 参数类型

表单、Body(Json/XML...)
