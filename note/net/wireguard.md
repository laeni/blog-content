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

#### 创建网络接口。

进入**网络 -> 接口 -> 新建新接口**，**名称**习惯性填写`wg0`，**协议**选择`WireGuard VPN`。

1. 常规配置。

   一般填写**私钥**和**IP**即可。

2. 防火墙设置。

   **创建/分配防火墙区域**选择`lan`。

3. 对端。

   配置上和另外一台机器的约定配置即可。

   > 注意，一般需要从外部访问连接到路由器上的终端，所以最好设置**持续 Keep-Alive**参数（最好和服务器一致），否则路由器启动后如果路由器这端没有应用访问对端，则网络链路不会打通，这样也不能通过对端连接到路由器网络。

#### 防火墙配置

防火墙配置的目的主要是允许转发，且对转发流量进行**NAT**。如果不允许转发，那从外部直接访问连接在*OpenWRT*设备上的其他终端时，这些设备能完全收不到任何数据包；如果仅开启转发而不做NAT，那一般情况下这些设备能收到包，但是回包会有问题，做了NAT之后，其他网络设备收到包时，看到的原IP地址是网关IP，这样回包就没问题了。

进入**网络 -> 防火墙**页面，将**Forward（转发）**更改为`accept`；勾选`lan => wan`的**Masquerading**。然后保存并应用即可。

该操作可直接用**iptables**命令操作，比如在常规*Liunx*服务器上配置*WireGuard*需要增加**PostUp**和**PostDown**脚本（这里的设置相当于**PostUp**脚本）。即上述操作等同于命令`iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o br-lan -j MASQUERADE`。

```
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o br-lan -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o br-lan -j MASQUERADE
```

> 脚本中`br-lan`是网络接口（平时可能说网卡或网桥）名称，不同设备可能不一样，对于路由器上的*OpenWRT*一般是`br-lan`。
>
> 确认具体是哪个网卡的方法很多，比如通过*SSH*登录到*OpenWRT*后通过`ifconfig`命令得到所有网络接口信息之后，看其他设备接入网络后自动获取的IP地址属于哪个网络接口的网段，那这个网络接口就是目标。

## Mac OS

如果不作为代理节点，那随便安装即可使用，但是如果要作为代理，则需要开启转发并进行**NAT**。

在Mac中，可以使用pf（Packet Filter）来进行转发和NAT。以下是在Mac中开启转发并进行NAT的步骤：

1. 输入以下命令来启用IP转发功能：

   ```bash
   sudo sysctl -w net.inet.ip.forwarding=1
   ```

2. 编辑/etc/pf.conf文件并添加以下规则（假设你的内网接口是en0，外网接口是en1）：

   ```bash
   nat on en0 from en1:network to any -> (en0)
   ```

   > 这将使得来自en1接口的流量通过en0接口进行NAT转发。

3. 输入以下命令启用pf规则：

   ```bash
   sudo pfctl -e
   ```

> 请确保在进行上述操作之前，已经正确配置了wireguard代理节点。并根据实际情况修改相应的接口和网络配置。

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