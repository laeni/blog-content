---
title: 有关非对称加密的相关概念
author: 'Laeni'
tags: 加密, 证书, SLL, TLS
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

Certificate authority（证书颁发机构）。

### SSL

SSL(Secure Socket Layer 安全套接层)是一个加密函数库，目前基本使用v3，后来IETF（The Internet Engineering Task Force - 互联网工程任务组）将其标准化后，命名为TLS。

### TLS

TLS(Transport Layer Security 安全传输层协议)，目前已经发展出三个版本:TLSv1.0、 TLSv1.1、TLSv1.2。目前TLSv1 相当于 SSLv3，通常我们将可不对它们做过细的区分。

## 其他

### RFC

RFC是Internet协议字典，它里包含了所有互联网技术的规范！RFC官方网站: [www.rfc-editor.org](http://www.rfc-editor.org/)

