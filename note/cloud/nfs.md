---
title: NFS常用配置笔记
tags: 'Linux, NFS'
author: Laeni
date: 2022-07-30
updated: 2024-06-02
---

## 服务端配置

### 安装NFS服务器

#### Centos

```shell
$ yum install -y nfs-utils rpcbind
```

#### Ubuntu

```shell
$ sudo apt install nfs-common nfs-kernel-server
```

#### 常用命令

```shell
$ service rpcbind start
$ sudo systemctl enable rpcbind
$ service nfs start
$ service nfs enable
$ service nfslock start
# 重新加载 /etc/exports 配置
$ exportfs -avr
$ systemctl reload nfs
# 显示目标服务器能挂载的目录
$ showmount -e <目标服务器IP>

# 执行以下命令，可提高同时发起的NFS请求数量
sudo echo "options sunrpc tcp_slot_table_entries=128" >>  /etc/modprobe.d/sunrpc.conf 
sudo echo "options sunrpc tcp_max_slot_table_entries=128" >>  /etc/modprobe.d/sunrpc.conf
```

### 暴露可挂载目录

服务器通过`/etc/exports`配置文件暴露可以被客户端挂载的文件夹以及相关权限，文件每一行为一个挂载点，如：

```
# V2/V3
# /srv/homes       hostname1(rw,sync,no_root_squash,no_subtree_check) hostname2(ro,sync,no_root_squash,no_subtree_check)
# V4
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_root_squash,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_root_squash,no_subtree_check)
```

> /home/nfst_shared — 要共享的目录（就是步骤2创建的nfs目录的绝对路径）
> 192.168.0.*              — 允许访问的客户端，可以是ip地址、主机名（能够被服务器解析）、如果设置为"*"，就是所有人都能访问
> rw                              — 读/写权限
> sync                          — 数据同步写入内存和硬盘
> no_root_squash      — 服务器允许远程系统以root特权存取该目录
> no_subtree_check  — 关闭子树检查

修改配置后是热加载配置

```bash
$ exportfs -avr
  -a exports all directory
  -r reexports
```

必要时可能需要重启服务

```shell
$ sudo service nfs-server restart # 或 sudo service nfs-kernel-server restart
```

## 挂载NFS

### Mac

命令：`mount -o resvport <ip>:<NFS目录> <本地挂载目录>`

```shell
$ sudo mount -o resvport 10.10.1.1:/nfs/node/mac nfs_node_mac
```

### Linux

以V3版本挂载：

```shell
$ sudo mount -t nfs -o vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,resvport 10.10.1.1:/nfs/ <本地挂载目录>
```

[建议在 Linux 上使用挂载选项的以下默认值](https://docs.amazonaws.cn/efs/latest/ug/mounting-fs-nfs-mount-settings.html)：

- `rsize=1048576`— 设置 NFS 客户端对每个网络 READ 请求可以接收的数据最大字节数。
- `wsize=1048576`— 设置 NFS 客户端对每个网络 WRITE 请求可以发送的数据最大字节数。
- `hard`— 设置 NFS 客户端在 NFS 请求超时之后的恢复行为，这样 NFS 请求在服务器回复之前会无限重试。建议您使用硬挂载选项 (`hard`) 以确保数据完整性。如果您使用 `soft` 挂载，请将 `timeo` 参数至少设置为 `150` 分秒（15 秒）。这样做可尽量减少源自软挂载的数据损坏风险。
- `timeo=600`— 将超时值设置为 600 分秒（60 秒），这是 NFS 客户端在重试 NFS 请求之前等待响应的时间。如果您必须更改超时参数 (`timeo`)，我们建议您使用至少为 `150` 的值，这相当于 15 秒。这样做有助于避免性能下降。
- `retrans=2`— 将 NFS 客户端重试请求的次数设置为 2，超过此次数之后将尝试进一步的恢复操作。
- `noresvport`— 告知 NFS 客户端在重新建立网络连接时，使用新的传输控制协议 (TCP) 源端口。这样做有助于确保 EFS 文件系统在网络恢复事件后具有不间断的可用性。（NFS服务器可能不支持）
- `_netdev`— 当存在于`/etc/fstab`，阻止客户端尝试挂载 EFS 文件系统，直到启用了网络。
- `nofail`— 如果需要启动您的 EC2 实例而不考虑挂载的 EFS 文件系统状态，请将`nofail`选项将文件系统条目添加到您的`/etc/fstab`文件。

> V3虽然没有V4高，但是V3比V4性能更高，而V4的好处是支持多客户端同时挂载，所以需要根据实际应用场景选择协议版本。

### Windows

1. 启用NFS客户端

   控制面板 -> 启用或关闭Windows功能 -> 勾选全部“NFS服务/NFS客户端”

2. 挂载

   1. [推荐]通过快捷方式挂载（方式一）

      “鼠标右键->新建快捷方式“，填入NFS服务器地址以及挂载路径即可`\\192.168.122.1\home\laeni\Desktop\vm-share`

   2. 通过图形化界面挂载（方式二）

      打开文件管理器 -> 通过顶部的“连接到网络驱动器”，输入通过`showmount -e <ip>`查询的到可挂载路径进行挂载，其中格式为：`\\<ip>\path`，但是这种挂载方式只能挂载到某个盘符下，所以一般使用`mklink`或`mount`命令进行挂载。

   3. 通过mklink挂载（方式三）

      以管理员身份启动`cmd.exxe`进行挂载。此方式本质上和一是一样的。

      ```
      mklink /d C:\Users\xx\Desktop\NFS \\192.168.122.1\home\laeni\Desktop\vm-share
      ```

> 1. Windows挂载后可能没有权限写入，如果没有权限，首先要检查NFS服务器文件权限设置，需要将挂载文件夹设置为`777`或者设置为`nogroup`用户组和`nobody`用户（Ubuntu测试，设置为`777`后随笔通过Windows写入文件之后查看用户信息即可得到）。
>
>    如果还是不行，可能需要添加以下注册表：
>
>    ```
>    Windows Registry Editor Version 5.00
>    
>    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default]
>    "AnonymousUid"=hex(b):00,00,00,00,00,00,00,00
>    "AnonymousGid"=hex(b):00,00,00,00,00,00,00,00
>    
>    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default\RegNotify]
>    "Default"=dword:00000000
>    ```
>
>    将上述内容写入`.reg`后缀的文本文件中双击即可导入，上述注册表的意思是设置默认用户的用户ID和组ID为0，Linux中0表示root用户。
>
> 2. 解决目前（Win10）挂载NFS后文件中文名显示乱码问题
>
>    乱码主要原因是微软NFS协议不支持UTF-8的问题，导致文件乱码，NAS中文件管理器显示正常。目前WIN10中已含有一个Beta设置，支持全局 UTF-8，修改后即可正常显示，步骤如下：
>
>    1. 按下Win+R，输入 intl.cpl，点击确定
>    2. 切换tab 进入"管理"中
>    3. 点击更改系统区域设置
>    4. 勾选Beta版，点击确定，重启后即可

