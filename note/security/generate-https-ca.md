---
title: 使用cfssl工具签发包含中间CA的https证书链，并让浏览器和java信任根CA
author: 'Laeni'
tags: cfssl, https, ssl, 自签发证书, 根证书, 根CA, 中间CA, 中间证书
date: '2021-05-22'
updated: '2022-07-08'
---

本文先使用 cfssl 生成根CA证书，然后使用根证书签发中间CA证书（模拟中间颁发机构），再通过中间CA证书签发实际使用的https证书，这样可以通过内置一个根证书来验证该根证书直接或间接生成的所有证书的有效性。以此来模拟目前常见的证书颁发流程，从而更好地理解和学习证书颁发流程及证书验证原理，如果仅仅只是想生成一个用于测试的证书，那可以百度一下[CSR文件在线生成工具](https://csr.chinassl.net/generator-csr.html)等工具可以快速在线生成。

我们这里虽然是通过其他证书来签发https证书，但是根证书任然是不受信任（注意任何根证书都是自签名的），所以本质上还是属于平时说的自签名证书，所以实际对外的网站使用时还是要去权威机构申请。不过这样生成的证书虽然不能用于对外场景，但对内还是很有用的，比如k8s中这么多证书全部都是使用同一个根证书签发的，这对学习k8s会有一定帮助。

## 客户端怎样验证证书的真伪

https协议是建立在http的基础之上的，即在http的基础了加了非对称加密等技术，而证书实际上包含了非对称加密中生成的公钥。在https通信过程中，客户端需要先使用从服务端获取的证书和服务端协商一个对称加密的密码以用于后续数据传输时加密使用，但是协商之前客户端需要确认服务端给的证书的真伪及有效性。而为达到此种目的一般有两种方法，一种方法是提前将该证书本身添加到自己的白名单（证书库）中来直接验证该证书的真假，这种方式产生一对公私钥即可；另一种方法则是通过提前加白的其他更具一般性的证书来间接验证其他证书，而这种方式则至少产生两对公私钥，一对用于生成其他证书，该证书就是根证书，由根证书生成的子证书一般叫做中间证书，实际颁发的也就是这些中间证书。目前https协议中使用的方式基本都是后者，而根证书则由厂家出厂时就内置在客户端（也可能会通过某种方式定期自动更新），用于验证由根证书颁发的中间证书。

## 证书生成

![image-20220705223819578](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/note/security/generate-https-ca/image-20220705223819578.png)

上图所示所示是一般https证书的层级结构，我们最终也将实现这种效果。

### 安装cfssl工具

由于[cfssl](https://github.com/cloudflare/cfssl)是通过golang来写的，所以安装十分方便，最简单的方式是直接去github下载对应系统预先编译好的二进制可执行文件到系统环境变量目录下即可，下载时只需要下载`cfssl`和`cfssljson`即可，`cfssl`用于密匙生成等工作，但是它输出结果是`json`格式的字符串，所以一般使用`cfssljson`将`json`中的证书密匙内容提取到对应的文件中。

安装后先创建一份cfssl常用配置文件以便后面使用，省略的话会使用默认配置的。创建时可以通过`cfssl print-defaults config > default.conf`输出一份默认配置，然后在此基础修改。

```shell
$ cat > cfssl-config.json <<EOF
{
    "signing": {
        "default": {
            "expiry": "8760h"
        },
        // 这里的 profiles 里面可以定义多项，实际使用时通过命令参数指定需要使用的配置即可
        "profiles": {
            "ca": {
                "usages": [
                    "signing",
                    "digital signature",
                    "key encipherment",
                    "cert sign",
                    "crl sign",
                    "server auth",
                    "client auth"
                ],
                "expiry": "43800h",
                "ca_constraint": {
                    "is_ca": true,
                    "max_path_len": 0, 
                    "max_path_len_zero": true
                }
            },
            "www": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "kubernetes": {
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ],
                "expiry": "87600h"
            }
        }
    }
}
EOF
```

### 生成根CA

本质上，根CA与其他正常https证书一样都是公私钥，所以生成方式类似，区别在于根CA只用于生成和验证其他CA，所以不需要参与生成的`hosts`（`hosts`列表中指定的主机名或域名将用于证书的生成，可用于验证，这样就能防止证书伪造的可能，就像用百度的证书放到阿里的网站上验证不通过用的就是这个机制）。此外，由于根CA没有任何人给它签名，所以根CA都是自签名的。

1. 首先创建如下生成根CA需要使用的`CSRJSON`配置文件`ca-csr.json`。

   ```shell
   $ cat > ca-csr.json <<EOF
   {
     "CN": "Laeni Global Root CA", //【一般必须】颁发者/机构名称,后面生成的证书会显示该证书由某某机构颁发
     "key": { //【必须】指明证书类型与强度
       "algo": "rsa",
       "size": 2048
     },
     "ca": { //【必须】证书过期时间，由于是根证书，所以过期时间会稍微长一点
       "expiry": "87600h"
     },
     "names": [
       {
         "C": "CN",           // 国家
         "ST": "YunNan",      // 州/省
         "L": "KunMing",      // 市
         "O": "Laeni Inc",    // 组织
         "OU": "www.laeni.cn" // 机构
       }
     ]
   }
   EOF
   ```

   该文件可以在`cfssl print-defaults csr`命令输出的默认配置基础上进行更改。

2. 通过配置文件生成证书

   ```bash
   $ cfssl gencert -initca ca-csr.json | cfssljson -bare ca # 将生成的内容保持到名为"ca.*"的文件中
   ```
   
   以上命令会生成`ca.pem`（证书/公钥）、`ca-key.pem`（密匙/私钥）和`ca.csr`（证书请求，在生成证书的过程中生存的，现在证书已经生成，所以可删除）。

### 通过根CA签发中间CA

中间CA用于模拟二级颁发机构，比如我们一般办理身份证都是去派出所（公安的派出机构）办理，而不是直接去公安部办理。当然也可以直接去公安部办理，同样的根CA也可以直接签发https证书，只是一般不这样做罢了。

另外，中间CA的生成方式有点特殊，不能直接通过`cfssl gencert`生成，否则后面证书验证的时候会报类型错误，而是需要采用`cfssl sign`（签名）的方式。

1. 创建中间CA的`CSRJSON`配置文件，该文件与根CA配置文件的结构相比少了`ca`属性。

   ```shell
   $ cat > middle-ca-csr.json <<EOF
   {
     "CN": "Laeni CA",
     "key": {
       "algo": "rsa",
       "size": 2048
     },
     "names": [
       {
         "C":  "CN",
         "ST": "YunNan",
         "L":  "KunMing",
         "O":  "Laeni Inc"
       }
     ]
   }
   EOF
   ```

2. 根据`CSRJSON`生成私钥和证书请求文件`.csr`。

   ```shell
   $ cfssl genkey middle-ca-csr.json | cfssljson -bare middle-ca
   ```

   注意证书请求`.csr`文件中同时包含公钥以及`CSRJSON`文件中的信息。

3. 采用根CA对中间CA的公钥（证书请求）进行签名。

   ```shell
   $ cfssl sign \
     -ca ca.pem \ # 该选项为根CA的证书路径
     -ca-key ca-key.pem \ # 该选项为根CA的私钥路径
     -config cfssl-config.json \ # 最开始创建的cfssl工具配置文件
     -profile ca \ # 指定要用配置文件中的哪个 profile
     middle-ca.csr \ # 该文件为第二步生成的证书请求文件
     | cfssljson -bare middle-ca
   ```

   通过签名操作将生成中间CA的证书`middle-ca.pem`。

### 通过中间CA签发https证书

1. 创建域名对应的`CSRJSON`配置文件。

   ```shell
   cat > laeni.cn-csr.json <<EOF
   {
       "CN": "*.laeni.cn", // 该属性一般填写申请的域名，以便可以直观看出证书适用的域名
       // hosts 列表列出该域名进行DNS解析时的名称或者url（一般填写DNS名就行）
       "hosts": [
           "*.laeni.cn"
       ],
       "key": {
           "algo": "rsa",
           "size": 2048
       },
       "names": [
           {
               "C": "CN",
               "ST": "YunNan",
               "L": "KunMing"
           }
       ]
   }
   EOF
   ```

2. 通过`CSRJSON`配置文件生成证书

   ```bash
   $ cfssl gencert \
     -ca middle-ca.pem \ # 前面生成的中间CA证书
     -ca-key middle-ca-key.pem \ # 前生成的中间CA私钥
     -config cfssl-config.json \ # 最开始创建的cfssl工具配置文件
     -profile www \ # 指定要用配置文件中的哪个 profile
     laeni.cn-csr.json \ # 证书请求配置文件
     | cfssljson -bare laeni.cn # 将生成的内容保持到名为"laeni.cn.*"的文件中
   ```
   
   以上命令会生成`laeni.cn.pem`（证书/公钥）、`laeni.cn-key.pem`（密匙/私钥）和`ca.csr`（证书请求）。
   
   ```
   cfssl gencert \
     -ca middle-ca.pem \
     -ca-key middle-ca-key.pem \
     -config ../cfssl-config.json \
     -profile www \
     laeni.cn-csr.json \
     | cfssljson -bare laeni.cn
   ```
   
   
   
4. 制作证书链

   到目前为止，我们已经生成了三个组数据（根CA、中间CA和网站域名对于的证书），但是我们计算机一般只会内置根CA，而中间CA和网站证书是不会内置的，这就没办法直接通过根CA来验证网站证书的真实性，因此需要将他们绑定在一起形成一个证书链，服务器发送网站证书的时和会一起发送中间CA和根CA，这样我们可以通过中间CA来验证网站证书，又可以通过根CA来验证中间CA，而中间CA我们是信任的，所以我们也就间接信任了网站的证书。
   
   而绑定的方式很简单，实际上就是简单地将网站证书和中间证书组合到一个文件中，而根证书可以不用和网站证书绑定在一起，因为根证书将会内置在系统中，绑定后一般以`.crt`格式结尾。
   
   如果用`mkbundle`工具，则多次执行的结果可能不一样（证书顺序可能不一样），所以这里直接简单拼接即可：
   
   ```shell
   $ cat laeni.cn.pem middle-ca.pem > laeni.cn.crt
   ```
   

## Nginx配置

配置参考如下

```
server {
    listen       443 ssl;
    listen  [::]:443 ssl;
    server_name  localhost;

	# 证书/公钥
    ssl_certificate /etc/nginx/ssl/laeni.cn.crt;
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

由于这里生成了三个证书，理论上添加哪一个都能实现验证的效果，但是一般都是添加根CA，这样的话如果是用根CA签发新的证书也可以照常使用而不用再添加一次，这就是使用根CA的意义所在。

但是不同的客户端添加方式不同，理论上讲直接添加到系统证书列表即可，但是实际上有些应用可能还需要单独添加，比如360浏览器和火狐浏览器等有自己的根证书库（没试过直接添加到系统库能否生效），再比如编程使用时可能也需要针对特定的语言使用特定的方式进行添加。
