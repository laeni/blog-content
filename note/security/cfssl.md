---
title: cfssl工具帮助文档
author: 'Laeni'
tags: cfssl, https, ssl, 自签名证书, 根证书, 中间证书, 证书链
date: '2022-01-03'
updated: '2022-11-05'
---

[CFSSL](https://github.com/cloudflare/cfssl) 是 CloudFlare 的 PKI/TLS 瑞士军刀。它既是一个命令行工具，也是一个用于签署、验证和捆绑 TLS 证书的 HTTP API 服务器。

## cfssl使用示例

> 由于`cfssl`只返回`json`内容，一般需要通过`cfssljson`从返回的`json`中提取对内容并写入到文件中，所以后续示例一般都需要结合`cfssljson`来使用。假如将`cfssl`输出的内容保存到`ca-out.txt`文件中，那么删除`ca-out.txt`中多余的内容（仅保留最后的Json）后，使用`cat ca-out.txt | cfssljson -bare ca`将Json中的内容提取到文件中，生成的文件分别为：`CA证书（ca.pem）`、`私钥（ca-key.pem）`和`证书请求（ca.csr）`

### 配置cfssl

配置是可选的，如果不配置则使用默认配置。

查看默认配置

```shell
$ cfssl print-defaults config # 这里的config为参数类型，使用 cfssl print-defaults list 查看支持的类型
{
    "signing": {
        "default": {
            "expiry": "168h"
        },
        "profiles": {
            "www": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            }
        }
    }
}
```

根据自身需要修改配置

```sh
$ cat <<EOF | tee cfssl-config.json
{
    "signing": {
        "default": {
            "expiry": "8760h"
        },
        "profiles": {
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

> **usages**可能的值：
>
> - signing：表示该证书可用于签发其它证书。
> - key encipherment：表示该证书可以用于加密。
> - server auth：表示该证书可用于服务器认证（比如https证书就是服务端证书）。
> - client auth：表示该证书可以用于客户端认证。

### CA证书

这里我们生成的CA证书由于没有其他颁发机构给我们签名，所以它只能自签名，这种证书又叫根证书或 Root CA。

#### 根据配置生成CA

##### cfssl

1. 创建`CSRJSON`
    ```shell
    $ cat > ./global_root_ca-csr.json <<EOF
    {
        "CN": "Laeni Global Root CA",
        "key": {
            "algo": "rsa",
            "size": 2048
        },
        "ca": {
        	"expiry": "87600h"
        },
        "names": [{
            "C": "CN",
            "ST": "YunNan",
            "L": "KunMing",
            "O": "Laeni Inc",
            "OU": "www.laeni.cn"
        }]
    }
    EOF
    ```
    
2. 根据`CSRJSON`生成私钥和证书请求（可忽略）

    ```shell
    # 如果要生成证书请求，需要将 global_root_ca-csr.json 中的 ca 字段删除
    $ cfssl genkey global_root_ca-csr.json | cfssljson -bare global_root_ca
    ```

    > 由于**证书请求**（包含公钥和申请人基本信息等）是证书颁发机构给我们颁发证书时用的，而这里我们生成的是自签名（不需要别人给我们签名）证书，所以在这里生成的证书请求是没用的，故而这里仅仅只是记录下生成方式。

3. 生成证书

    ```shell
    $ cfssl gencert -initca global_root_ca-csr.json | cfssljson -bare global_root_ca
    ```

4. 重命名为常见后缀（不需要做特殊转换）

    ```sh
    $ mv global_root_ca-key.pem global_root_ca.key
    $ mv global_root_ca.pem global_root_ca.crt
    ```

##### openssl

1. 生成私钥
    ```shell
    $ openssl genrsa -out global_root_ca.key 2048
    ```
    
2. 生成证书请求（可忽略）

    ```shell
    $ openssl req -days 90 -x509 -key global_root_ca.key -out global_root_ca.csr
    ```
    
    > 同`cfssl`一样，这里仅仅只是记录下生成方式。
    
3. 直接根据私钥生成证书

    ```shell
    # 由于openssl没有指定类似上面的 csr.json 配置内容，所以生成时需要根据提示输入相关的主题信息
    $ openssl req -new -x509 -days 9131 -key global_root_ca.key -extensions v3_ca -out global_root_ca.crt
    ```

4. 如有需要，还可以生成`p12`格式证书

    ```shell
    $ openssl pkcs12 -export -inkey global_root_ca.key -in global_root_ca.crt -out global_root_ca.pfx
    ```

> `cfssl`和`openssl`生成的文件作用完全相同，仅仅只是文件后缀名不一致（可以直接重命名为一致）。

---

#### 根据配置和已有密钥生成CA

由于已经明确提供了密钥，所以不会再生成新的密钥。

使用`cfssl gencert -initca -ca-key <key.pem> CSRJSON`生成证书以及证书请求：

```sh
$ cfssl gencert -initca -ca-key global_root_ca.key global_root_ca-csr.json | cfssljson -bare 2-global_root_ca
```

输出文件：`2-global_root_ca.pem（证书）`、`2-global_root_ca.csr（证书请求）`

#### 根据原CA证书和密钥重新生成新CA

由于已经有证书了，所以不会再生成证书请求，因为证书请求可以看作生成证书的配置，而从证书中可以得到这些配置。

使用`cfssl gencert -renewca -ca <cert.pem> -ca-key <key.pem>`重新生成证书：

```sh
$ cfssl gencert -renewca -ca global_root_ca.crt -ca-key global_root_ca.key | cfssljson -bare 3-global_root_ca
```

输出文件：`3-global_root_ca.pem（证书）`

### 签名

```sh
$ cfssl sign \
  -ca <ca.pem / ca.crt> `# 用于签名的证书颁发机构证书（CA）` -ca-key <ca-key.pem / ca.key> `# CA对应的私钥`\
  -config <cfssl-config.json> `# cfssl工具配置文件` -profile ca `# 指定要用配置文件中的哪个 profile`\
  <ca.csr> `# 证书请求文件（由申请人提供，里面包含公钥/申请人基本信息等）`\
  | cfssljson -bare <xx> `# 将签名后生成的证书写入'xx'开头的文件（xx.pem，可直接重命名为'.crt'后缀）`
```

## cfssl HELP

使用 CFSSL 包的规范命令行工具。

该`cfssl`命令行工具需要一个命令来指定应该

```
sign             签署证书 | signs a certificate
bundle           构建证书包 | build a certificate bundlesss
genkey           生成私钥和证书请求 | generate a private key and a certificate request
gencert          生成私钥和证书 | generate a private key and a certificate
version          打印出当前版本 | prints out the current version
selfsign         生成自签名证书 | generates a self-signed certificate

sign      签名一个客户端证书，通过给定的CA和CA密钥，和主机名
bundle: 创建包含客户端证书的证书包
genkey: 生成一个key(私钥)和CSR(证书签名请求)
scan: 扫描主机问题
revoke: 吊销证书
certinfo: 输出给定证书的证书信息， 跟cfssl-certinfo 工具作用一样
gencrl: 生成新的证书吊销列表
selfsign: 生成一个新的自签名密钥和 签名证书
print-defaults 打印默认配置，这个默认配置可以用作模板
serve          启动一个HTTP API服务
gencert        生成新的key(密钥)和签名证书
 -ca      指明ca的证书
 -ca-key  指明ca的私钥文件
 -config  指明请求证书的json文件
 -profile 与-config中的profile对应，是指根据config中的profile段来生成证书的相关信息
ocspdump        从cert db 中的所有 OCSP 响应中生成一系列连贯的 OCSP 响应，供 ocspserve 使用
ocspsign  为给定的CA、Cert和状态签署OCSP响应。返回一个base64编码的OCSP响应
info      获取有关远程签名者的信息
ocsprefresh: 用所有已知未过期证书的新OCSP响应刷新ocsp_responses表。
ocspserve: 设置一个HTTP服务器，处理来自文件或直接来自数据库的OCSP请求（见RFC 5019）。
```

使用`cfssl [command] -help`以了解更多的命令。该`version`命令不带任何参数。

### 证书生成流程

1. 按照需求和规则创建`证书请求配置文件`，一般命名为`ca-csr.json`。
3. 生成一对公私钥，其中公钥是从私钥中提取出来的，其中私钥一般命名为`ca-key.pem`或`ca.key`，公钥一般命名为`ca.pub`（`cfssl`工具会省略公钥文件直接生成证书）。
3. 使用`公钥（可能不需要）`、`证书请求配置文件`生成`证书请求`文件，一般命名为`ca.csr`。
4. 使用`公钥`和`证书请求配置文件`生成证书，这一步一般由证书颁发机构完成，所以生成的证书中包含了第三步中生成的公钥以及证书颁发机构密钥生成的签名，并以签名来判断证书的合法性。

> 以上四步中，第一步并非标准，而它只是`cfssl`工具的配置文件而已，真正的标准是从第二步开始的。

### 常用的配置文件

#### CSRJSON - 请求JSON文件

默认csr`cfssl print-defaults csr`

```json
{
    "CN": "example.net",
    "hosts": [
        "example.net",
        "www.example.net"
    ],
    "key": {
        "algo": "ecdsa",
        "size": 256
    },
    "names": [{
        "C": "US",
        "ST": "CA",
        "L": "San Francisco"
    }]
}
```

证书请求配置文件，用于生成证书请求（`.csr`）。

```json
// 可能的文件名: ca.json | ca-csr.json
{
    "CN": "Ibox Inc", //【一般必须】常用名/身份/认证机构/颁发者，后面生成的证书会显示该证书由某某机构颁发，该名称一般不能随意 填写，比如 etcd 会将该值当作用户名
    "key": { //【必须】指明证书类型与强度
        "algo": "rsa",
        "size": 2048
    },
    "ca": {
        "expiry": "876000h" //【必须】证书过期时间，一般根证书的过期时间会稍微长一点
    },
    "names": [{ //【可选】主体/颁发者详细信息，理论上该属性及其子属性都是可选的，且一般为英文(没试过中文)
        "C": "CN",         // 国家
        "ST": "GuangDong", // 州/省
        "L": "ShenZhen",   // 位置（一般填写“市”）
        "O": "Laeni, Inc.", // 组织
        "OU": "Laeni"      // 单位/机构
    }]
}
```

#### SUBJECT - 主题配置文件

该文件一般是不需要的，因为该信息一般在`csr.json`中配置。

```json
{
    "CN": "example.com",
    "names": [{
        "C":  "US",
        "ST": "California",
        "L":  "San Francisco",
        "O":  "Internet Widgets, Inc.",
        "OU": "WWW"
    }]
}
```

#### cfssl 配置文件

默认配置`cfssl print-defaults config`

```json
{
    "signing": {
        "default": {
            "expiry": "168h"
        },
        "profiles": {
            "www": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            }
        }
    }
}
```

实例

```json
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        // 可以定义多个 profiles，分别指定不同的过期时间、使用场景等参数；后续在签名证书时使用某个 profile；
        "profiles": {
            // 此实例只有一个kubernetes模板。
            "kubernetes": {
                "usages": [
                    "signing",          // 表示该证书可用于签名其它证书；生成的 ca.pem 证书中CA=TRUE
                    "key encipherment",
                    "server auth",      // 表示client可以用该 CA 对server提供的证书进行验证；
                    "client auth"       // 表示server可以用该CA对client提供的证书进行验证；
                ],
                "expiry": "87600h"
            }
        }
    }
}
```

### Gencerting - 生成证书

```sh
$ cfssl gencert --help
        cfssl gencert -- generate a new key and signed certificate

Usage of gencert:
    从 CSR（CSRJSON） 配置文件生成新密钥和证书 | Generate a new key and cert from CSR:
        cfssl gencert -initca CSRJSON
        cfssl gencert -ca cert -ca-key key [-config config] [-profile profile] [-hostname hostname] CSRJSON
        cfssl gencert -remote remote_host [-config config] [-profile profile] [-label label] [-hostname hostname] CSRJSON

Arguments:
        CSRJSON:    JSON file containing the request, use '-' for reading JSON from stdin

Flags:
  -initca=false: 初始化新 CA | initialise new CA
  -remote="": remote CFSSL server
  -ca="": CA（旧证书），用于签署新证书 -- 接受 '[file:]fname' or 'env:varname' | CA used to sign the new certificate -- accepts '[file:]fname' or 'env:varname'
  -ca-key="": CA 私钥 -- 接受 '[file:]fname' or 'env:varname' | CA private key -- accepts '[file:]fname' or 'env:varname'
  -config="": path to configuration file
  -cn="": 证书通用名 | certificate common name (CN)
  -hostname="": 证书的主机名，可以是逗号分隔的主机名列表 | Hostname for the cert, could be a comma-separated hostname list
  -profile="": 要使用的签名配置文件 | signing profile to use
  -label="": key label to use in remote CFSSL server
  -loglevel=1: Log level (0 = DEBUG, 5 = FATAL)
```

#### e.g

点击下载示例使用的所有文件 TODO

1. 使用配置直接生成CA证书、证书请求、密钥

   1. 创建CA证书 `CSRJSON`，见`1-ca-csr.json`
   2. 使用`cfssl gencert -initca 1-ca-csr.json`生成`CA证书`、`私钥`和`证书请求`，输出见`2-ca-out.txt`
   3. 删除`2-ca-out.txt`中多余的内容（仅保留最后的Json），然后使用`cat 2-ca-out.txt | cfssljson -bare 3-ca`将Json中的内容提取到文件中，分别见：`CA证书（3-ca.pem）`、`私钥（3-ca-key.pem）`和`证书请求（3-ca.csr）`

   > 由于演示目的，所以分了三步，一般直接使用`cfssl gencert -initca 1-ca-csr.json | cfssljson -bare 3-ca`生成最终的文件。

2. 使用配置和密钥生成CA证书、证书请求

   由于已经明确提供了密钥，所以不会再生成新的密钥。

   使用`cfssl gencert -initca -ca-key key CSRJSON`生成证书以及证书请求：

   输出见：`证书（4-ca.pem）`、`证书请求（4-ca.csr）`

3. 使用原CA证书和密钥重新生成新CA证书

   由于已经有证书了，所以不会再生成证书请求，因为证书请求可以看作生成证书的配置，而从证书中可以得到这些配置。

   使用`cfssl gencert -renewca -ca cert -ca-key key`重新生成证书：

   ```sh
   $ cfssl gencert -renewca -ca 3-ca.pem -ca-key 3-ca-key.pem | cfssljson -bare 5-ca
   ```

   输出见：`证书（5-ca.pem）`

### 签名 | Signing

```sh
$ cfssl sign --help
        cfssl sign -- 通过给定的CA的证书和私钥使用主机名签署客户端证书 | signs a client cert with a host name by a given CA and CA key

Usage of sign:
        cfssl sign -ca cert -ca-key key [mutual-tls-cert cert] [mutual-tls-key key] [-config config] [-profile profile] [-hostname hostname] [-db-config db-config] CSR [SUBJECT]
        cfssl sign -remote remote_host [mutual-tls-cert cert] [mutual-tls-key key] [-config config] [-profile profile] [-label label] [-hostname hostname] CSR [SUBJECT]

Arguments:
        CSR:        用于证书请求的 PEM 文件，使用“-”从标准输入读取 PEM。| PEM file for certificate request, use '-' for reading PEM from stdin.

Note: CSR 也可以通过'-csr'标志值提供；标志值将优先于参数。| CSR can also be supplied via flag values; flag value will take precedence over the argument.

Flags:
  -hostname="": 证书的主机名，可以是逗号分隔的主机名列表，它覆盖证书 SAN 扩展中的 DNS 名称和 IP 地址 | Hostname for the cert, could be a comma-separated hostname list
  -csr="": 新公钥的证书签名请求文件 | Certificate signature request file for new public key
  -ca="": CA机构的证书 用于签署新证书——接受'[file:]fname'或'env:varname' | CA used to sign the new certificate -- accepts '[file:]fname' or 'env:varname'
  -ca-key="": CA机构的证书的私钥 | A private key -- accepts '[file:]fname' or 'env:varname'
  -config="": 配置文件路径 | path to configuration file
  -profile="": 要使用的签名配置文件 | signing profile to use
  -label="": 用于远程 CFSSL 服务器的密钥标签 | key label to use in remote CFSSL server
  -remote="": 远程CFSSL服务器 | remote CFSSL server
  -db-config="": 证书数据库配置文件 | certificate db configuration file
  -loglevel=1: Log level (0 = DEBUG, 5 = FATAL)
```

主题是一个可选文件，其中包含用于证书的主题信息，而不是 CSR 中的主题信息，即其中的信息会覆盖 CSR 中的信息。它应该是一个 JSON 文件，如下所示：

```json
{
    "CN": "example.com",
    "names": [{
        "C":  "US",
        "L":  "San Francisco",
        "O":  "Internet Widgets, Inc.",
        "OU": "WWW",
        "ST": "California"
    }]
}
```

### 捆绑 | Bundling

```
cfssl bundle [-ca-bundle bundle] [-int-bundle bundle] \
             [-metadata metadata_file] [-flavor bundle_flavor] \
             -cert certificate_file [-key key_file]
```

这些捆绑包用于根证书池和中间证书池。另外，平台元数据是通过`-metadata`指定的。捆绑文件、元数据文件（和辅助文件）可以在以下位置找到：| The bundles are used for the root and intermediate certificate pools. In addition, platform metadata is specified through `-metadata`. The bundle files, metadata file (and auxiliary files) can be found at:

```
    https://github.com/cloudflare/cfssl_trust
```

分别通过 `-cert` 和 `-key` 指定 PEM 编码的客户端证书和密钥。如果指定了密钥，则将使用密钥构建和验证捆绑包。否则，捆绑包将在没有私钥的情况下构建。代替文件路径，使用 `-` 从标准输入读取证书 PEM。证书文件应包含（部分）证书包也是可以接受的。| Specify PEM-encoded client certificate and key through `-cert` and `-key` respectively. If key is specified, the bundle will be built and verified with the key. Otherwise the bundle will be built without a private key. Instead of file path, use `-` for reading certificate PEM from stdin. It is also acceptable that the certificate file should contain a (partial) certificate bundle.

通过 `-flavor` 指定捆绑风味。有三种风格：“optimal-最佳”用于生成最短链和最先进的加密算法的捆绑，“ubiquitous-无处不在”用于生成在不同浏览器和操作系统平台上最广泛接受的捆绑，以及“force-强制”寻找可接受的捆绑与输入证书文件的内容相同。| Specify bundling flavor through `-flavor`. There are three flavors: `optimal` to generate a bundle of shortest chain and most advanced cryptographic algorithms, `ubiquitous` to generate a bundle of most widely acceptance across different browsers and OS platforms, and `force` to find an acceptable bundle which is identical to the content of the input certificate file.

或者，可以直接从域中提取客户端证书。也可以通过 `-ip` 连接到远程地址。| Alternatively, the client certificate can be pulled directly from a domain. It is also possible to connect to the remote address through `-ip`.

```
cfssl bundle [-ca-bundle bundle] [-int-bundle bundle] \
             [-metadata metadata_file] [-flavor bundle_flavor] \
             -domain domain_name [-ip ip_address]
```

捆绑输出形式应遵循示例：| The bundle output form should follow the example:

```json
{
    "bundle": "CERT_BUNDLE_IN_PEM",
    "crt": "LEAF_CERT_IN_PEM",
    "crl_support": true,
    "expires": "2015-12-31T23:59:59Z",
    "hostnames": ["example.com"],
    "issuer": "ISSUER CERT SUBJECT",
    "key": "KEY_IN_PEM",
    "key_size": 2048,
    "key_type": "2048-bit RSA",
    "ocsp": ["http://ocsp.example-ca.com"],
    "ocsp_support": true,
    "root": "ROOT_CA_CERT_IN_PEM",
    "signature": "SHA1WithRSA",
    "subject": "LEAF CERT SUBJECT",
    "status": {
        "rebundled": false,
        "expiring_SKIs": [],
        "untrusted_root_stores": [],
        "messages": [],
        "code": 0
    }
}
```

### 生成证书签名请求和私钥

```sh
$ cfssl genkey csr.json | cfssljson -bare xxx
```

要生成私钥和相应的证书请求，请将密钥请求指定为 JSON 文件。 该文件应遵循以下格式：| To generate a private key and corresponding certificate request, specify the key request as a JSON file. This file should follow the form:

```json
{
    "hosts": [
        "example.com",
        "www.example.com",
        "https://www.example.com",
        "jdoe@example.com",
        "127.0.0.1"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C":  "US",
        "L":  "San Francisco",
        "O":  "Internet Widgets, Inc.",
        "OU": "WWW",
        "ST": "California"
    }]
}
```

### 生成自签名根 CA 证书和私钥 | Generating self-signed root CA certificate and private key

```sh
$ cfssl genkey -initca csr.json | cfssljson -bare ca
```

要生成自签名根 CA 证书，请将密钥请求指定为与“genkey”格式相同的 JSON 文件。 输出中将出现三个 PEM 编码的实体：私钥、csr 和自签名证书。 | To generate a self-signed root CA certificate, specify the key request as a JSON file in the same format as in 'genkey'. Three PEM-encoded entities will appear in the output: the private key, the csr, and the self-signed certificate.

### 生成远程颁发的证书和私钥。| Generating a remote-issued certificate and private key.

```sh
$ cfssl gencert -remote=remote_server [-hostname=comma,separated,hostnames] csr.json
```

This calls `genkey` but has a remote CFSSL server sign and issue the certificate. You may use `-hostname` to override certificate SANs.

### 生成本地颁发的证书和私钥。| Generating a local-issued certificate and private key.

```sh
$ cfssl gencert -ca cert -ca-key key [-hostname=comma,separated,hostnames] csr.json
```

这会通过 JSON 请求从本地 CA 生成并颁发证书和私钥。 您可以使用 `-hostname` 覆盖证书 SANs。| This generates and issues a certificate and private key from a local CA via a JSON request. You may use `-hostname` to override certificate SANs.

### 使用新颁发的证书更新 OCSP 响应文件 | Updating an OCSP responses file with a newly issued certificate

```sh
$ cfssl ocspsign -ca cert -responder key -responder-key key -cert cert \
 | cfssljson -bare -stdout >> responses
```

This will generate an OCSP response for the `cert` and add it to the `responses` file. You can then pass `responses` to `ocspserve` to start an OCSP server.

### ---

### 启动 API 服务器 | Starting the API Server

CFSSL comes with an HTTP-based API server; the endpoints are documented in `doc/api/intro.txt`. The server is started with the `serve` command:

```
cfssl serve [-address address] [-ca cert] [-ca-bundle bundle] \
            [-ca-key key] [-int-bundle bundle] [-int-dir dir] [-port port] \
            [-metadata file] [-remote remote_host] [-config config] \
            [-responder cert] [-responder-key key] [-db-config db-config]
```

Address and port default to "127.0.0.1:8888". The `-ca` and `-ca-key` arguments should be the PEM-encoded certificate and private key to use for signing; by default, they are `ca.pem` and `ca_key.pem`. The `-ca-bundle` and `-int-bundle` should be the certificate bundles used for the root and intermediate certificate pools, respectively. These default to `ca-bundle.crt` and `int-bundle.crt` respectively. If the `-remote` option is specified, all signature operations will be forwarded to the remote CFSSL.

`-int-dir` specifies an intermediates directory. `-metadata` is a file for root certificate presence. The content of the file is a json dictionary (k,v) such that each key k is an SHA-1 digest of a root certificate while value v is a list of key store filenames. `-config` specifies a path to a configuration file. `-responder` and `-responder-key` are the certificate and the private key for the OCSP responder, respectively.

The amount of logging can be controlled with the `-loglevel` option. This comes *after* the serve command:

```
cfssl serve -loglevel 2
```

The levels are:

- 0 - DEBUG
- 1 - INFO (this is the default level)
- 2 - WARNING
- 3 - ERROR
- 4 - CRITICAL

### 多根CA | The multirootca

`cfssl` 程序可以充当在线证书颁发机构，但它只使用一个密钥。 如果需要多个签名密钥，可以使用“multirootca”程序。 它只提供 `sign`、`authsign` 和 `info` 端点。 该文档包含有关配置和运行 CA 的说明。 | The `cfssl` program can act as an online certificate authority, but it only uses a single key. If multiple signing keys are needed, the `multirootca` program can be used. It only provides the `sign`, `authsign` and `info` endpoints. The documentation contains instructions for configuring and running the CA.

### mkbundle 工具 | The mkbundle Utility

`mkbundle` is used to build the root and intermediate bundles used in verifying certificates. It can be installed with

```
go get github.com/cloudflare/cfssl/cmd/mkbundle
```

It takes a collection of certificates, checks for CRL revocation (OCSP support is planned for the next release) and expired certificates, and bundles them into one file. It takes directories of certificates and certificate files (which may contain multiple certificates). For example, if the directory `intermediates` contains a number of intermediate certificates:

```
mkbundle -f int-bundle.crt intermediates
```

will check those certificates and combine valid certificates into a single `int-bundle.crt` file.

The `-f` flag specifies an output name; `-loglevel` specifies the verbosity of the logging (using the same loglevels as above), and `-nw` controls the number of revocation-checking workers.

### cfssljson 工具 | The cfssljson Utility

Most of the output from `cfssl` is in JSON. The `cfssljson` utility can take this output and split it out into separate `key`, `certificate`, `CSR`, and `bundle` files as appropriate. The tool takes a single flag, `-f`, that specifies the input file, and an argument that specifies the base name for the files produced. If the input filename is `-` (which is the default), cfssljson reads from standard input. It maps keys in the JSON file to filenames in the following way:

- if **cert** or **certificate** is specified, **basename.pem** will be produced.
- if **key** or **private_key** is specified, **basename-key.pem** will be produced.
- if **csr** or **certificate_request** is specified, **basename.csr** will be produced.
- if **bundle** is specified, **basename-bundle.pem** will be produced.
- if **ocspResponse** is specified, **basename-response.der** will be produced.

Instead of saving to a file, you can pass `-stdout` to output the encoded contents to standard output.

### 静态构建 | Static Builds

By default, the web assets are accessed from disk, based on their relative locations. If you wish to distribute a single, statically-linked, `cfssl` binary, you’ll want to embed these resources before building. This can by done with the [go.rice](https://github.com/GeertJohan/go.rice) tool.

```
pushd cli/serve && rice embed-go && popd
```

Then building with `go build` will use the embedded resources.

### 附加文件 | Additional Documentation

Additional documentation can be found in the "doc" directory:

- `api/intro.txt`: documents the API endpoints

## multirootca

是一个可以使用多个签名密钥的证书颁发机构服务器。

与`cfssl serve`的区别在于`cfssl`只能使用一个签名密钥，但是`multirootca`可以使用多个。

## mkbundle

用于构建证书池。

## cfssljson

从`cfssl`和`multirootca`程序中获取 JSON 输出， 并将certificates(证书)、keys(密钥)、CSR 和bundles(包)写入磁盘。



