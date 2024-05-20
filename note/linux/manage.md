---
title: Linux 基本系统管理
tags: Liunx
author: Laeni
date: 2023-11-25
updated: 2023-11-25
---

# 用户和组

## 用户分类

- 超级用户：root。
- 普通用户：一般情况下使用的用户，有权限限制。
- 系统用户：不能直接登录，但可以以其身份运行软件。

## UID与GID

**UID** 即用户 ID，用于唯一标识一个用户；同理，**GID** 为组 ID，用于唯一标识一个组。

常见的 UID 范围如下（有些并不强制，但是为了统一最好遵循这个约定）：

| UID 范围                  | 用户                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 0                         | root                                                         |
| 1 ~ 499                   | 系统用户                                                     |
| 500 ~ $2^{16}-1$（65535） | 普通用户<br />`2.6.x`版本内核开始，UID 最大已经支持$2^{32}-1$ |

## 用户管理

常见的用户管理以及相关的命令（命令使用参见[相关命令](#相关命令)）：

- 创建用户：[useradd](#useradd)
- 删除用户：[userdel](#userdel)
- 密码相关：[passwd](#passwd)
- 修改用户信息：[usermod](#usermod)

## 组管理

常见的组管理以及相关的命令（命令使用参见[相关命令](#相关命令)）：

- 添加用户组：[groupadd](#groupadd)
- 删除用户组：[groupdel](#groupdel)
- 将用户添加进组或从组中删除：[gpasswd](#gpasswd)

## 相关文件

### /etc/passwd

系统用户配置文件，存储了系统中所有用户的基本信息。

含义：`用户名 ：密码 ：UID ：GID ：描述性信息 ：主目录 ：默认Shell`

注意：最开始该文件的第二个字段保存用户的密码 Hash，但由于一些原因，比如普通用户也需要知道该系统有哪些用户，所以该文件对所有用户是可读的，而这带来了安全隐患，所以现在的内核已经将第二个字段全部设置为默认值（`x`），真正的密码存在于 `/etc/shadow` 文件内（该文件仅 root 用户能访问）。

### /etc/shadow

用于存储 Linux 系统中用户的敏感信息（如密码），该文件仅 root 用户能访问，且又称为“影子文件”。

含义：`用户名 ：加密密码 ：最后一次修改时间 ：最小修改时间间隔 ：密码有效期 ：密码需要变更前的警告天数 ：密码过期后的宽限时间 ：账号失效时间 ：保留字段`

### /etc/group

用户组配置文件。

含义：`组名 ：密码 ：GID ：该用户组中的用户列表`

## 相关命令

### id

查询用户的 UID、GID（主组 ID）和所有组（`groups`，主组加附加组）。

示例：

- 查询指定用户的 ID

  ```sh
  $ id root
  uid=0(root) gid=0(root) groups=0(root)
  ```

  > 当缺省用户时表示“当前用户”。

### useradd

创建用户。

基本用法：

```
Usage: useradd [options] LOGIN
       useradd -D
       useradd -D [options]
```

| 选项                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| --badnames                   | do not check for bad names                                   |
| b, --base-dir BASE_DIR       | base directory for the home directory of the new account     |
| --btrfs-subvolume-home       | use BTRFS subvolume for home directory                       |
| -c, --comment COMMENT        | 账户的 GECOS 字段（用户描述）。（对应 `/etc/passwd` 文件中的第 5 个字段） |
| -d, --home-dir HOME_DIR      | 指定用户的主目录                                             |
| -D, --defaults               | 打印或更改 useradd 的默认配置                                |
| -e, --expiredate EXPIRE_DATE | 指定用户的失效曰期。格式为 “YYYY-MM-DD”。（对应 `/etc/shadow` 文件中的第 8 个字段） |
| -f, --inactive INACTIVE      | password inactivity period of the new account                |
| -g, --gid GROUP              | 指定用户的主组（可以是组名或 GID）                           |
| -G, --groups GROUPS          | 指定用户的附加组（可以指定多个）                             |
| -h, --help                   | display this help message and exit                           |
| -k, --skel SKEL_DIR          | use this alternative skeleton directory                      |
| -K, --key KEY=VALUE          | override /etc/login.defs defaults                            |
| -l, --no-log-init            | do not add the user to the lastlog and faillog databases     |
| -m, --create-home            | 创建用户的主目录                                             |
| -M, --no-create-home         | 不要创建用户的主目录                                         |
| -N, --no-user-group          | 不要创建与用户同名的组                                       |
| -o, --non-unique             | allow to create users with duplicate (non-unique) UID        |
| -p, --password PASSWORD      | encrypted password of the new account                        |
| -r, --system                 | 将新用户创建为系统用户                                       |
| -R, --root CHROOT_DIR        | directory to chroot into                                     |
| -P, --prefix PREFIX_DIR      | prefix directory where are located the /etc/* files          |
| -s, --shell SHELL            | 指定用户的登录 Shell                                         |
| -u, --uid UID                | 指定用户的 UID                                               |
| -U, --user-group             | create a group with the same name as the user                |
| -Z, --selinux-user SEUSER    | use a specific SEUSER for the SELinux user mapping           |
| --extrausers                 | Use the extra users database                                 |

> 手册：<http://bash.lutixia.cn/c/useradd.html>。

### userdel

删除用户。

基本用法：

```
Usage: userdel [options] LOGIN
```

| 选项                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| -f, --force                  | 强制删除文件，即使这些文件不属于用户 |
| -h, --help                   | display this help message and exit                           |
| -r, --remove                 | 删除用户的同时，删除与用户相关的所有文件（主目录和邮件池） |
| -R, --root CHROOT_DIR        | directory to chroot into                                     |
| -P, --prefix PREFIX_DIR      | prefix directory where are located the /etc/* files          |
| --extrausers | Use the extra users database   |
| -Z, --selinux-user | 删除该用户的任何 SELinux 用户映射 |

### passwd

配置密码。

基本用法：

```
Usage: passwd [options] [LOGIN]
```

| 选项                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| -a, --all | report password status on all accounts |
| -d, --delete | 删除指定帐户的密码 |
| -e, --expire | force expire the password for the named account |
| -h, --help | display this help message and exit |
| -k, --keep-tokens | change password only if expired |
| -i, --inactive INACTIVE | 将密码过期后无效设置为 INACTIVE（对应 `/etc/shadow` 文件中的第 7 个字段） |
| -l, --lock | 锁定指定帐户的密码（在 `/etc/shadow` 文件中用户密码前添加 `!`） |
| -n, --mindays MIN_DAYS | 将更改密码前的最短天数设置为 MIN_DAYS（对应 `/etc/shadow` 文件中的第 4 个字段），即至少要间隔 MIN_DAYS 后才能修改密码 |
| -q, --quiet | quiet mode |
| -r, --repository REPOSITORY | change password in REPOSITORY repository |
| -R, --root CHROOT_DIR | directory to chroot into |
| -S, --status | 查看指定帐户的密码状态 |
| -u, --unlock | 解锁指定帐户的密码（和 `-I` 选项对应） |
| -w, --warndays WARN_DAYS | 将过期警告天数设置为 WARN_DAYS（对应 `/etc/shadow` 文件中的第 6 个字段） |
| -x, --maxdays MAX_DAYS | 将更改密码前的最大天数设置为 MAX_DAYS（对应 `/etc/shadow` 文件中的第 5 个字段），即超过 MAX_DAYS 后必须修改密码 |

### usermod

修改用户信息。

基本用法：

```
Usage: usermod [options] LOGIN
```

| 选项                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| -b, --badnames | allow bad names |
| -c, --comment COMMENT | 账户的 GECOS 字段（用户描述）。（对应 `/etc/passwd` 文件中的第 5 个字段） |
| -d, --home HOME_DIR | 用户的主目录。（对应 `/etc/passwd` 文件中的第 6 个字段） |
| -e, --expiredate EXPIRE_DATE | 将帐户到期日期设置为 EXPIRE_DATE（格式为 “YYYY-MM-DD”）。 |
| -f, --inactive INACTIVE | set password inactive after expiration to INACTIVE |
| -g, --gid GROUP | force use GROUP as new primary group |
| -G, --groups GROUPS | new list of supplementary GROUPS |
| -a, --append | append the user to the supplemental GROUPS mentioned by the -G option without removing the user from other groups |
| -h, --help | display this help message and exit |
| -l, --login NEW_LOGIN | new value of the login name |
| -L, --lock | lock the user account |
| -m, --move-home | move contents of the home directory to the new location (use only with -d) |
| -o, --non-unique | allow using duplicate (non-unique) UID |
| -p, --password PASSWORD | use encrypted password for the new password |
| -R, --root CHROOT_DIR | directory to chroot into |
| -P, --prefix PREFIX_DIR | prefix directory where are located the /etc/* files |
| -s, --shell SHELL | new login shell for the user account |
| -u, --uid UID | new UID for the user account |
| -U, --unlock | unlock the user account |
| -v, --add-subuids FIRST-LAST | add range of subordinate uids |
| -V, --del-subuids FIRST-LAST | remove range of subordinate uids |
| -w, --add-subgids FIRST-LAST | add range of subordinate gids |
| -W, --del-subgids FIRST-LAST | remove range of subordinate gids |
| -Z, --selinux-user SEUSER | new SELinux user mapping for the user account |

### groupadd

添加用户组。

基本用法：

```
Usage: groupadd [options] GROUP
```

| 选项                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| -f, --force | 如果组已存在则命令成功执行并正常结束；如果 GID 已被使用则取消 -g 选项 |
| -g, --gid GID | 指定组 GID |
| -h, --help | display this help message and exit |
| -K, --key KEY=VALUE | override /etc/login.defs defaults |
| -o, --non-unique | allow to create groups with duplicate (non-unique) GID |
| -p, --password PASSWORD | 指定组的加密密码 |
| -r, --system | create a system account |
| -R, --root CHROOT_DIR | directory to chroot into |
| -P, --prefix PREFIX_DIR | directory prefix |
| --extrausers | Use the extra users database |

### gpasswd

基本用法：

```
Usage:  gpasswd [options]
```

| 选项 | 说明                                                 |
| ---- | ---------------------------------------------------- |
| -a   | 添加用户到组                                         |
| -d   | 从组删除用户                                         |
| -A   | 指定管理员                                           |
| -M   | 指定组成员和-A的用途差不多                           |
| -r   | 删除密码                                             |
| -R   | 限制用户登入组，只有组中的成员才可以用newgrp加入该组 |



