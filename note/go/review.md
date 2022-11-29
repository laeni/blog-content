---
title: Golang 知识点复习/易错点记录
author: Laeni
tags: go,golang
date: '2022-10-10'
updated: '2022-10-10'
---

# 通道

1. 向关闭的通道发送请求会导致`panic`

# 其他

1. 类型转换（`Type(expression)`）用于两种不同的类型，在`type A B`中`A`和`B`是两种不同的类型，但是它们是兼容的，所以使用**转换**，而不是类型断言（`expression.(Type)`）；类型断言适用于“它本身就是那种类型“的情况，并不存在转换。

