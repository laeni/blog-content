--二进制安装k8s--



安装前，需要先安装Etcd，参见[Etcd 帮助文档](/note/db/etcd)。

## 规划

| 名称        | Value                 | 备注                      |
| ----------- | --------------------- | ------------------------- |
| Etcd地址    | http://10.10.1.1:2379 |                           |
| Node(单机)  | 10.0.0.1              |                           |
| Service网段 | 10.1.0.0/24           |                           |
| Pod网段     | 10.2.0.0/24           |                           |
| 集群DNS     | 10.1.0.2              | 必须为Service网段中的一个 |

## 生成证书及用户配置文件

生成参见[生成K8s所需的证书和密钥及用户配置文件](/node/k8s/gen-k8s-cert)。

先生成用户配置文件的原因是可以及时进行测试，不用全部操作完了才进行验证。

## 部署Etcd

由于k8s使用Etcd存储数据，所以需要先部署Etcd。Etcd如果开启认证，则需要在Etcd中创建`apiserver`使用的证书`CN`对应的用户并授权（一般将其添加到`root`组）。

部署成功后，需要确保后面用于启动`apiserver`的证书密钥能正常使用Etcd（可以使用`etcdctl`验证，因为`apiserver`也是Etcd的一个客户端）。

Etcd不一定要部署在Master上，只要能访问就行。

## 下载二进制文件

清单如下表，根据下表可知，一般下载Server即可，除非确实不需要才考虑两个之一仅仅下载其他。

| Server                                                       | Node                                                         | Client                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------- |
| apiextensions-apiserver<br/>kubeadm<br/>kube-aggregator<br/>kube-apiserver<br/>kube-apiserver.docker_tag<br/>kube-apiserver.tar<br/>kube-controller-manager<br/>kube-controller-manager.docker_tag<br/>kube-controller-manager.tar<br/>kubectl<br/>kubectl-convert<br/>kubelet<br/>kube-log-runner<br/>kube-proxy<br/>kube-proxy.docker_tag<br/>kube-proxy.tar<br/>kube-scheduler<br/>kube-scheduler.docker_tag<br/>kube-scheduler.tar<br/>mounter | kubeadm<br/>kubectl<br/>kubectl-convert<br/>kubelet<br/>kube-log-runner<br/>kube-prox | kubectl<br />kubectl-convert |

下载后将可执行文件复制或链接到`/usr/local/bin/`。

## Kubernetes control plane

一般来说，控制平面所在的那个节点叫做`Master`，该节点可以不用`kubelet`，甚至可以没有容器环境。

### 部署kube-apiserver

```shell
$ cat <<EOF > start-kube-apiserver.sh
kube-apiserver \\
--bind-address=0.0.0.0 `# 监听地址,这里监听全部地址，如果需要更高安全性可以只监听maste节点ip地址`\\
--secure-port=6443 `# https安全端口`\\
--advertise-address=10.0.0.1 `# 集群通告地址，即告诉其他客户端自己的地址，实际上可以不一定用该地址`\\
--allow-privileged=true `# 是否启用授权`\\
--service-cluster-ip-range=10.1.0.0/24 `# Service虚拟IP地址段`\\
--enable-admission-plugins=NodeRestriction `# 准入控制模块`\\
--authorization-mode=RBAC,Node `# 认证授权，启用RBAC授权和节点自管理`\\
--enable-bootstrap-token-auth=true `# 是否启用TLS bootstrap机制`\\
--token-auth-file=/opt/kubernetes/cfg/token.csv `# 如果设置，该文件将用于通过令牌方式验证客户端。bootstrap token文件`\\
--service-node-port-range=30000-32767 `# Service nodeport类型默认分配端口范围`\\
--kubelet-client-certificate=/etc/kubernetes/pki/apiserver.crt `# apiserver访问kubelet客户端证书`\\
--kubelet-client-key=/etc/kubernetes/pki/apiserver.key `# apiserver访问kubelet客户端证书私钥`\\
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt  `# apiserver https证书`\\
--tls-private-key-file=/etc/kubernetes/pki/apiserver.key `# apiserver https证书私钥`\\
--client-ca-file=/etc/kubernetes/pki/ca.crt \\
--service-account-key-file=/etc/kubernetes/pki/ca.key \\
--service-account-issuer=api `# 1.20+版本必须加的参数`\\
--service-account-signing-key-file=/etc/kubernetes/pki/ca.key `# 1.20+版本必须加的参数`\\
--etcd-servers=https://10.10.1.1:2379 `# etcd集群地址`\\
--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt `# Etcd集群CA证书`\\
--etcd-certfile=/etc/kubernetes/pki/etcd/server.crt `# Etcd集群服务端证书`\\
--etcd-keyfile=/etc/kubernetes/pki/etcd/server.key `# Etcd集群服务端证书私钥`\\
--requestheader-client-ca-file=/etc/kubernetes/pki/ca.crt `# 启动聚合层相关配置`\\
--proxy-client-cert-file=/etc/kubernetes/pki/apiserver.crt `# 启动聚合层相关配置`\\
--proxy-client-key-file=/etc/kubernetes/pki/apiserver.key `# 启动聚合层相关配置`\\
--requestheader-allowed-names=kubernetes `# 启动聚合层相关配置`\\
--requestheader-extra-headers-prefix=X-Remote-Extra- `# 启动聚合层相关配置`\\
--requestheader-group-headers=X-Remote-Group `# 启动聚合层相关配置`\\
--requestheader-username-headers=X-Remote-User `# 启动聚合层相关配置`\\
--enable-aggregator-routing=true `# 启动聚合层相关配置`\\
--audit-log-maxage=30 `# 审计日志相关`\\
--audit-log-maxbackup=3 `# 审计日志相关`\\
--audit-log-maxsize=100 `# 审计日志相关`\\
--audit-log-path=/opt/kubernetes/logs/k8s-audit.log `# 审计日志相关`
EOF
```

验证`apiserver`是否正常：

```shell
$ kubectl \
  --kubeconfig /etc/kubernetes/admin.conf `# 该配置文件为前面生成证书的时候生成的`\
  get ns
NAME              STATUS        AGE
default           Active        1h
kube-node-lease   Active        1h
kube-public       Active        1h
kube-system       Active        1h
```

> 正常输出命令空间则说明`apiserver`部署成功，否则先解决该问题再进行后面操作，因为后面操作全部需要和`apiserver`通信。

### 部署kube-controller-manager

```shell
$ cat << EOF > start-kube-controller-manager.sh
kube-controller-manager \\
--leader-elect `# 当该组件启动多个时，自动选举（HA）`\\
--kubeconfig=/etc/kubernetes/controller-manager.conf `# 连接apiserver配置文件`\\
--bind-address=0.0.0.0 `# 监听--secure-port端口的IP地址(default 0.0.0.0)`\\
--allocate-node-cidrs=true `# 是否应在云提供商上分配和设置Pod的CIDR`\\
--cluster-cidr=10.2.0.0/24 `# 集群中Pod的CIDR范围，要求--allocate-node-cidrs为true`\\
--service-cluster-ip-range=10.0.0.0/24 `# 集群service的cidr范围，需要--allocate-node-cidrs设置为true`\\
--cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt `# 用于颁发集群范围的证书。 如果指定，则不能指定更具体的 --cluster-signing-* 标志`\\
--cluster-signing-key-file=/etc/kubernetes/pki/ca.key  `# --cluster-signing-cert-file CA证书对应密钥`\\
--root-ca-file=/etc/kubernetes/pki/ca.crt `# 根CA，可以制定签发 --cluster-signing-cert-file 的CA，也可以与之相同`\\
--service-account-private-key-file=/etc/kubernetes/pki/ca.key `# 用于签署服务帐户令牌`\\
--use-service-account-credentials `# 为每个控制器使用单独的服务帐户凭据`\\
--cluster-signing-duration=87600h0m0s
EOF
```

验证组件是否正常

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
controller-manager   Healthy   ok                              
etcd-0               Healthy   {"health":"true","reason":""}
```

### 部署kube-scheduler

```shell
$ cat << EOF > start-kube-scheduler.sh
kube-scheduler \\
--bind-address=0.0.0.0 `# 侦听 --secure-port 端口的 IP 地址。 集群的其余部分和 CLI/Web 客户端必须可以访问关联的接口。 如果为空白或未指定地址（0.0.0.0 或 ::），将使用所有接口。 （默认 0.0.0.0）`\\
--leader-elect `# 当该组件启动多个时，自动选举（HA）`\\
--kubeconfig=/etc/kubernetes/scheduler.conf
EOF
```

验证组件是否正常

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
controller-manager   Healthy   ok                              
scheduler            Healthy   ok                              
etcd-0               Healthy   {"health":"true","reason":""}
```

## Worker节点

授权kubelet-bootstrap用户允许请求证书：

```shell
$ kubectl create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```

### 部署kubelet

#### 生成kubelet初次加入集群引导kubeconfig文件

`--bootstrap-kubeconfig`和`--kubeconfig`的区别：由于`--kubeconfig`配置文件中的密钥并不是所有节点通用的，因为证书的CN中包含了对应的主机名，这使得需要单独为每一个Worker节点创建对应的证书。而`--bootstrap-kubeconfig`则提供了一个临时连接`apiserver`的配置（里面使用了一个临时`token`），用于请求`apiserver`为当前节点颁发相应的证书，并将证书存储到`--kubeconfig`配置文件中。

```shell
$ KUBE_CONFIG="/opt/kubernetes/cfg/bootstrap.conf"

$ kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --embed-certs=true \
  --server=https://local.laeni.cn:6443 \
  --kubeconfig=${KUBE_CONFIG}
$ kubectl config set-credentials "kubelet-bootstrap" \
  --token=bc9fc609f131d003a49bd41ec43046d1 `# 与token.csv里保持一致`\
  --kubeconfig=${KUBE_CONFIG}
$ kubectl config set-context default \
  --cluster=kubernetes \
  --user="kubelet-bootstrap" \
  --kubeconfig=${KUBE_CONFIG}
$ kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 生成kubelet配置文件

虽然所有配置文件里的参数都可以通过启动时参数选项提供，但是目前[官方文档](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubelet-config-file/)建议通过配置文件的方式提供参数，因为这样可以简化节点部署和配置管理。命令行和配置文件都没有的则使用默认值，如果都提供则命令行优先级高于配置文件。

所有选项及其说明参见[官方文档](https://kubernetes.io/zh-cn/docs/reference/config-api/kubelet-config.v1beta1/)。

```shell
$ cat << EOF | sudo tee /opt/kubernetes/cfg/kubelet-config.yml
# see https://kubernetes.io/zh-cn/docs/reference/config-api/kubelet-config.v1beta1/
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0 # 允许访问的地址
port: 10250 # 监听的端口
readOnlyPort: 10255
serializeImagePulls: false # 以并行方式拉取镜像
clusterDNS: # 集群 DNS 服务器的 IP 地址的列表。 如果设置了，kubelet 将会配置所有容器使用这里的 IP 地址而不是宿主系统上的 DNS 服务器来完成 DNS 解析。
  - 10.1.0.2
clusterDomain: cluster.local 
failSwapOn: false # 节点上启用交换分区时是否拒绝启动
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt 
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi # 当可用内存低于 100Mi 时, kubelet 将会开始驱逐 Pod
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
runtimeRequestTimeout: 2m # 设置除长期运行的请求（pull、 logs、exec和attach）之外所有运行时请求的超时时长。默认值："2m"
# TODO 目前该选项报错，应该是格式不对
cgroupDriver: systemd # 操控宿主系统上控制组（CGroup） 的驱动程序（cgroupfs 或 systemd）。默认值："cgroupfs"
#featureGates: # 是一个从功能特性名称到布尔值的映射，用来启用或禁用实验性的功能。 此字段可逐条更改文件 "k8s.io/kubernetes/pkg/features/kube_features.go" 中所给的内置默认值。
#  IPv6DualStack: true
EOF
```

#### 启动kubelet

```shell
$ cat << EOF > start-kubelet.sh
kubelet \\
--hostname-override=local `# 部署kubelet节点的hostname名称，即对应的客户端证书的CN为"system:node:local"`\\
--kubeconfig=/etc/kubernetes/kubelet.conf `# 用于访问 apiserver 的配置，初次启动时，如果 --bootstrap-kubeconfig 可用，则本选项对应的配置文件可以不存在或为空（会自动生成）`\\
--bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.conf `# 用于首次启动且 --kubeconfig 不可用时使用，即与 --kubeconfig 必须要用一个可用`\\
--config=/opt/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/etc/kubernetes/pki \\
--container-runtime-endpoint=unix:///run/containerd/containerd.sock  \\
--node-labels=node.kubernetes.io/node=''
EOF
```

#### 批准kubelet证书申请并加入集群

只用`--kubeconfig`不可用且使用`--bootstrap-kubeconfig`创建时需要。

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf get csr
NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           REQUESTEDDURATION   CONDITION
node-csr-KLFsUNn9eyQ_JaQjwgZmeokMm7JwwheNyHKSoPIROz0   38s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Pending
$ kubectl certificate approve node-csr-KLFsUNn9eyQ_JaQjwgZmeokMm7JwwheNyHKSoPIROz0
```

> 本实例暂未通过`--bootstrap-kubeconfig`的方式实验成功，估计是token用法不对，而是直接手动生成kubelet用户配置文件启动的。

#### 验证节点是否成功加入

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf get node
NAME    STATUS   ROLES    AGE   VERSION
local   Ready    <none>   10h   v1.24.3
```

> 一般来讲，还没部署网络插件时，节点状态是`NotReady`，但是现在却是`Ready`，应该是它和控制平面在一台机器的原因。

### 部署kube-proxy

#### 生成配置文件

```shell
$ cat << EOF > /opt/kubernetes/cfg/kube-proxy-config.yml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /etc/kubernetes/kube-proxy.conf
hostnameOverride: local # 如果非空，将使用此字符串而不是实际的主机名作为标识
clusterCIDR: 10.2.0.0/24 # 集群中 Pod 的 CIDR 范围。配置后，从该范围之外发送到服务集群 IP 的流量被伪装，从 Pod 发送到外部 LoadBalancer IP 的流量将被重定向到相应的集群 IP。 对于双协议栈集群，接受一个逗号分隔的列表， 每个 IP 协议族（IPv4 和 IPv6）至少包含一个 CIDR。
# mode: ipvs
# ipvs:
#   scheduler: "rr"
iptables:
  masqueradeAll: true
EOF
```

#### 启动kube-proxy

```shell
$ cat << EOF > start-kube-proxy.sh
kube-proxy \\
--config=/opt/kubernetes/cfg/kube-proxy-config.yml
EOF
```

### 安装coredns

下载部署文件

```shell
$ curl -LO https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed
```

修改预留的地方（全大写的变量）如下，修改后去掉`.sed`后缀，表示已经修改完成

```
- kubernetes CLUSTER_DOMAIN REVERSE_CIDRS {
+ kubernetes cluster.local in-addr.arpa ip6.arpa {

# 改陈外部DNS服务，即非k8s本身的域名怎么解析
- forward . UPSTREAMNAMESERVER {
+ forward . /etc/resolv.conf {

# 目前直接删除
- }STUBDOMAINS
+ }

# DNS 集群地址，该地址为服务网段中的一个，与kubelet配置中的DNS地址一致
- clusterIP: CLUSTER_DNS_IP
+ clusterIP: 10.1.0.2
```

部署

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f coredns.yaml
```

### 授权apiserver访问kubelet

应用场景：例如kubectl logs

```shell
$ cat > apiserver-to-kubelet-rbac.yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
      - pods/log
    verbs:
      - "*"

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF

$ kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f apiserver-to-kubelet-rbac.yaml
clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet created
clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver created
```

### 删除master污点

```shell
# 查看污点
$ kubectl --kubeconfig /etc/kubernetes/admin.conf describe nodes <node-name> | grep Taints
```



### 安装网络插件（calico）

下载部署文件

```shell
$ curl -LO https://docs.projectcalico.org/manifests/calico.yaml
```

取消`CALICO_IPV4POOL_CIDR`注释，并将`alue`改为POD网段

```shell
vim +4434 calico.yaml
...
- name: CALICO_IPV4POOL_CIDR
  value: "10.2.0.0/24"
...
```

部署

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f calico.yaml
```



## 参考

[k8s 1.24.x 二进制部署最新版本](https://blog.51cto.com/flyfish225/5396121)

[PKI 证书和要求](https://kubernetes.io/zh-cn/docs/setup/best-practices/certificates/)
