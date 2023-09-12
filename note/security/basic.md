---
title: 安全相关概念
author: 'Laeni'
tags: 安全, 证书, PKI, SSL, TLS
date: '2022-01-03'
updated: '2022-01-03'
---

## 安全

### PKI

PKI: Public Key Infrastructure: 公钥基础设施。

公钥是从私钥中按照一定的算法提取出来的密码.这个密码很长，且不用于用户名口令的方式使用.所以给它起了个名字叫公钥。

公安部若给全球人民颁发证书，公安部肯定要忙死！所以一般公安部是不直接给任何人颁发证书的，颁发证书的事都由具体的各级的各个派出机构来负责具体的证书颁发。你信任公安部，那么你就必须要信任他的派出机构；CA证书颁发机构也是如此，全球有大概有13家证书颁发机构，他们就是这样做的。所以，这整个信任体系够构成了当今互联网安全通信的框架，而这个框架就是PKI 体系。
【PKI的核心即是证书颁发机构，证书撤销列表 和 它的彼此间的信任关系】
　　CA：certificate authority
　　CRL：证书撤销列表

　　　　![img](https://img2018.cnblogs.com/blog/922925/201905/922925-20190517190259379-1191539986.png)

### CA

Certificate authority（证书颁发机构）。CA证书，即证书颁发机构颁发的证书。

### PEM

PEM(Privacy Enhanced Mail)通常用于数字证书认证机构（Certificate Authorities，CA），扩展名为`.pem`, `.crt`, `.cer`, 和` .key`。内容为`Base64`编码的ASCII码文件，有类似的头尾标记服务器认证证书。Apache和nginx等类似的服务器使用PEM格式证书。

### DER

DER(Distinguished Encoding Rules)与PEM不同之处在于其使用二进制而不是Base64编码的ASCII。扩展名为`.der`，但也经常使用`.cer`用作扩展名，所有类型的认证证书和私钥都可以存储为DER格式，Java是其典型使用平台。

### CSR

CSR(Certificate Signing Request)，它是向CA机构申请数字证书时使用的请求文件。在生成请求文件前，我们需要准备一对对称密钥。私钥信息自己保存，请求中会附上公钥信息以及国家，城市，域名，Email等信息，CSR中还会附上签名信息。当我们准备好CSR文件后就可以提交给CA机构，等待他们给我们签名，签好名后我们会收到`.crt`文件，即证书。

### SSL

SSL(Secure Socket Layer 安全套接层)是一个加密函数库，目前基本使用v3，后来IETF（The Internet Engineering Task Force - 互联网工程任务组）将其标准化后，命名为TLS。

### TLS

TLS(Transport Layer Security 安全传输层协议)，目前已经发展出三个版本:TLSv1.0、TLSv1.1、TLSv1.2。目前TLSv1 相当于 SSLv3，通常我们将可不对它们做过细的区分。

### ECDH

`ECDH = ECC + DH`，一般用于密钥交换。

### ECDSA

`ECDSA = ECC + DSA`，一般用于签名，比如通过**Let’s Encrypt**申请的证书，公钥算法使用的就是*ECDSA*。

## 非对称加密相关算法

| 算法            | 加密/解密 | 数字签名 | 密钥交换 |
| :-------------- | :-------- | :------- | :------- |
| RSA             | ✅         | ✅        | ✅        |
| Diffie-Hellman  | ❌         | ❌        | ✅        |
| DSA             | ❌         | ✅        | ❌        |
| ECC（椭圆曲线） | ✅         | ✅        | ✅        |

## 其他

### RFC

RFC是Internet协议字典，它里包含了所有互联网技术的规范！RFC官方网站: [www.rfc-editor.org](http://www.rfc-editor.org/)

### 常用属性

这些属性大部分用作主题（Subject）属性。

| 名称                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| CN（Common Name）       | 常用名/身份/认证机构/颁发者<br />后面生成的证书会显示该证书由某某机构颁发，该名称一般不能随意 填写。<br />etcd、kube-apiserver 会将该值当作用户名。 |
| O（Organization）       | 组织名称，公司名称<br />kube-apiserver 会将该值当作请求用户所属的组 (Group)。 |
| C（Country）            | 国家                                                         |
| ST（State）             | 州，省                                                       |
| L（Locality）           | 地区，城市（一般填写“市”）                                   |
| OU（Organization Unit） | 组织单位名称，机构，公司部门                                 |

## 参考文档

- [【密码学】非对称加密算法 - ECDH-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1171611)





