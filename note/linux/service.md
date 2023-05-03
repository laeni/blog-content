---
title: Linux 中的服务
tags: service, systemd
author: Laeni
date: 2023-05-03
updated: 2023-05-03
---

Linux 中的服务一般用于定义一些软件的开机自启或一些脚本的开机自执行。

# 目录规范

```shell
/
├── etc/systemd/system/ # 系统级别的 service 定义（一般为安装软件时系统自动生成）
├── home/<user_name>/
│   └── .config/systemd/user/ # 用户级别的 service 定义，使用`systemctl --user status xxx.service`操作服务
├── lib*/ -> usr/lib*/
└── usr/
    ├── lib/systemd/system        # 系统级别的 service 定义（一般为系统核心服务）
    └── local/lib/systemd/system/ # 系统级别的 service 定义（一般为手动创建，但是目前测试Ubuntu不生效）
```

> 根据`systemd`文档，service 文件确实可以定义在`/usr/local/lib/systemd/system/`目录中，但是 Ubuntu 中测试不生效，估计是配置不对。
>
> 另外可通过`pkg-config systemd --variable=systemdsystemconfdir`和`pkg-config systemd --variable=systemdsystemunitdir`命令 service 文件所在的目录。
>
> 关于服务单元文件定义可参见[systemd](http://www.jinbuguo.com/systemd/systemd.html#%E7%9B%AE%E5%BD%95)。

# 相关命令

## 检查 service 文件定义是否正确

```sh
$ systemd-analyze verify xxx.service
```

## 查看`xxx`服务的相关启动日志

```sh
$ journalctl -u xxx.service
```

# 参考资料

- [systemctl service失效，在start后自动调用stop (ExecStop)，排查分析及处理过程](https://blog.csdn.net/Peter_JJH/article/details/108446380)
- [systemctl 中文手册](http://www.jinbuguo.com/systemd/systemctl.html)
- [systemd 中文手册](http://www.jinbuguo.com/systemd/systemd.html)