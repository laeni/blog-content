---
title: Linux中一些操作流程及原理（仅备案）
tags: 'Linux常用操作'
author: Laeni
date: '2022-07-28'
updated: '2022-07-28'
---

## 关闭`systemd-resolve`监听的53端口

`systemd-resolve`监听53端口后，可以将地址`127.0.0.53`作为DNS服务器，但是如果需要搭建DNS时就需要将其关闭，关闭步骤如下：

1. 重新链接`/etc/resolv.conf`

   在`/etc/resolv.conf`配置文件中指定了系统所使用的DNS服务器，而该文件默认是链接自`/run/resolvconf/resolv.conf`，而该文件中的DNS服务器`127.0.0.53`实际上只是指向`systemd-resolved`进程，该进程监听53端口并拿到DNS解析请求后，会将请求转发给真实的DNS服务器。之所以这样做是因为实际的DNS服务器可能会发生变化，比如网络环境变化时，默认的DNS地址一般也会随之变化。而`/run/systemd/resolve/resolv.conf`配置文件记录了真实DNS地址，该文件由`systemd-resolved`管理，不能人为编辑，当DNS变更后，`systemd-resolved`会自动更新该配置文件。

   由于需要关闭该程序监听的53端口，关闭后`127.0.0.53`地址将不可用，即`/run/resolvconf/resolv.conf`配置文件将失效，所以可以直接将`/run/systemd/resolve/resolv.conf`链接到`/etc/resolv.conf`，以解决关闭53端口后DNS服务不可用的问题：

   ```shell
   # 这一步一般要提前做，防止关闭53端口后出现异常
   $ ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
   ```

   系统不直接使用`/run/systemd/resolve/resolv.conf`而要使用`/run/resolvconf/resolv.conf`的原因暂时未知，可能是想给应用提供一个永远固定的地址，但是我们可以放心地使用`/run/systemd/resolve/resolv.conf`，因为该文件里有相关说明。

2. 修改`etc/systemd/resolved.conf`配置文件关闭监听

   ```text
   - #DNSStubListener=yes
   + DNSStubListener=no
   ```

3. 重启`systemd-resolve`服务

   ```shell
   $ systemctl restart systemd-resolved.service
   ```

   注意尽量不要将其关闭，甚至是禁用，因为禁用后可能导致网络出现问题（我有一次将云服务器的该服务禁用掉，连阿里云救援模式也没用，差点就重装系统了）。



## Liunx目录规范

```shell
/
├── boot
├── cdrom
├── data
├── dev
├── etc
│   └── systemd
│       └── system/ # 系统级别的 service 定义
├── home
│   └── <user_name>
│       ├── .config
│       │   └── systemd/user/ # 用户级别的 service 定义，使用`systemctl --user status xxx.service`操作服务
│       └── .local
│           └── bin/  # 用户级别的可执行二进制软件
├── /lib
│   └── /systemd/system # 系统级别的 service 定义（一般为系统核心服务）
├── media
├── mnt
├── opt
├── proc
├── run
├── srv
├── sys
├── tmp
├── usr
└── var
```



