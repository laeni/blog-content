---
title: 从 Let’s Encrypt 申请免费证书并自动续签
author: 'Laeni'
tags: SSL, HTTPS, 证书, 安全
date: '2022-11-29'
updated: '2022-12-03'
---

Let’s Encrypt 是一个证书颁发机构（CA），对外提供[ACME](https://www.rfc-editor.org/rfc/rfc8555)协议的API，可以通过该API申请证书。而可以与ACME协议的API交互的客户端工具有很多，官方列举了一些[比较好的实现](https://letsencrypt.org/zh-cn/docs/client-options/)。这里我们选用其中的[lego](https://github.com/go-acme/lego)工具使用，选它的一个重要原因是该工具使用Go编写，直接到[项目的releases页](https://github.com/go-acme/lego/releases)下载可执行二进制即可使用，并且它不会尝试编辑Web服务器的配置文件，只专注于证书的申请和续期。

在`lego`中，除非使用命令行标志`--path`或环境变量`LEGO_PATH`单独指定，否则`lego`将在*当前用户的工作目录中*查找`.lego`命名的目录（即`~/.lego`）。 

后面假设工作目录为以下目录：

```sh
LEGO_PATH=/mnt/share/archive/cert/www/.lego
```

> 无论使用哪款工具，最后都需要访问 Let’s Encrypt 服务器，而大陆目前是无法正常访问的，所以操作服务器需要代理 Let’s Encrypt 的网络（目前DNS解析正常，只需要代理`172.65.0.0/16`网段的流量即可）。

# 申请证书

证书不是随便都能申请的，*证书颁发机构*需要验证域名所有权后才能给对应域名颁发证书，而验证域名所有权的方法一般为两种：

1. DNS验证。添加一个`TEXT`类型的DNS解析记录，记录值为证书颁发机构生成的随机值。
2. 文件验证。将证书颁发机构生成的随机值写入指定的Web服务器静态文件中，该文件必须能通过申请证书的域名访问到，访问端口必须是`80`或`443`。不过使用该方式时必须确保该服务器能够从境外访问，否则可能无法验证通过。

> 由于工具是自动化的，所以采用DNS方式验证应该是最稳定的，但是必须得支持对应的云服务商；相比之下文件验证方式理论上要容易得多，但是经过测试，国内服务器不容易成功，验证的时候已经有来自多个地方的IP正常访问验证文件，但是最后还是提示超时，暂时不知道什么原因。所以一般优先选择文件验证，如果文件验证无法使用才使用DNS验证。
>
> 另外，第一次尝试一般都会遇到各种问题，所以建议先使用测试环境（Let’s Encrypt 提供了不受速率限制的测试环境，一般工具都支持更改为环境，比如`lego`通过指定`--server=https://acme-staging-v02.api.letsencrypt.org/directory`来使用测试环境）尝试，通过后再使用正式环境。

## 案例

1. 自动启动一个HTTP服务器提供验证文件（上面说的文件验证方式）。

   如果`80`或`443`端口没有被占用，则使用此方式是最简单的。工具会自动开启一个临时服务器用于验证，使用完之后会自动关闭掉。

   ```sh
   $ lego \
     --server=https://acme-staging-v02.api.letsencrypt.org/directory `# 由于演示，所以使用Mock服务器，所以不会颁发真实证书，真实申请正式时去掉该选项即可`\
     -a `# 该选项表示静默接受条款，否则需要交互式输入'Y'显示接受`\
     --path $LEGO_PATH `# 证书存放路径，建议指定，如设置了该环境变量可省略`\
     --email="m@laeni.cn" `# 邮箱，用于紧急情况下发送通知等`\
     --domains="laeni.cn" --domains="*.laeni.cn" `# 指定域名，多个域名需要多次指定`\
     --http `# 启动一个HTTP服务器来提供验证文件`\
     run
   ```

1. 将验证文件写入现有服务器静态目录。

   如果已经有服务器在`80和443`端口提供WEB服务，并且外部程序可以往网站静态目录写入内容则可以使用此方式，比如常见的Nginx。

   ```sh
   $ lego \
     --server=https://acme-staging-v02.api.letsencrypt.org/directory `# 由于演示，所以使用Mock服务器，所以不会颁发真实证书，真实申请正式时去掉该选项即可`\
     -a `# 该选项表示静默接受条款，否则需要交互式输入'Y'显示接受`\
     --path $LEGO_PATH `# 证书存放路径，建议指定，如设置了该环境变量可省略`\
     --email="m@laeni.cn" `# 邮箱，用于紧急情况下发送通知等`\
     --domains="laeni.cn" --domains="*.laeni.cn" `# 指定域名，多个域名需要多次指定`\
     --http `# 启动一个HTTP服务器来提供验证文件`\
     --http.webroot /etc/nginx/html `# 指定WEB服务器根目录，运行时验证文件将写入该目录`\
     run
   ```

1. 使用阿里云DNS验证。

   创建一个子用户并授权DNS操作权限（安全的迁移下也可以直接使用主账户），然后记录下账户密钥。

   ```sh
   ALICLOUD_ACCESS_KEY=abcdefghijklmnopqrstuvwx
   ALICLOUD_SECRET_KEY=your-secret-key
   ```

   指定为DNS方式验证。

   ```sh
   $ ALICLOUD_ACCESS_KEY=$ALICLOUD_ACCESS_KEY ALICLOUD_SECRET_KEY=$ALICLOUD_SECRET_KEY \
     lego \
     --server=https://acme-staging-v02.api.letsencrypt.org/directory `# 由于演示，所以使用Mock服务器，所以不会颁发真实证书，真实申请正式时去掉该选项即可`\
     -a `# 该选项表示静默接受条款，否则需要交互式输入'Y'显示接受`\
     --path $LEGO_PATH `# 证书存放路径，建议指定，如设置了该环境变量可省略`\
     --email="m@laeni.cn" `# 邮箱，用于紧急情况下发送通知等`\
     --domains="laeni.cn" --domains="*.laeni.cn" `# 指定域名，多个域名需要多次指定`\
     --dns alidns `# 指定使用阿里云 DNS 提供商`\
     run
   ```
   
1. 使用自定义`CSR`申请证书。

   默认情况下工具会自动创建私钥，并根据私钥提取的公钥以及域名等信息创建`CSR`提交给证书颁发机构，但也可以自己创建好`CSR`，创建方式可参考可参考[cfssl工具帮助文档](/note/security/cfssl)或[生成K8s所需的证书和密钥及用户配置文件](/note/k8s/gen-k8s-cert)。

   申请证书：

   ```sh
   $ ALICLOUD_ACCESS_KEY=$ALICLOUD_ACCESS_KEY ALICLOUD_SECRET_KEY=$ALICLOUD_SECRET_KEY \
     lego \
     --server=https://acme-staging-v02.api.letsencrypt.org/directory `# 由于演示，所以使用Mock服务器，所以不会颁发真实证书，真实申请正式时去掉该选项即可`\
     -a `# 该选项表示静默接受条款，否则需要交互式输入'Y'显示接受`\
     --path $LEGO_PATH `# 证书存放路径，建议指定，如设置了该环境变量可省略`\
     --email="m@laeni.cn" `# 邮箱，用于紧急情况下发送通知等`\
     --dns alidns `# 指定使用阿里云 DNS 提供商`\
     --csr $LEGO_PATH/tmp/_.laeni.cn.csr `# 自定义创建的 CSR，里面已包含公钥，所以最后的文件不会再自动生成私钥`\
     run
   ```
   
   > **强烈不推荐**自定义`CSR`方式，理由如下：
   >
   > 1. 自定义`CSR`方式方式较为麻烦，增加了复杂度。
   > 2. 自动方式也无需担心私钥泄露。虽然没有细致研究`ACME`协议，但是根据`pki`常用玩法，私钥是自己生成的，而非证书颁发机构生成，所以无需担心私钥泄露问题。
   > 3. 如果自定义`CSR`是为了重复使用同一个私钥，那安全性将降低，并且违背了证书到期的初衷，所以不管使用哪种方式都需要再每次申请新证书时创建新的密钥对。
   > 4. 在 Let’s Encrypt  中，自定义`CSR`并不能自定义证书信息（比如`C-国家`/`O-组织`等）。经过测试，即时通过自定义`CSR`明确指定这些信息，最后也会被  Let’s Encrypt 忽略，可能  Let’s Encrypt 认为，他们只能验证域名的所有权，而不能验证其他信息的正确性，所以干脆只接丢掉，毕竟没有这些信息也不影响证书的正常工作。（目前没深入研究忽略自定义证书信息这一策略是  Let’s Encrypt 规定还是`ACE`协议规定，又或者是工具的Bug。）

> 调试通过后注意去掉`--server`选项（默认为正式环境地址），以申请正式证书。

# 续签证书

续签证书参数和申请证书类似，但是要确保`--path`路径存在已申请证书，否则会续签失败。

要定期续签证书，只需要定期执行续签命令即可，Linux中常用的方式是通过创建一个 cron 作业（或 systemd 计时器）来自动更新证书，详情参见[lego官方文档](https://go-acme.github.io/lego/usage/cli/renew-a-certificate/)。而即使续期脚本频繁执行（比如每天一次），证书也不会频繁续签，默认情况下只有当证书有效期小于30天时才续签。

```sh
$ cat <<EOF | tee $LEGO_PATH/renew.sh
#!/bin/sh
# 续签证书脚本，一般只会执行一次，后续的都是定期执行续签脚本进行续签

# 阿里云密钥，要注意保密，过程中需要操作DNS
ALICLOUD_ACCESS_KEY=$ALICLOUD_ACCESS_KEY ALICLOUD_SECRET_KEY=$ALICLOUD_SECRET_KEY LEGO_PATH=$LEGO_PATH \
/usr/local/bin/lego \
  -a `# 该选项表示静默接受条款，否则需要交互式输入'Y'显示接受`\
  --path $LEGO_PATH `# 证书存放路径，建议指定，如设置了该环境变量可省略`\
  --email="m@laeni.cn" `# 邮箱，用于紧急情况下发送通知等`\
  --domains="laeni.cn" --domains="*.laeni.cn" `# 指定域名，多个域名需要多次指定`\
  --dns alidns `# 指定使用阿里云 DNS 提供商`\
  renew \
  --days 7 `# 只有当证书过期时间小于7天时才续期（默认是30天）`\
  --renew-hook 'docker exec nginx nginx -s reload' `# 续期成功后执行后续脚本，这里续期成功后 reload Nginx.`
EOF
```

## 定时执行续签脚本

使用`crontab -e`打开任务列表后，添加续签脚本。

```
36 5 * * * $LEGO_PATH/renew.sh
```

> `cron`定时人物上下文默认没有`/usr/local/bin/`环境变量，所以尽量使用全路径。
