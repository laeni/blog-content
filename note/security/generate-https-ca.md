---
title: 使用cfssl工具签发https证书
description: 本文采用的方式先使用 cfssl 生成根证书，然后再通过根证书生成实际使用的https证书（中间证书），这样可以通过内置一个根证书来验证该根证书生成的所有中间证书的有效性
author: 'Laeni'
tags: cfssl, https, ssl, 自签发证书, 根证书, 中间证书
date: '2021-05-22'
updated: '2021-05-23'
---

除了某些特殊场景下外，自签发证书（CA）一般不用于生产环境，因为一般情况客户端（如浏览器）并不信任自签发证书，生产中一般需要到权威的证书签发机构购买相应证书使用。但是作为测试或者某些特殊情况下确实需要自签发证书。本文采用[cfssl](https://github.com/cloudflare/cfssl)生成相关证书并做记录，且先生成根证书，再通过根证书签发其他证书，目的是为了学习[cfssl](https://github.com/cloudflare/cfssl)的基本使用以及了解客户端如果基于证书来保证通信的安全性，如果仅仅只是想生成一个用于测试的证书，那可以百度一下[CSR文件在线生成工具](https://csr.chinassl.net/generator-csr.html)等工具可以快速在线生成。

## 客户端怎样验证证书的真伪

https协议是建立在http的基础之上的，即在http的基础了加了非对称加密技术，而http证书实际上包含了非对称加密中生成的公钥。在https通信过程中，客户端需要先使用从服务端获取的证书和服务端协商一个对称加密的密码以用于后续数据传输时加密使用，但是协商之前客户端需要确认服务端给的证书的真伪及有效性。而为达到此种目的一般有两种方法，一种方法是提前将该证书本身添加到自己的白名单（证书库）中来直接验证该证书的真假，这种方式产生一对公私钥即可；另一种方法则是通过提前加白的其他更具一般性的证书来间接验证其他证书，而这种方式则至少产生两对公私钥，一对用于生成其他证书，该证书就是根证书，由根证书生成的子证书一般叫做中间证书，实际颁发的也就是这些中间证书。目前https协议中使用的方式基本都是后者，而根证书则由厂家出厂时就内置在客户端（也可能会通过某种方式定期自动更新），用于验证由根证书颁发的中间证书。

由于是自签发证书，所以为了方便起见，一般都不采用根证书方式，而是直接生成一对就好了，生成之后将生成的证书添加到客户端的证书库就可以正常使用了。但是本文的目的主要目的是学习和测试，所以会选择先生成中间证书的方式来生成https证书。

## 证书生成

### 安装cfssl工具

由于[cfssl](https://github.com/cloudflare/cfssl)是通过golang来写的，所以安装十分方便，最简单的方式是直接去github下载对应系统预先编译好的二进制可执行文件到系统环境变量目录下即可，下载时只需要下载`cfssl`和`cfssljson`即可，`cfssl`用于密匙生成等工作，但是它输出结果是`json`格式的字符串，所以一般使用`cfssljson`将`json`中的证书密匙内容提取到对应的文件中。

### 生成根CA

本质上，根CA与其他正常https证书一样都是公私钥，所以生成方式类似，区别在于根CA只用于验证其他CA，所以不需要参与生成的`hosts`（`hosts`列表中指定的主机名或域名将用于证书的生成，可用于验证，这样就能防止证书伪造的可能，就像用百度的证书放到阿里的网站上验证不通过用的就是这个机制）。

1. 首先创建如下生成根CA需要使用的配置文件`ca.json`。

   ```javascript
   // 注意根据实际情况修改相关属性值并删除所有注释
   {
     "CN": "Ibox Inc", //【一般必须】颁发者/机构名称,后面生成的证书会显示该证书由某某机构颁发
     "key": { //【必须】指明证书类型与强度
       "algo": "rsa",
       "size": 2048
     },
     "ca": { //【必须】证书过期时间，由于是根证书，所以过期时间会稍微长一点
       "expiry": "876000h"
     },
     "names": [ //【可选】主体/颁发者详细信息，理论上该属性及其子属性都是可选的，且一般为英文(没试过中文)
       {
         "C": "CN",         // 国家
         "ST": "GuangDong", // 州/省
         "L": "ShenZhen",   // 市
         "O": "Ibox, Inc.", // 组织
         "OU": "Laeni"      // 机构
       }
     ]
   }
   ```

   可以输出默认配置

   ```bash
   $ cfssl print-defaults csr
   ```

2. 通过配置文件生成证书

   ```bash
   # 注意根据实际情况修改相关属性值并删除所有注释
   $ cfssl gencert \
     -initca ca.json \  # 证书配置文件
     | cfssljson -bare ca # 将生成的内容保持到名为"ca.*"的文件中
   ```

   以上命令会生成`ca.pem`（证书/公钥）、`ca-key.pem`（密匙/私钥）和`ca.csr`（暂无用处，可删除）。

### 通过根CA签发https证书

1. 创建cfssl配置文件`config.json`（理论上可以通过其他方式代理）

   ```javascript
   // 注意根据实际情况修改相关属性值并删除所有注释
   {
     "signing": {
       "profiles": {
         "xxx": { // 该属性名可以自定义，在生成命令中需要指定该值以表明需要使用该配置
           "usages": ["signing", "key encipherment", "server auth", "client auth"], // 【必须】指明该证书可以用于干什么事情（正式签发的证书基本也是这样，唯一多一个“撤销证书”功能）
           "expiry": "87600h"  // 【必须】同根CA一样，这里指定该证书的过期时间，且该时间一般比根CA短。
         }
       }
     }
   }
   ```

   可以输出默认配置

   ```bash
   $ cfssl print-defaults config
   ```

2. 创建生成域名证书对应的配置文件`laeni.cn.json`(基本同根CA配置文件)

   ```javascript
   // 注意根据实际情况修改相关属性值并删除所有注释
   {
     "CN": "laeni.cn", //【一般必须】这里一般为要颁发的域名(约定),但是也可以为其他的,作用仅仅是查看时显示的名称而已
     "hosts": [ // 这里填写该证书能够使用的域名或ip,且可以精确到协议和端口（暂时没研究过泛域名该怎么填写）
       "localhost",
       "LAENI",
       "127.0.0.1",
       "ip6-localnet",
       "laeni.cn"
     ],
     "key": { //【必须】同根CA，指定证书类型与强度（尚未测试这里如果与根CA不一致会不会成功）
       "algo": "rsa",
       "size": 2048
     },
     "names": [ //【可选】同根CA，主体/颁发者详细信息
       {
         "C": "CN",
         "ST": "GuangDong",
         "L": "ShenZhen",
         "O": "Ibox, Inc.",
         "OU": "Laeni"
       }
     ]
   }
   ```

3. 通过配置文件生成证书

   ```bash
   # 注意根据实际情况修改相关属性值并删除所有注释
   $ cfssl gencert \
     -ca=ca.pem \ # 前面生成的根CA的证书/公钥
     -ca-key=ca-key.pem \ # 前生成的根CA的密匙/私钥
     -config=config.json \ # cfssl 配置文件
     -profile=xxx \ # 这里的"xxx"即为 config.json 中配置的字段名
     laeni.cn.json \ # 证书配置文件
     | cfssljson -bare laeni.cn # 将生成的内容保持到名为"laeni.cn.*"的文件中
   ```

   以上命令会生成`laeni.cn.pem`（证书/公钥）、`laeni.cn-key.pem`（密匙/私钥）和`ca.csr`（暂无用处，可删除）。

4. 制作绑定证书（可选）

   到目前为止已经生成了两个证书，但通常的做法是将颁发给指定网站的证书与根证书绑定在一起，绑定的方式很简单，实际上就是简单地将根证书（这里为`ca.pem`）的内容全部复制到网站证书（这里为`laeni.cn.pem`）的后面即可，实际上不绑定也是可以使用的，只不过绑定之后感觉更符合约定。

## Nginx配置

配置参考如下

```
server {
    listen       443 ssl;
    listen  [::]:443 ssl;
    server_name  localhost;

	# 证书/公钥
    ssl_certificate /etc/nginx/ssl/laeni.cn.pem;
    # 密匙/私钥
    ssl_certificate_key /etc/nginx/ssl/laeni.cn-key.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    error_page   500 502 503 504  /50x.html;

    location / {
    	root   /usr/share/nginx/html;
    	index  index.html index.htm;
    }
}
```

## 添加根CA到客户端证书列表

由于这里生成了两个证书，所有添加哪一个都能实现验证的效果，但是一般都是添加根CA，这样的话如果是用根CA签发新的证书也可以照常使用而不用再添加一次，这就是使用根CA的意义所在。

但是不同的客户端添加方式不同，理论上讲直接添加到系统证书列表即可，但是实际上有些应用可能还需要单独添加，比如360浏览器和火狐浏览器等有自己的根证书库（没试过直接添加到系统库能否生效），再比如编程使用时可能也需要针对特定的语言使用特定的方式进行添加。

---

以上内容为本人的理解并结合测试得到的结果，如有错误敬请谅解并欢迎指正（在评论未支持前可通过[关于博主](../../../about/self.html)中的[邮箱](mailto:m@laeni.cn)进行反馈）。