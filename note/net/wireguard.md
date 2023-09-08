---
title: WireGuard 的简单使用
author: 'Laeni'
tags: VPN
date: '2022-12-06'
updated: '2023-02-17'
---

关于*WireGuard*的详细介绍可参见官方[WireGuard官网](https://www.wireguard.com/)或查看[另一篇博文](/share/20221208)。

# 安装

以下为**Ubuntu**安装方式，其他系统可以在官网找到对应的包或安装方法。

```sh
$ sudo apt update # 新系统如果不更新源可能存在找不到 wireguard 的问题
$ sudo apt install wireguard
```

> 除了 Linux 外一般安装图形化界面，官网下载页面提供了所有系统的实现。

**命令方式**开机自启动：

```sh
$ systemctl enable wg-quick@wg0
```

> 不要安装完成后就开启自启动，因为上面命令中的`wg0`对应`/etc/wireguard/`目录下名为`wg0.conf`的配置文件，所以需要手动配置好之后再使用自启动。

目前*Windows*需要FQ，*Mac*和*iOS*需要非大陆帐号才能下载，所以为了方便使用，以下提供`Android`、`Windows`和`Mac`安装包：

- [Android-1.0.20220515.apk](https://chengdu-1252266447.cos.ap-chengdu.myqcloud.com/share/package/wireguard/Android-1.0.20220515.apk)
- [Windows-amd64-0.5.3.msi](https://chengdu-1252266447.cos.ap-chengdu.myqcloud.com/share/package/wireguard/Windows-amd64-0.5.3.msi)
- [Mac-amd64-1.0.16.tar.gz](https://chengdu-1252266447.cos.ap-chengdu.myqcloud.com/share/package/wireguard/Mac-amd64-1.0.16.tar.gz) [Mac-amd64-1.0.15.tar.gz](https://chengdu-1252266447.cos.ap-chengdu.myqcloud.com/share/package/wireguard/Mac-amd64-1.0.15.tar.gz)

## OpenWRT

### 安装

进入应用商店**系统->Software**，更新包列表之后直接安装`luci-i18n-wireguard-zh-cn`即可（其他依赖的包会自动安装）。安装完成后要重启，否则WEB会缺失一些内容。

### 配置

1. 创建网络接口。

   进入**网络 -> 接口 -> 新建新接口**，**名称**习惯性填写`wg0`，**协议**选择`WireGuard VPN`。

2. 常规配置。

   一般填写**私钥**和**IP**即可。

3. 防火墙设置。

   **创建/分配防火墙区域**选择`lan`。

4. 对端。

   配置上和另外一台机器的约定配置即可。

   > 注意，一般需要从外部访问连接到路由器上的终端，所以最好设置**持续 Keep-Alive**参数（最好和服务器一致），否则路由器启动后如果路由器这端没有应用访问对端，则网络链路不会打通，这样也不能通过对端连接到路由器网络。

5. 添加启动定制命令

   ```
   PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o wan -j MASQUERADE
   PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o wan -j MASQUERADE
   
   PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o br-lan -j MASQUERADE
   PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o br-lan -j MASQUERADE
   ```

   

# 常用命令

```shell
# 不中断活跃连接的情况下重新加载配置文件
$ wg syncconf wg0 <(wg-quick strip wg0)
```

> 有时候会不生效，如果不生效就只能重启了。

# 参考文档

- https://gobomb.github.io/post/wireguard-notes/
- <https://blog.mygallop.cn/2022/06/centos/wireguard-install/>
- [wg-quick 路由策略解读](https://icloudnative.io/posts/linux-routing-of-wireguard/)
- [Wireguard 使用笔记](https://gobomb.github.io/post/wireguard-notes/)
- [WG-API](https://github.com/jamescun/wg-api)