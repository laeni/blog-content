---
title: 基于Nginx搭建正向代理服务器实现科学上网
description: '使用Nginx + ngx_http_proxy_connect_module 模块搭建支持https的正向代理服务器，以解决某些开发过程中很多国外文档网站被屏蔽的问题'
author: 'Laeni'
tags: Nginx, 正向代理, ngx_http_proxy_connect_module, FanQiang, VPN, 科学上网
date: '2021-05-16'
updated: '2021-05-23'
---
**注意：**

- 本文涉及的内容仅仅用于学习与交流,，切勿用于非法用途。
- 本文会尽量详细的说明正向代理的搭建已经相关理论（个人理解），如有错误敬请谅解并欢迎指正（在评论未支持前可通过[关于博主](../../../about/self)中的[邮箱](mailto:m@laeni.cn)进行反馈）。

作为一个普通开发人员，访问国外网站是很常见的，比如[github](https://github.com)、[tailwindcss](https://tailwindcss.com)。这不，最近似乎是受到建党100周年的影响，凡是基于[Vercel](https://vercel.com/)的所有博客和文档站点统统无法访问，刚开始以为是Vercel本身挂了，但是后面发现被墙的可能性比较大，因为所有请求都拒绝连接且从大陆以外的地方访问不受影响。

有些必须访问的网站被墙了怎么办呢？答案肯定是“翻”。所以接下来就是怎么“翻”的问题了。对于这个问题，我想大部分人第一想法应该都是VPN，因为我刚开始也是这样想的，我甚至还专门去对比了下买一个VPN划算还是自己搭建一个划算（对比结果是都不划算，而且VPN太麻烦了）。除了VPN之外，其实还有其他方法可以实现，比如本文接下来讲的正向代理。

下面我们来讲一下VPN与正向代理，以及怎样利用Nginx搭建正向代理服务器。

## VPN与正向代理

### VPN与正向代理的区别

VPN ：全称是虚拟专用网，只的是符合VPN定义的这一类网，所以它并没有一个死标准，所以这也是为什么有很多VPN协议的原因。下面我们从理论上实现一个自己的VPN。

正向代理：这里的正向代理是狭义的正向代理，比如Nginx的功能，在本小节中，我们后面将这个狭义的正向代理称之为“普通正向代理”。

从“翻”这个动作来看，其实VPN与普通正向代理是一样的，也属于正向代理的一种，都是委托第三方帮我们获取我们想要的内容，即：本地 -> 服务器 -> 目标网站。他们的区别就在于，VPN是“安全”的，这里的安全体现在“本地 -> 服务器”之间，而“服务器 -> 目标网站”之间的安全性在VPN和普通正向代理之间都是一样的。即在VPN情况下，不管是http还是https，本地与服务器的传输内容不会被别人知道，但是普通的正向代理就不一定了，如果通过网站内容是http的话，理论上本地与服务器间的传输内容是不安全的，因为没有加密；但是使用https的网站却是安全的，因为https是建立在http的基础上，对于普通正向代理来说，它只管http就行了（可以这样理解），但是通过http传输的内容是被加密的。

### 将普通正向代理改造为VPN

上面说了，普通正向代理与VPN的区别在于“本地 -> 服务器”间的传输内容是否安全，根据这个理论，我们只要想办法保证这二者之间的安全，那么它就是VPN。实际上解决方法有很多，比如可以将服务端实际负责正向代理的应用（如Nginx）的端口对外屏蔽，这样该应用就变成一个内网应用，所以可以认为是安全的，然后可以使用内网穿透工具（如[frp](https://github.com/fatedier/frp)的stcp协议）或类似工具将本地请求安全代理到目标服务器，这样就实现了简易版本的VPN。

但是实际上，我们没必要将普通正向代理改造为VPN，因为我们这里的作用是通过代理访问外部内容，而且这些内容大多都是https协议的，即使有http也没关系（因为该http网站提供方都不介意，我们使用者又何必介意）。这样的话代理使用和搭建都非常简单，要是使用VPN，动不动就证书密钥啥的。

## 利用Nginx搭建正向代理服务器

### Nginx正向代理与反向代理

其实网上有很多关于正向代理与反向代理的概念，不过这里还是稍微提一下，且以我理解的方式复述一遍，以便后面讲怎么配置。

正向代理简称“代理”，即委托的意思，比如A委托B访问C的内容，这里B就是代理。由于每次代理时都要告诉代理需要访问的内容，所有被代理的内容可以是任意的，访问什么内容完全由委托人（这里的A）决定。

反向代理是相对于正向代理而言的，比如A直接访问B，但是B可能将这个请求转给C或者D，这样对A来说，它是不知道它访问的内容实际来源于哪里，对于B来说，B是很清楚要将请求转给谁的。

在Nginx配置中，对应上面的配置就是：反向代理可以将代理服务器地址写死，而正向代理则不能写死。在nginx中，可以使用相应的变量来表示被代理的资源。具体核心配置如下所示（仅部分，正向代理完整可用配置见文末安装脚本）：

**反向代理**

```bash
server {
    ...

    location / {
        # 明确指定需要将转给谁（可以同时指定多个，但即使是多个，相对于正向代理来说也是确定的）
        proxy_pass http://xxx:port;
    }

}
```

**正向代理**

```bash
server {
    ...

    location / {
        # 通过变量拼接代理地址
        proxy_pass $scheme://$http_host$request_uri;
    }

}
```

### 搭建反向代理

默认情况下，正向代理通过如上配置之后，能正确代理http协议的请求，但是对于https协议的内容将会失败，解决的方式是需要给Nginx安装ngx_http_proxy_connect_module模块，具体安装方法我没试过，但是想来网上多的是。不过再怎么多，但是肯定不简单， 多数情况下需要从源码编译安装。而实际上，我们可以找一个别人已经做好的镜像即可直接用，比如*arilot/docker-nginx*，这样不仅不用自己安装要踩一堆坑外，还能将很方便的使用。

现实中，我们有代理需求的情况应该还是不多的，所以我们一般情况下会在有需要时去云服务商买一个最低配的海外服务器即可。由于我们不要求它有很高的稳定性，所以我们还可以选择竞价实例以节省成本，用完了回收掉就行。只不过为了进一步简化我们的操作，在购买云服务器的时候可以去镜像市场选择带docker环境的，这样拿来就可以使用通过一个脚本安装好Nginx服务器。安装好后还可以做成自定义镜像，每次购买云服务器时直接通过自定义镜像初始化服务器（没尝试过，感觉也没必要）。

以下是在Docker环境下搭建使用Nginx镜像搭建正向代理服务器脚本，一般情况是无需修改即可直接使用。

```shell
#!/bin/bash
# 用于快速在docker环境中搭建nginx正向代理

# 生成 Nginx 配置文件
newNgConf(){
  cat > ./nginx.conf <<EOF
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    client_max_body_size 2048m;
    log_format  main  '\$remote_addr - \$remote_user [\$time_local] "\$request" '
                      '\$status \$body_bytes_sent "\$http_referer" '
                      '"\$http_user_agent" "\$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;

    server {
        listen       80;
        listen  [::]:80;
        server_name  localhost;
        resolver 8.8.8.8;

        # forward proxy for CONNECT request
        proxy_connect;
        proxy_connect_allow            443 563;
        proxy_connect_connect_timeout  10s;
        proxy_connect_read_timeout     10s;
        proxy_connect_send_timeout     10s;

        location / {
            proxy_pass \$scheme://\$http_host\$request_uri;
            proxy_set_header Host \$host;
            proxy_buffers   256 4k;
            proxy_max_temp_file_size 0k;
        }
    }
}
EOF
}
newNgConf

name=forward-nginx

# 尝试停止并删除名为 $name 的容器，第一次执行会输出错误，不过不影响
docker stop $name
docker rm $name

docker run -d \
  --restart=always \
  --name $name \
  -p 8080:80 \
  -v "$(pwd)/nginx.conf:/etc/nginx/nginx.conf" \
  -v /etc/localtime:/etc/localtime \
  arilot/docker-nginx:latest

```

在本脚本中，服务对外的端口是8080，如果使用云服务器时需要开房对于的端口并确保安全组也开放该端口。可以使用将其改为80，这样一般不用动防火墙，只需要检查安全组即可，因为防火墙一般都公开80端口，并且安全组一般也公开80端口。只不过根据日常习惯，作为一个正向代理服务器，似乎使用80端口是不合适的，而反向代理服务器则很合适。

搭建后之后就可以在浏览器或系统代理中直接使用服务器外网ip+端口的形式使用了。

## 浏览器中配合插件实现自动选择是否使用代理

默认情况下，如果给浏览器或者系统设置代理后，则几乎所有请求都会被代理，只有像localhost或127.0.0.1这样的特殊且极少数的请求不会走代理（也可以配置都走），但我们的需求是只有极少数需要代理的部分才使用代理，其他的不使用。在浏览器中可以通过插件来实现，具体实现方法参考下面的360示例，不知道系统级别的代理有没有类似的解决方案。

以360安全浏览器（个人虽然是程序员，但对360浏览器情有独钟，原因有二：一是书签同步方便，不用担心书签丢失；二是window下360为双内核，曾经开发web时可以当做两个浏览器调试）为例，可以在扩展管理中安装“*Proxy SwitchyOmega*”插件的“*auto switch*”功能来解决自动选择代理问题，即只为需要代理的域名使用代理，其他的走“直连”就好。安装完成后该插件会提示怎么配置，仔细观看即可。下图为360浏览器中“*Proxy SwitchyOmega*”插件的部分配置，其他浏览器（如Chrome）一般也有类似插件。

![SwitchyOmega](https://pictures-1252266447.cos.ap-chengdu.myqcloud.com/blog/program/net/forward-proxy/SwitchyOmega.jpg)

