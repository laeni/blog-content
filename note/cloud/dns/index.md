# DNS记录类型

以下为常见的DNS记录类型。

| 类型    | 说明                                         |
| ------- | -------------------------------------------- |
| A       | 将域名指向一个IPV4地址                       |
| CNAME   | 将域名指向另外一个域名                       |
| AAAA    | 将域名指向一个IPV6地址                       |
| NS      | 将子域名指定其他DNS服务器解析                |
| MX      | 将域名指向邮件服务器地址                     |
| SRV     | 记录提供特定的服务的服务器                   |
| TXT     | 文本长度限制512，通常做SPF记录（反垃圾邮件） |
| CAA     | CA证书颁发机构授权校验                       |
| 显性URL | 将域名重定向到另外一个地址                   |
| 隐性URL | 与显性URL类似，但是会隐藏真实目标地址        |

