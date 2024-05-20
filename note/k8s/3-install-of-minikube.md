# 快速上手

## 安装

参考[官方文档](https://minikube.sigs.k8s.io/docs/start/)安装对应系统和架构的`minikube`命令行工具。

```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

## 初始化集群

```sh
minikube start \
    --cert-expiration=87600h0m0s `# minikube 证书有效期，默认为三年（26280小时）。` \
    --apiserver-ips='' `# 一组在为 kubernetes 生成的证书中使用的 apiserver IP 地址。如果您希望将此 apiserver 设置为可从机器外部访问，则可以使用这组 apiserver IP 地址` \
    --apiserver-name='minikubeCA' `# 用于 apiserver 证书和连接的权威 apiserver 主机名。如果您希望使 apiserver 从计算机外部可用，可以使用此选项` \
    --apiserver-names='cluster.local' `# 一组在为 kubernetes 生成的证书中使用的 apiserver 名称。如果您希望将此 apiserver 设置为可从机器外部访问，则可以使用这组 apiserver 名称` \
    --apiserver-names='*.cluster.local' \
    --apiserver-names='*.laeni.cn' \
    --apiserver-port=8443 `# apiserver 侦听端口，默认值: 8443` \
    --container-runtime='' `# The container runtime to be used. Valid options: docker, cri-o, containerd (default: auto)` \
    --cpus='2' `# 分配给 Kubernetes 的 CPU 数量。 使用“max”可使用最大 CPU 数量。 使用“no-limit”不指定限制（仅限 Docker/Podman）` \
    --delete-on-failure=true `# 如果设置为 true，则在启动失败时删除当前群集，然后重试。默认为 false。` \
    --dns-domain='cluster.local' `# Kubernetes 集群中使用的集群 dns 域名` \
    --download-only=false `# 如果为 true，仅会下载和缓存文件以备后用 - 不会安装或启动任何项。` \
    --dry-run=false `# dry-run 模式。仅验证配置，不改变系统状态` \
    --image-mirror-country='cn' `# 需要使用的镜像镜像的国家/地区代码。留空以使用全球代码。对于中国大陆用户，请将其设置为 cn。` \
    --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' `# 用于从中拉取 docker 镜像的备选镜像存储库。如果您对 gcr.io 的访问受到限制，则可以使用该镜像存储库。将镜像存储库设置为“auto”可让 minikube 为您选择一个存储库。对于中国大陆用户，您可以使用本地 gcr.io 镜像，例如 registry.cn-hangzhou.aliyuncs.com/google_containers ` \
    --keep-context=true `# 这将保留现有 kubectl 上下文并创建 minikube 上下文。默认为 false` \
    --kubernetes-version='' `# minikube VM 将使用的 Kubernetes 版本（例如：v1.2.3、v1.30.0 的“stable”、v1.30.0 的“latest”）。 默认为“稳定”。` \
    --listen-address='0.0.0.0' `# 用于暴露端口的IP地址（仅适用于docker和podman驱动程序）` \
    --driver='docker' `# Driver is one of: virtualbox, kvm2, qemu2, qemu, vmware, none, docker, podman, ssh (defaults to auto-detect)` \
    --memory='2g' `# 分配给 Kubernetes 的 RAM 量（格式：<number>[<unit>]，其中unit = b、k、m 或 g）。 使用“max”可使用最大内存量。 使用“no-limit”不指定限制（仅限 Docker/Podman）` \
    `#--network=''` `# 运行 minikube 的网络。现在它被 docker/podman 和 KVM 驱动程序使用。如果留空，minikube 将创建一个新的网络。` \
    `#-n, --nodes=1` `# 要启动的节点总数。 默认为 1。` \
    `#-o, --output='text'` `# 标准输出的格式。可选项包括：[text,json]` \
    --service-cluster-ip-range='10.1.0.0/16' `# 需要用于服务集群 IP 的 CIDR。` \
    --subnet='10.2.0.0/16' `# 在 kic 集群上使用的子网。如果留空，minikube 将从 192.168.49.0 开始选择子网地址。（仅适用于 docker 和 podman 驱动程序）` \
    --static-ip='' `# 为 minikube 集群设置静态IP，该IP必须是私有IPv4地址，最后一位必须介于2和254之间，例如：192.168.200.200（仅适用于 Docker 和 Podman 驱动程序）` \
    --ports=[] `# 应该公开的端口列表（仅适用于 docker 和 podman 驱动）` \

    --addons=[] `# 启用插件。执行 'minikube addons list' 查看可用插件名称列表 ` \
    --cni='' `# 使用 CNI 插件。可选包括：auto、bridge、calico、cilium、flannel、kindnet 或 CNI 配置清单的路径（默认值：auto）` \
    --cri-socket='' `# 需要使用的 cri 套接字路径。` \
    --disable-driver-mounts=false `# 停用由管理程序提供的文件系统装载` \
    --disable-metrics=false `# 如果设置为 true，则禁用指标报告（CPU和内存使用率），这可以提高 CPU 利用率。默认为 false。` \
    --disable-optimizations=false `# 如果设置为 true，则禁用为本地 Kubernetes 做设置的优化，包括将 CoreDNS 副本数从2减少到1。默认值为false。` \
    --disk-size='20000mb' `# 分配给 minikube 虚拟机的磁盘大小（格式：<数字>[<单位>]，其中单位 = b、k、m 或 g）。` \
    --docker-env=[] `# 传递给 Docker 守护进程的环境变量。（格式：键值对）` \
    --docker-opt=[] `# 指定要传递给 Docker 守护进程的任意标志。（格式：key=value）` \
    --embed-certs=false `# 如果为 true，将在 kubeconfig 中嵌入证书。` \
    --extra-config='' `# 一组键=值对，描述可以传递给不同组件的配置。键应该是“.” 分开，点之前的第一部分是要应用配置的组件。有效组件有：kubelet、kubeadm、apiserver、controller-manager、etcd、proxy、scheduler。有效的 kubeadm 参数：ignore-preflight-errors、dry-run、kubeconfig、kubeconfig-dir、node-name、cri-socket、experimental-upload-certs、certificate-key、rootfs、skip-phases、pod-network-cidr` \
    --extra-disks=0 `# 创建并附加到 minikube VM 的额外磁盘数量（当前仅针对 hyperkit、kvm2 和 qemu2 驱动程序实现）` \
    --feature-gates='' `# 一组用于描述 alpha 版功能/实验性功能的功能限制的键值对。` \
    --force=false `# 强制 minikube 执行可能有风险的操作` \
    --force-systemd=false `# 如果设置为 true，则强制容器运行时使用 systemd 作为 cgroup 管理器。默认为false。` \
    -g, --gpus='' `# 所有 pods 使用您的英伟达 GPUs。选项包括:[all,nvidia](仅支持Docker容器运行时的Docker驱动程序)` \
    --ha=false `# 创建高度可用的多控制平面集群，至少包含三个控制平面节点，这些节点也将被标记为工作。` \
    --insecure-registry=[] `# 传递给 Docker 守护进程的不安全 Docker Registry。 系统会自动添加默认 service CIDR 范围。` \
    --install-addons=true `# 如果设置为 true，则安装插件。默认为true。` \
    --interactive=true `# 允许用户提示以获取更多信息` \
    --mount=false `# 这将启动挂载守护进程并将文件自动挂载到 minikube 中。` \
    --mount-9p-version='9p2000.L': `# 指定挂载应使用的 9p 版本` \
    --mount-gid='docker' `# 用于挂载默认的 group id` \
    --mount-ip='' `# 指定挂载应该设置的IP` \
    --mount-msize=262144 `# 用于 9p 数据包有效负载的字节数` \
    --mount-options=[] `# 其他挂载选项，例如：cache=fscache` \
    --mount-port=0 `# 指定应设置挂载的端口，其中 0 表示任何空闲端口。` \
    --mount-string='/root:/minikube-host': `# 传递 minikube mount 命令的参数。` \
    --mount-type='9p': `# 指定挂载文件系统类型（支持的类型：9p）` \
    --mount-uid='docker' `# 用于挂载默认的 user id` \
    --namespace='default' `# 启动后要激活的命名空间` \
    --native-ssh=true `# 使用原生的Golang SSH客户端（默认为true）。将其设置为 'false' 以在访问 Docker 机器时使用命令行的 'ssh' 命令。对于那些不以 'Waiting for SSH' 开头的机器驱动程序来说非常有用。` \
    --no-kubernetes=false `# 如果设置为 true，minikube虚拟机/容器将在不启动或配置Kubernetes的情况下启动。(只适用于新集群)` \
    --preload=true `# 如果设置为true，则在可用时下载预加载映像的tarball，以提高启动时间。默认为true。` \
    --registry-mirror=[] `# 传递给 Docker 守护进程的注册表镜像` \
    --trace='' `# 发送跟踪事件。包含的选项：[gcp]`
    --wait=[apiserver,system_pods] `# 启动集群后要验证和等待的 Kubernetes 组件的逗号分隔列表。 默认为“apiserver,system_pods”，可用选项：“apiserver,system_pods,default_sa,apps_running,node_ready,kubelet”。 其他可接受的值为 'all' or 'none', 'true' and 'false'` \
    --wait-timeout=6m0s `# Kubernetes 或主机正常运行前的最大等待时间。`
```

