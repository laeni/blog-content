---
title: '使用Kubeadm安装k8s'
author: 'Laeni'
tags: 'k8s, kubeadm'
date: '2022-10-29'
updated: '2022-10-29'
---

# 安装前准备

## 安装kubelet

kubelet安装可参考[K8s官方文档](https://kubernetes.io/zh-cn/docs/tasks/tools/#kubectl)。

> 如果是采用二进制方式安装的情况，需要手动添加`kubelet.service`服务文件并按实际要求配置，如果是kubeadm工具使用，则可以参考[官网例子](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/kubelet-integration/#the-kubelet-drop-in-file-for-systemd)。
>
> ```sh
> $ cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
> [Unit]
> Description=Kubernetes Kubelet Server
> Documentation=https://github.com/GoogleCloudPlatform/kubernetes
> After=network-online.target
> After=network-online.target
> 
> [Service]
> User=root
> # 用于 TLS 引导程序的 KubeConfig 文件为 /etc/kubernetes/bootstrap-kubelet.conf（一般采用 kubeadm init 生成的都没有该文件）， 但仅当 /etc/kubernetes/kubelet.conf 不存在时才能使用，即两个文件存在其中一个即可，如果都存在，则使用后者。
> Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
> Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
> # 这是 "kubeadm init" 和 "kubeadm join" 运行时生成的文件，
> # 动态地填充 KUBELET_KUBEADM_ARGS 变量
> EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
> # 这是一个文件，用户在不得已下可以将其用作替代 kubelet args（对于 DEB：/etc/default/kubelet , 对于 RPM: /etc/sysconfig/kubelet），KUBELET_EXTRA_ARGS 应该从此文件中获取。
> # 用户最好使用 .NodeRegistration.KubeletExtraArgs 对象在配置文件中替代。
> # KUBELET_EXTRA_ARGS 在标志链中排在最后，并且在设置冲突时具有最高优先级。
> EnvironmentFile=-/etc/default/kubelet
> ExecStart=/usr/local/bin/kubelet \$KUBELET_OPTS \$KUBELET_KUBECONFIG_ARGS \$KUBELET_CONFIG_ARGS \$KUBELET_KUBEADM_ARGS \$KUBELET_EXTRA_ARGS
> Restart=always
> RestartSec=10s
> 
> [Install]
> WantedBy=multi-user.target
> EOF
> ```

## 创建kubeadm配置文件

kubeadm配置文件分为`init`配置（通过`kubeadm config print init-defaults`获得默认配置）和`join`配置（通过`kubeadm config print join-defaults`获得默认配置），一般会优先取集群 kube-system 命名空间中名为"kubeadm-config"的 ConfigMap 配置，如果集群无法访问则使用默认配置（如果明确通过`--config`选项指定了配置的情况下还会不会去集群读取配置暂时未知）。

### kubeadm-init.yaml

新版本中，该配置用完之后会自动上传到集群中保存起来以便后用，如果老版本没有自动上传的需要手动上传。

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.10.1.2 # 集群公告地址 default: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: node-pc-ubuntu # node名称，如有需要可以进行更改
  taints: [] # 如果需要在主节点运行其他pod则需要去除"污点". default: null
---
apiServer:
  timeoutForControlPlane: 4m0s
  extraArgs:
    # 由于这里使用外部共享Etcd共享存储，并且为了数据污染，序言指定前缀. 默认值："/registry"
    etcd-prefix: /k8s/registry-ubuntu
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
# 由于是笔记本搭建，想尽可能少的安装服务，所以这里Etcd使用外部共享的
etcd:
  #local:
  #  dataDir: /var/lib/etcd
  external:
    endpoints:
      - https://10.10.1.1:2379
    caFile: /home/laeni/Desktop/NFS/archive/cert/etcd/ca.crt
    certFile: /home/laeni/Desktop/NFS/archive/cert/etcd/client-k8s-pc-ubuntu.crt
    keyFile: /home/laeni/Desktop/NFS/archive/cert/etcd/client-k8s-pc-ubuntu.key
imageRepository: registry.k8s.io # 默认情况下，大陆地区需要梯子才能访问，或者还成其他内国源也行
kind: ClusterConfiguration
kubernetesVersion: 1.25.3
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.1.0.0/16 # default: 10.96.0.0/12
  podSubnet: 10.2.0.0/16 # clusterCIDR.最好明确指定pod网段 default: 未定义
scheduler: {}
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
authentication:
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
cgroupDriver: systemd
failSwapOn: false # 如果不是生产环境，一般都都会开启交换分区，所以需要指定开启Swap时不失败退出
staticPodPath: /etc/kubernetes/manifests
```

## 启用 Shell 自动完成

### Bash

在`.bashrc`中添加`source <(kubeadm completion bash)`

### Zsh

在`.zshrc`中添加`source <(kubeadm completion zsh)`

## 根据`init phase preflight`提示安装缺少的工具

`sudo kubeadm --config .. init phase preflight`

- [ERROR FileExisting-crictl]: crictl not found in system path

  kubeadm 使用 crictl 操作容器，所以需要安装它，安装方法参见[官网](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)：

  ```sh
  $ VERSION="v1.25.0"
  $ curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-${VERSION}-linux-amd64.tar.gz --output crictl-${VERSION}-linux-amd64.tar.gz
  $ sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
  $ rm -f crictl-$VERSION-linux-amd64.tar.gz
  ```

- [ERROR FileExisting-conntrack]: conntrack not found in system path

  ```sh
  $ sudo apt install conntrack
  ```

- [WARNING FileExisting-ethtool]: ethtool not found in system path

  ```sh
  $ sudo apt install ethtool
  ```

- [WARNING FileExisting-socat]: socat not found in system path

  ```sh
  $ sudo apt install socat
  ```

- hostname "xxx" could not be reached

  编辑'/etc/hosts'bingham添加本机IP地址解析

# 初始化集群示例

```sh
$ # kubeadm init --config kubeadm-init.yaml 
[init] Using Kubernetes version: v1.25.3
[preflight] Running pre-flight checks
	[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
	[WARNING SystemVerification]: missing optional cgroups: blkio
	[WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local node-pc-ubuntu] and IPs [10.96.0.1 10.10.1.2]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] External etcd mode: Skipping etcd/ca certificate authority generation
[certs] External etcd mode: Skipping etcd/server certificate generation
[certs] External etcd mode: Skipping etcd/peer certificate generation
[certs] External etcd mode: Skipping etcd/healthcheck-client certificate generation
[certs] External etcd mode: Skipping apiserver-etcd-client certificate generation
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 23.528051 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node node-pc-ubuntu as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[bootstrap-token] Using token: abcdef.0123456789abcdef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.10.1.2:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:d93d99bec77ab9d1ab842cc54f80471a9b3488fe29fea2376f7100d4e503fa77
```

# Help

官方文档：<https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/>

```sh
$ kubeadm help

    ┌──────────────────────────────────────────────────────────┐
    │ KUBEADM                                                  │
    │ 轻松引导安全的 Kubernetes 集群                              │
    │                                                          │
    │ Please give us feedback at:                              │
    │ https://github.com/kubernetes/kubeadm/issues             │
    └──────────────────────────────────────────────────────────┘

Example usage:

    创建一个包含一个控制平面节点（控制集群）和一个工作节点（您的工作负载，如 Pod 和 Deployments 运行的地方）的两台机器集群。

    ┌──────────────────────────────────────────────────────────┐
    │ On the first machine:                                    │
    ├──────────────────────────────────────────────────────────┤
    │ control-plane# kubeadm init                              │
    └──────────────────────────────────────────────────────────┘

    ┌──────────────────────────────────────────────────────────┐
    │ On the second machine:                                   │
    ├──────────────────────────────────────────────────────────┤
    │ worker# kubeadm join <arguments-returned-from-init>      │
    └──────────────────────────────────────────────────────────┘

    然后，您可以在任意数量的其他机器上重复第二步。

Usage:
  kubeadm [command]

Available Commands:
  certs       与处理 Kubernetes 证书相关的命令
  completion  输出指定 shell（bash 或 zsh）的 shell 完成代码
  config      管理持久在集群中的 ConfigMap 中的 kubeadm 集群的配置
  help        关于任何命令的帮助
  init        运行此命令以设置 Kubernetes 控制平面
  join        在您希望加入现有集群的任何机器上运行它
  kubeconfig  Kubeconfig 文件工具
  reset       尽最大努力还原“kubeadm init”或“kubeadm join”对此主机所做的更改
  token       管理引导令牌
  upgrade     使用此命令将您的集群顺利升级到更新版本
  version     Print the version of kubeadm

Flags:
  -h, --help                     help for kubeadm

Global Flags: # 实际上该命令中没有 Global Flags ，但是子命令基本都有，所以这里提出来，并且子命令中不再重复
      --add-dir-header           如果为 true，则将文件目录添加到日志消息的标题中
      --log-file string          如果非空，则使用此日志文件（-logtostderr=true 时无效）
      --log-file-max-size uint   定义日志文件可以增长到的最大大小（-logtostderr=true 时无效）。 单位是兆字节。 如果值为 0，则最大文件大小不受限制。 （默认 1800）
      --one-output               如果为 true，则仅将日志写入其本机严重性级别（vs 同时写入每个较低严重性级别；当 -logtostderr=true 时无效）
      --rootfs string            [EXPERIMENTAL] “真实”主机根文件系统的路径。
      --skip-headers             如果为 true，避免在日志消息中使用标头前缀
      --skip-log-headers         如果为 true，则在打开日志文件时避免使用标头（-logtostderr=true 时无效）
  -v, --v Level                  日志级别详细程度的数字

Additional help topics:
  kubeadm alpha      Kubeadm 实验性子命令

Use "kubeadm [command] --help" for more information about a command.
```

## certs

```sh
$ kubeadm help certs

Commands related to handling kubernetes certificates

Usage:
  kubeadm certs [command]

Aliases:
  certs, certificates

Available Commands:
  certificate-key  生成证书密钥
  check-expiration 检查 Kubernetes 集群的证书是否过期
  generate-csr     生成密钥和证书签名请求
  renew            为 Kubernetes 集群续订证书

Flags:
  -h, --help   help for certs

Use "kubeadm certs [command] --help" for more information about a command.
```

## config

```sh
$ kubeadm help config

在 kube-system 命名空间中有一个名为"kubeadm-config"的 ConfigMap，kubeadm 使用它来存储有关集群的内部配置。
kubeadm CLI v1.8.0+ 会使用与'kubeadm init'一起使用的配置自动创建此 ConfigMap，但如果您使用 kubeadm v1.7.x 或更低版本初始化集群，则必须使用'config upload'命令来创建此 ConfigMap。 这是必需的，以便'kubeadm upgrade'可以正确配置升级后的集群。

Usage:
  kubeadm config [flags]
  kubeadm config [command]

Available Commands:
  images      与 kubeadm 使用的容器镜像交互
  migrate     从文件中读取 API 类型的 kubeadm 旧版本配置，并转换为新版本输出
  print       Print configuration

Flags:
  -h, --help                help for config
      --kubeconfig string   与集群通信时使用的 kubeconfig 文件。 如果未设置该标志，则可以在一组标准位置中搜索现有的 kubeconfig 文件。 (default "/etc/kubernetes/admin.conf")

Use "kubeadm config [command] --help" for more information about a command.
```

### print

```sh
$ kubeadm help config print

此命令打印提供的子命令的配置。
For details, see: https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#section-directories

Usage:
  kubeadm config print [flags]
  kubeadm config print [command]

Available Commands:
  init-defaults 打印默认 init 配置，可用于 'kubeadm init'
  join-defaults 打印默认 join 配置，可用于 'kubeadm join'

Flags:
  -h, --help   help for print

Use "kubeadm config print [command] --help" for more information about a command.
```

> 如果需要查看特定结构的默认值可以使用类似命令：`kubeadm config print init-defaults --component-configs KubeletConfiguration`。

## init

````sh
$ sudo kubeadm help init
运行此命令以设置 Kubernetes 控制平面

init 命令执行以下阶段:
```
preflight                    正式开始前进行环境检查
certs                        Certificate generation
  /ca                          生成自签名 Kubernetes CA 以为其他 Kubernetes 组件提供身份验证（如果已存在则跳过）
  /apiserver                   生成 apiserver 的证书
  /apiserver-kubelet-client    生成 apiserver 连接 kubelet 的证书
  /front-proxy-ca              生成自签名 CA 为前端代理提供身份验证（如果已存在则跳过）
  /front-proxy-client          为前端代理客户端生成证书
  /etcd-ca                     生成自签名 CA 为 etcd 提供身份验证（如果已存在则跳过）
  /etcd-server                 生成 etcd 服务端证书
  /etcd-peer                   生成 etcd 对等节点（节点相互通信）的证书
  /etcd-healthcheck-client     生成 etcd 健康检查证书
  /apiserver-etcd-client       生成 apiserver 用来访问 etcd 的证书
  /sa                          生成用于签署服务帐户令牌的私钥及其公钥
kubeconfig                   生成所有控制平面和 admin 的 kubeconfig 配置文件
  /admin                       为管理员和 kubeadm 本身生成一个 kubeconfig 文件
  /kubelet                     为 kubelet 生成一个 kubeconfig 文件，仅（*only*）用于集群引导
  /controller-manager          生成一个供控制器管理器使用的 kubeconfig 文件
  /scheduler                   生成一个供调度程序使用的 kubeconfig 文件
kubelet-start                编写 kubelet 设置并（重新）启动 kubelet
control-plane                生成创建控制平面所需的所有静态 Pod 清单文件
  /apiserver                   生成 kube-apiserver 静态 Pod 清单
  /controller-manager          生成 kube-controller-manager 静态 Pod 清单
  /scheduler                   生成 kube-scheduler 静态 Pod 清单
etcd                         为本地 etcd 生成静态 Pod 清单文件
  /local                       为本地单节点本地 etcd 实例生成静态 Pod 清单文件
upload-config                将 kubeadm 和 kubelet 配置上传到 ConfigMap
  /kubeadm                     将 kubeadm ClusterConfiguration 上传到 ConfigMap
  /kubelet                     将 kubelet 组件配置上传到 ConfigMap
upload-certs                 将证书上传到 kubeadm-certs
mark-control-plane           将节点标记为控制平面
bootstrap-token              生成用于将节点加入集群的引导令牌
kubelet-finalize             在 TLS 引导后更新与 kubelet 相关的设置
  /experimental-cert-rotation  启用 kubelet 客户端证书轮换
addon                        安装所需的插件以通过一致性测试
  /coredns                     将 CoreDNS 插件安装到 Kubernetes 集群
  /kube-proxy                  将 kube-proxy 插件安装到 Kubernetes 集群
```

Usage:
  kubeadm init [flags]
  kubeadm init [command]

Available Commands:
  phase       使用此命令调用初始化工作流的单个阶段

# 以下选项一般通过配置文件完成
Flags:
      --apiserver-advertise-address string   The IP address the API Server will advertise it's listening on. If not set the default network interface will be used.
      --apiserver-bind-port int32            Port for the API Server to bind to. (default 6443)
      --apiserver-cert-extra-sans strings    Optional extra Subject Alternative Names (SANs) to use for the API Server serving certificate. Can be both IP addresses and DNS names.
      --cert-dir string                      The path where to save and store the certificates. (default "/etc/kubernetes/pki")
      --certificate-key string               Key used to encrypt the control-plane certificates in the kubeadm-certs Secret.
      --config string                        Path to a kubeadm configuration file.
      --control-plane-endpoint string        Specify a stable IP address or DNS name for the control plane.
      --cri-socket string                    Path to the CRI socket to connect. If empty kubeadm will try to auto-detect this value; use this option only if you have more than one CRI installed or if you have non-standard CRI socket.
      --dry-run                              Don't apply any changes; just output what would be done.
      --feature-gates string                 A set of key=value pairs that describe feature gates for various features. Options are:
                                             PublicKeysECDSA=true|false (ALPHA - default=false)
                                             RootlessControlPlane=true|false (ALPHA - default=false)
                                             UnversionedKubeletConfigMap=true|false (default=true)
  -h, --help                                 help for init
      --ignore-preflight-errors strings      A list of checks whose errors will be shown as warnings. Example: 'IsPrivilegedUser,Swap'. Value 'all' ignores errors from all checks.
      --image-repository string              Choose a container registry to pull control plane images from (default "registry.k8s.io")
      --kubernetes-version string            Choose a specific Kubernetes version for the control plane. (default "stable-1")
      --node-name string                     Specify the node name.
      --patches string                       Path to a directory that contains files named "target[suffix][+patchtype].extension". For example, "kube-apiserver0+merge.yaml" or just "etcd.json". "target" can be one of "kube-apiserver", "kube-controller-manager", "kube-scheduler", "etcd", "kubeletconfiguration". "patchtype" can be one of "strategic", "merge" or "json" and they match the patch formats supported by kubectl. The default "patchtype" is "strategic". "extension" must be either "json" or "yaml". "suffix" is an optional string that can be used to determine which patches are applied first alpha-numerically.
      --pod-network-cidr string              Specify range of IP addresses for the pod network. If set, the control plane will automatically allocate CIDRs for every node.
      --service-cidr string                  Use alternative range of IP address for service VIPs. (default "10.96.0.0/12")
      --service-dns-domain string            Use alternative domain for services, e.g. "myorg.internal". (default "cluster.local")
      --skip-certificate-key-print           Don't print the key used to encrypt the control-plane certificates.
      --skip-phases strings                  List of phases to be skipped
      --skip-token-print                     Skip printing of the default bootstrap token generated by 'kubeadm init'.
      --token string                         The token to use for establishing bidirectional trust between nodes and control-plane nodes. The format is [a-z0-9]{6}\.[a-z0-9]{16} - e.g. abcdef.0123456789abcdef
      --token-ttl duration                   The duration before the token is automatically deleted (e.g. 1s, 2m, 3h). If set to '0', the token will never expire (default 24h0m0s)
      --upload-certs                         Upload control-plane certificates to the kubeadm-certs Secret.

Use "kubeadm init [command] --help" for more information about a command.
````

