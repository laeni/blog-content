---
title: 从 Let’s Encrypt 申请免费证书
author: 'Laeni'
tags: SSL, HTTPS, 证书, 安全
date: '2022-11-29'
updated: '2022-11-29'
---

Let’s Encrypt 是一个证书颁发机构（CA），对外提供[ACME](https://www.rfc-editor.org/rfc/rfc8555)协议的API，可以通过该API申请证书。而可以与ACME协议的API交互的客户端工具有很多，官方列举了一些[比较好的实现](https://letsencrypt.org/zh-cn/docs/client-options/)。这里我们选用其中的[lego](https://github.com/go-acme/lego)工具使用，选它的一个重要原因是该工具使用Go编写，直接到[项目的releases页](https://github.com/go-acme/lego/releases)下载可执行二进制即可使用。

除非使用命令行`--path`标志单独说明，否则`lego`将在*当前工作目录中*查找`.lego`命名的目录。 如果您运行`cd /dir/a && lego ... run`，`lego`将创建一个`/dir/a/.lego`目录，它将在其中保存帐户注册和证书文件。 如果您稍后尝试续订证书`cd /dir/b && lego ... renew`，`lego`可能会产生错误（因为在新目录下无法找到可以续订的证书）。

# 获取证书

证书不是随便都能申请的，*证书颁发机构*需要验证域名所有权后才能给对应域名颁发证书，而验证域名所有权的方法一般为两种：

1. DNS验证。添加一个`TEXT`类型的DNS解析记录，记录值为证书颁发机构生成的随机值。
2. 文件验证。将证书颁发机构生成的随机值写入指定的Web服务器静态文件中，该文件必须能通过申请证书的域名访问到，访问端口必须是`80`或`443`。不过使用该方式时必须确保该服务器能够从境外访问，否则可能无法验证通过。

由于工具是自动化的，所以采用DNS方式验证很困难（工具自动添加DNS记录很困难），所以一般采用第二种方式，这里也是采用第二种方式。

## 案例

1. 自动启动一个HTTP服务器提供验证文件（上面说的文件验证方式）。

   如果`80`或`443`端口没有被占用，则使用此方式是最简单的。工具会自动开启一个临时服务器用于验证，使用完之后会自动关闭掉。

   ```sh
   $ lego \
     --server=https://acme-staging-v02.api.letsencrypt.org/directory `# 由于演示，所以使用Mock服务器，所以不会颁发真实证书，真实申请正式时去掉该选项即可`\
     -a `# 该选项表示静默接受条款，否则需要交互式输入'Y'显示接受`\
     --path /mnt/share/archive/cert/www/.lego `# 证书存放路径，不指定时默认存放在当前目录的'.lego'下`\
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
     --path /mnt/share/archive/cert/www/.lego `# 证书存放路径，不指定时默认存放在当前目录的'.lego'下`\
     --email="m@laeni.cn" `# 邮箱，用于紧急情况下发送通知等`\
     --domains="laeni.cn" `# 指定域名，多个域名需要多次指定`\
     --http `# 启动一个HTTP服务器来提供验证文件`\
     --http.webroot /mnt/share/node/tx/app-data/nginx/html `# 指定WEB服务器根目录，运行时验证文件将写入该目录`\
     run
   ```
   
   

