Kubernetes的轻量级容器运行时。

[管网](https://cri-o.io/) [github](https://github.com/cri-o/cri-o)

## CRI-O功能

### 包括

- 在在符合OCI的运行时和kubelet之间提供集成路径
- 支持多种镜像格式，包括现有的Docker镜像格式
- 支持多种下载镜像的方式，包括信任和镜像验证
- 容器镜像管理（管理镜像层，覆盖文件系统等）
- 容器过程生命周期管理
- 满足CRI所需的监视和记录
- CRI要求的资源隔离

### 不包括

- 构建，签名并将图像推送到各种图像存储
- 用于与CRI-O交互的CLI实用程序

### 说明

- CRI-O使用 container/image 来管理图像
- CRI-O使用 containers/storage 来管理容器存储

## 安装

<https://github.com/cri-o/cri-o/blob/master/install.md>

### 基础依赖

> 这些依赖一般情况不需要显示安装，除非从源码构建时可能需要考虑

- [conmon](https://github.com/containers/conmon)

  conmon是每个容器的守护程序，CRI-O用其监视容器日志和退出信息

- [cni插件](https://github.com/cri-o/cri-o/blob/master/contrib/cni/README.md)

  众多网络插件(contrib/cni)

- ......

### 安装命令

以下仅仅是特定示例，完整教程见[管网](https://github.com/cri-o/cri-o/blob/master/install.md)。

```sh
# CentOS 8
export VERSION=1.22 # 设置需要安装的版本
export OS=CentOS_8  # CentOS 有多个版本,需要明确指定
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo
yum install cri-o

# Debian 11
export VERSION=1.22 # 设置需要安装的版本
export OS=Debian_11
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -
apt-get update
apt-get install cri-o cri-o-runc
```

##\# 启动

```sh
sudo systemctl daemon-reload
sudo systemctl enable crio
sudo systemctl start crio
```

## 配置文件

| 文件            | 描述             | PATH                            |
| --------------- | ---------------- | ------------------------------- |
| crio.conf       | CRI-O配置文件    | /etc/crio/crio.conf             |
| policy.json     | 签名验证策略文件 | /etc/containers/policy.json     |
| registries.conf | 注册表配置文件   | /etc/containers/registries.conf |
| storage.conf    | 存储配置文件     | /etc/containers/storage.conf    |

