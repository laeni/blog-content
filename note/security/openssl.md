---
title: openssl工具帮助文档
author: 'Laeni'
tags: openssl, https, ssl
date: '2022-12-01'
updated: '2022-12-06'
---

# rsa私钥生成

```sh
$ openssl genrsa -out private.pem 2048
```

# 从私钥中提取公钥

```sh
$ openssl rsa \
  -pubout `# 指示输出文件为公钥类型，不指定则默认为私钥，如果不指定常用于私钥类型转换`\
  -in private.pem `# 已经存在的私钥`\
  -inform pem `# 输入的私钥的类型. 可选类型为：pem/der`\
  -out public_key.der `# 输出文件名（公钥）`\
  -outform der `# 需要生成的公钥类型. 可选类型为：pem/der`
```

# 类型转换

一般常用的类型为`pem`，所以无需进行转换，但是java中需要进行转换，不过需要注意，即使在java中，**公钥一般不用进行转换**。类型转换和提取公钥类似，只是不加`-pubout`标记即可。

## `pem`和`der`格式的区别

`pem`格式为`der`编码为 Base64 字符串（为了方便人阅读，需要每隔`64`字符就进行换行）后在前面和后面加上固定的说明后得到，比如私钥为`-----BEGIN RSA PRIVATE KEY-----`和`-----END RSA PRIVATE KEY-----`，公钥为`-----BEGIN PUBLIC KEY-----`和`-----END PUBLIC KEY-----`。

## `pem` -> `der`

```sh
$ openssl rsa \
  -in private.pem `# 已经存在的私钥`\
  -inform pem `# 输入的私钥的类型. 可选类型为：pem/der`\
  -out private.der `# 输出文件名`\
  -outform der `# 需要生成的公钥类型. 可选类型为：pem/der`
```

## `der` -> `pk8`

```sh
$ openssl pkcs8 -topk8 -nocrypt \
  -in private.der `# 已经存在的私钥`\
  -inform der `# 输入的私钥的类型. 可选类型为：pem/der`\
  -out private.der `# 输出文件名`\
  -outform der `# 需要生成的公钥类型. 可选类型为：pem/der`
```

除上述示例外，[此处](/note/security/cfssl/#openssl)还有部分相关示例。
