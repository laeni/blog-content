---
title: Containerd ctr、crictl、nerdctl 客户端命令
tags: Kubernetes, Containerd, Docker
author: Laeni
date: '2022-12-11'
updated: '2022-12-11'
---

由于 [Containerd](https://github.com/containerd/containerd) 已经从 Docker 独立，这代表 Docker 底层也是使用的是 Containerd。而 Containerd 是实现了 CRI 规范的，所以 k8s 是可以直接使用 Containerd 而无需再使用 Docker 的。但是大部分时候我们还是使用 Docker 作为 k8s 的容器运行时，原因是因为我们习惯了 Docker，如果换为 Containerd 必定要使用新的工具来替换 Docker，而这些命令就是 ctr、crictl 等（不同的运行时会有一定差异）。

# 关系

**ctr：**[Containerd](https://github.com/containerd/containerd)自带的客户端工具，安装 Containerd 的同时就会安装 ctr（目前一起安装的工具为：`containerd`、`containerd-shim`、`containerd-shim-runc-v1`、`containerd-shim-runc-v2`、`containerd-stress`、`ctr`），如果使用其他容器运行时则会对应其他工具。

**crictl**：Kubelet 容器运行时接口 （CRI） 的 CLI 和验证工具，是[cri-tools](https://github.com/kubernetes-sigs/cri-tools)的别名。也就是说只要时实现了 CRI 规范的容器运行时都能使用它，一般安装 k8s 就会安装（手动安装）该工具，目前如果使用**kubeadm**初始化 k8s 集群前必须要先安装该工具。

**nerdctl**：[nerdctl](https://github.com/containerd/nerdctl)是[containerd组](https://github.com/containerd)中的一个子项目，目的是为了兼容 Docker CTL，即可以使用 nerdctl 代替 docker 命令，它只支持 containerd 容器运行时。

## 与 Docker 命令对比

|           命令           |  docker / nerdctl  |      ctr（containerd）       | crictl（kubernetes） |
| :----------------------: | :----------------: | :--------------------------: | :------------------: |
|      查看运行的容器      |     docker ps      | ctr task ls/ctr container ls |      crictl ps       |
|         查看镜像         |   docker images    |         ctr image ls         |    crictl images     |
|       查看容器日志       |    docker logs     |              无              |     crictl logs      |
|     查看容器数据信息     |   docker inspect   |      ctr container info      |    crictl inspect    |
|       查看容器资源       |    docker stats    |              无              |     crictl stats     |
|   启动/关闭已有的容器    | docker start/stop  |     ctr task start/kill      |  crictl start/stop   |
|     运行一个新的容器     |     docker run     |           ctr run            | 无（最小单元为 pod） |
|          打标签          |     docker tag     |        ctr image tag         |          无          |
|     创建一个新的容器     |   docker create    |     ctr container create     |    crictl create     |
|         导入镜像         |    docker load     |       ctr image import       |          无          |
|         导出镜像         |    docker save     |       ctr image export       |          无          |
|         删除容器         |     docker rm      |       ctr container rm       |      crictl rm       |
|         删除镜像         |     docker rmi     |         ctr image rm         |      crictl rmi      |
|         拉取镜像         |    docker pull     |        ctr image pull        |     ctictl pull      |
|         推送镜像         |    docker push     |        ctr image push        |          无          |
| 登录或在容器内部执行命令 |    docker exec     |              无              |     crictl exec      |
|      清空不用的镜像      | docker image prune |              无              |  crictl rmi --prune  |

# crictl

## Help

```sh
$ crictl help
NAME:
   crictl - client for CRI

USAGE:
   crictl [global options] command [command options] [arguments...]

VERSION:
   v1.25.0

COMMANDS:
   attach              附加到正在运行的容器 | Attach to a running container
   create              创建新容器 | Create a new container
   exec                在正在运行的容器中运行命令 | Run a command in a running container
   version             显示运行时版本信息 | Display runtime version information
   images, image, img  列出镜像 | List images
   inspect             显示一个或多个容器的状态 | Display the status of one or more containers
   inspecti            返回一个或多个镜像的状态 | Return the status of one or more images
   imagefsinfo         返回镜像文件系统信息 | Return image filesystem info
   inspectp            显示一个或多个 Pod 的状态 | Display the status of one or more pods
   logs                获取容器的日志 | Fetch the logs of a container
   port-forward        将本地端口转发到 Pod | Forward local port to a pod
   ps                  列出容器 | List containers
   pull                从注册表中拉取镜像 | Pull an image from a registry
   run                 在沙箱中运行一个新容器 | Run a new container inside a sandbox
   runp                运行一个新的 Pod | Run a new pod
   rm                  删除一个或多个容器 | Remove one or more containers
   rmi                 删除一个或多个镜像 | Remove one or more images
   rmp                 删除一个或多个 Pod | Remove one or more pods
   pods                列出 Pod | List pods
   start               启动一个或多个已创建的容器 | Start one or more created containers
   info                显示容器运行时信息 | Display information of the container runtime
   stop                停止一个或多个容器 | Stop one or more running containers
   stopp               停止一个或多个 Pod | Stop one or more running pods
   update              更新一个或多个正在运行的容器 | Update one or more running containers
   config              获取和设置 crictl 客户端配置选项 | Get and set crictl client configuration options
   stats               列出容器资源使用统计信息 | List container(s) resource usage statistics
   statsp              列出 Pod 资源使用统计信息 | List pod resource usage statistics
   completion          输出shell完成代码（Shell自动提示/补全） | Output shell completion code
   checkpoint          检查一个或多个正在运行的容器 | Checkpoint one or more running containers
   help, h             显示命令列表或一个命令的帮助 | Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config value, -c value            客户端配置文件的位置。 如果未指定且默认不存在，也会搜索程序的目录 | Location of the client config file. If not specified and the default does not exist, the program's directory is searched as well (default: "/etc/crictl.yaml") [$CRI_CONFIG_FILE]
   --debug, -D                         Enable debug mode (default: false)
   --help, -h                          show help (default: false)
   --image-endpoint value, -i value    CRI 镜像管理器服务的端点（默认：使用'runtime-endpoint'设置） | Endpoint of CRI image manager service (default: uses 'runtime-endpoint' setting) [$IMAGE_SERVICE_ENDPOINT]
   --runtime-endpoint value, -r value  CRI 容器运行时服务的端点（默认：按默认列表中的顺序使用第一个成功的）。现在不推荐使用默认值，应该明确设置。 | Endpoint of CRI container runtime service (default: uses in order the first successful one of [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]). Default is now deprecated and the endpoint should be set instead. [$CONTAINER_RUNTIME_ENDPOINT]
   --timeout value, -t value           以秒为单位连接到服务器的超时时间。| Timeout of connecting to the server in seconds (e.g. 2s, 20s.). 0 or less is set to default (default: 2s)
   --version, -v                       print the version (default: false)
```

### help runp

```sh
$ crictl help runp
NAME:
   crictl runp - Run a new pod

USAGE:
   crictl runp [command options] pod-config.[json|yaml]

OPTIONS:
   --cancel-timeout value, -T value  在取消请求之前等待运行 pod 沙箱请求完成的秒数 | Seconds to wait for a run pod sandbox request to complete before cancelling the request (default: 0s)
   --runtime value, -r value         要使用的运行时处理程序。 可用选项由容器运行时定义。 | Runtime handler to use. Available options are defined by the container runtime.
```

## 示例

### 配置

如设置运行时端点：

```sh
$ crictl config runtime-endpoint unix:///run/containerd/containerd.sock
```

> 该操作会将配置内容存储到`/etc/crictl.yaml`，所以可以直接修改该配置文件。

### Pod

#### 查看所有Pod

```sh
$ crictl pods
```

#### 停止Pod

```sh
$ crictl stopp <POD-ID>
```

#### 删除Pod

```sh
$ crictl rmp <POD-ID> # 只能删除已停止的POD
$ crictl rmp -f <POD-ID> # 即时POD未停止也强制删除
$ crictl rmp -a <POD-ID> # 删除全部POD
```

> 删除POD会自动删除POD中的容器。

#### 创建POD

```sh
$ cat <<EOF | tee pod-config.json
{
    "metadata": {
        "name": "nginx-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "logDirectory": "/tmp",
    "linux": {
    }
}
EOF
$ crictl runp pod-config.json
```

在POD中创建容器

```sh
$ cat <<EOF | tee pod-config.json
{
  "metadata": {
      "name": "busybox"
  },
  "image":{
      "image": "busybox"
  },
  "command": [
      "top"
  ],
  "log_path":"busybox.log",
  "linux": {
  }
}
EOF
$ crictl create <POD-ID> container-config.json pod-config.json
```

### 容器

#### 列出所有容器

```sh
$ crictl ps -a
```

> 类似*Docker*：`docker ps -a`

#### 查看容器日志

```sh
$ crictl logs [command options] <CONTAINER-ID>
```

# 参考资料

- <https://cdn.modb.pro/db/485911>
