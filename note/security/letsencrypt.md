---
title: 从 Let’s Encrypt 申请免费证书
author: 'Laeni'
tags: SSL, HTTPS, 证书, 安全
date: '2022-11-29'
updated: '2022-11-29'
---

Let’s Encrypt 是一个证书颁发机构（CA），对外提供[ACME](https://www.rfc-editor.org/rfc/rfc8555)协议的API，可以通过该API申请证书。而可以与ACME协议的API交互的客户端工具有很多，官方列举了一些[比较好的实现](https://letsencrypt.org/zh-cn/docs/client-options/)。这里我们选用其中的[lego](https://github.com/go-acme/lego)工具使用，选它的一个重要原因是该工具使用Go编写，直接到[项目的releases页](https://github.com/go-acme/lego/releases)下载可执行二进制即可使用。

除非使用命令行`--path`标志单独说明，否则`lego`将在*当前工作目录中*查找`.lego`命名的目录。 如果您运行`cd /dir/a && lego ... run`，`lego`将创建一个`/dir/a/.lego`目录，它将在其中保存帐户注册和证书文件。 如果您稍后尝试续订证书`cd /dir/b && lego ... renew`，`lego`可能会产生错误（因为在新目录下无法找到可以续订的证书）。
