---
title: '二进制方式安装K8s'
author: 'Laeni'
tags: 'k8s'
date: '2022-10-29'
updated: '2022-11-06'
---

# 前提条件

1. [安装容器运行时](./containerd)，这里使用*containerd*。

# 规划

```sh
NODE_NAME=pc-ubuntu # 当前节点主机名，如果有多个节点，需要保证唯一
NODE_HOST=10.10.1.2 # 当前节点 IP 地址
ETCD_PREFIX=/k8s/registry-$NODE_NAME # k8s 集群存储使用的前缀，后面可能需要使用共享 etcd，为避免冲突，这是指定单独的前缀。 default: /registry
SERVICE_CLUSTER_IP_RANGE=10.1.0.0/16 # Service网段 - service-cluster-ip-range
CLUSTER_CIDR=10.2.0.0/16 # Pod网段 - clusterCIDR
CLUSTER_DNS=10.1.0.53 # 集群DNS. 必须在 Service 网段内
APISERVER_SERVICE=10.1.0.1 # apiserver 服务IP，一般为网段的第一个

# 独立部署的 etcd 相关信息
ETCD_SERVERS=https://10.10.1.1:2379
ETCD_CAFILE=/mnt/share/archive/cert/etcd/ca.crt
ETCD_CERTFILE=/mnt/share/archive/cert/etcd/client-k8s-pc-ubuntu.crt
ETCD_KEYFILE=/mnt/share/archive/cert/etcd/client-k8s-pc-ubuntu.key

# 其他
CERT=/mnt/share/archive/cert # 全局根CA（global_root_ca.crt）所在路径
```

> 这里的值需要根据实际情况修改i并提前确定好。

> 此外，如果是笔记本安装的话，建议创建一个网桥，并使用网桥的 IP 地址作为本机地址，这样做的目的是确保 IP 是固定的，否则更换了网络环境将导致集群不可用。这里直接使用本机的[WireGuard](https://www.wireguard.com/)节点IP也能达到同样的效果。

# 生成证书、密钥及用户配置文件

根据[PKI证书和要求](https://kubernetes.io/zh-cn/docs/setup/best-practices/certificates/)创建符合K8s集群所需要的全部证书，目的是为了熟悉证书生成过程并且了解各个证书的作用。这里先生成根CA，然后再通过根CA创建K8s中需要的几个CA，最后通过这些CA来签发对应的证书。

相关证书说明：

| 类型                            | 路径                   | 父CA | 默认 CN                   | 描述                                                         |      |
| ------------------------------- | ---------------------- | ---- | ------------------------- | ------------------------------------------------------------ | ---- |
|                                 | ca.crt,key             | -    | kubernetes-ca             | Kubernetes 通用 CA                                           |      |
| apiserver端点证书               |                        |      |                           |                                                              |      |
| apiserver客户端(Kubelet)        |                        |      |                           | 用于和 Kubelet 的会话                                        |      |
| apiserver客户端(etcd)           |                        |      |                           | 用于和 etcd 的会话                                           |      |
| 集群管理员的客户端证书（admin） |                        |      |                           | 用于 API 服务器身份认证                                      |      |
| 控制器管理器的客户端证书        |                        |      |                           | 用于和 API 服务器的会话                                      |      |
| 调度器的客户端证书              |                        |      |                           | 用于和 API 服务器的会话                                      |      |
|                                 | etcd/ca.crt,key        | -    | etcd-ca                   | 与 etcd 相关的所有功能                                       |      |
|                                 |                        |      |                           |                                                              |      |
|                                 | front-proxy-ca.crt,key | -    | kubernetes-front-proxy-ca | 用于 [前端代理](https://kubernetes.io/zh-cn/docs/tasks/extend-kubernetes/configure-aggregation-layer/)。只有当运行 kube-proxy 并要支持 [扩展 API 服务器](https://kubernetes.io/zh-cn/docs/tasks/extend-kubernetes/setup-extension-api-server/) 时，才需要 `front-proxy` 证书 |      |
|                                 |                        |      |                           |                                                              |      |
| Kubelet 客户端证书              |                        |      |                           | 用于 API 服务器身份验证                                      |      |
| Kubelet 服务端证书              |                        |      |                           | 用于 API 服务器与 Kubelet 的会话                             |      |

K8s部署完成后所涉及的文件结构如下：

```
/etc/kubernetes# tree
.
├── admin.conf
├── controller-manager.conf
├── kubelet.conf
├── manifests
│   ├── etcd.yaml
│   ├── kube-apiserver.yaml
│   ├── kube-controller-manager.yaml
│   └── kube-scheduler.yaml
├── pki
│   ├── apiserver.crt
│   ├── apiserver-etcd-client.crt
│   ├── apiserver-etcd-client.key
│   ├── apiserver.key
│   ├── apiserver-kubelet-client.crt
│   ├── apiserver-kubelet-client.key
│   ├── ca.crt
│   ├── ca.key
│   ├── etcd
│   │   ├── ca.crt
│   │   ├── ca.key
│   │   ├── healthcheck-client.crt
│   │   ├── healthcheck-client.key
│   │   ├── peer.crt
│   │   ├── peer.key
│   │   ├── server.crt
│   │   └── server.key
│   ├── front-proxy-ca.crt
│   ├── front-proxy-ca.key
│   ├── front-proxy-client.crt
│   ├── front-proxy-client.key
│   ├── kubelet.crt
│   ├── kubelet.key
│   ├── sa.key
│   └── sa.pub
└── scheduler.conf
```

> 先生成用户配置文件的原因是可以及时进行测试，以便及是发现错误并解决后再进行下一步，不用全部操作完了才进行验证。

## 必需证书和密钥对

一般情况下，我们会借助 kubeadm 来部署集群，并且默认情况下 kubeadm 会自动生成所有证书，但是有时候为了更高的安全性和后面使用方便，我们也可以只生成一些**核心**的证书，其余的让 kubeadm 自动生成。

所需的核心证书：

| 路径                   | 默认 CN                   | 描述                                                         |
| ---------------------- | ------------------------- | ------------------------------------------------------------ |
| ca.crt,key             | kubernetes-ca             | Kubernetes 通用 CA                                           |
| etcd/ca.crt,key        | etcd-ca                   | 与 etcd 相关的所有功能                                       |
| front-proxy-ca.crt,key | kubernetes-front-proxy-ca | 用于 [前端代理](https://kubernetes.io/zh-cn/docs/tasks/extend-kubernetes/configure-aggregation-layer/) |

这些核心证书及密钥路径如下：

```
/etc/kubernetes/pki/
├── ca.crt
├── ca.key
├── etcd
│   ├── ca.crt
│   └── ca.key
├── front-proxy-ca.crt
├── front-proxy-ca.key
├── sa.key
└── sa.pub
```

### 生成全局根CA

参见[cfssl工具帮助文档#cfssl](/note/security/cfssl/#cfssl)或者参考[K8s官方文档 - 手动生成证书](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/certificates/#cfssl)生成全局根CA。这里假设生成的全局根CA`global_root_ca.crt`（证书）和`global_root_ca.key`（私钥），并假设其位于`$CERT/`目录下。

```sh
$ CERT=/mnt/share/archive/cert # 全局根CA（global_root_ca.crt）所在路径
```

> 这里的**全局根CA**就是[K8s 文档中的**单根CA**](https://kubernetes.io/zh-cn/docs/setup/best-practices/certificates/#single-root-ca)，该 CA 不是必须的，直接生成 K8s 必须的三个 CA 也是可以的，但是如果有它的话，会更安全（*略*）和更方便（客户端验证服务端证书合法性的时侯都可以直接使用该全局根 CA，如 *kubeconfig 配置文件* 和 *etcd 客户端*等，并且由于全局根 CA 有效期较长，所以一般不会更换它）。

### 生成必须CA

从安全角度来讲，K8s需要`kubernetes-ca`,`etcd-ca`和`kubernetes-front-proxy-ca`三个 CA，当然如果为了简单也可以只创建一个中间CA或者直接使用前面的*全局根CA*（凡是用到这三个CA的都替换为根CA）也是可以的。

为了尽可能和文档一致，假设当前处于`/etc/kubernetes/`目录下：

```sh
$ mkdir -p pki && cd pki/
```

使用 cfssl 生成步骤：

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee ca-csr.json
   {
       "CN": "kubernetes-ca",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{`# 需要根据实际情况进行替换`
       	"C":  "CN",
           "ST": "YunNan",
           "L":  "KunMing",
           "O":  "system",
           "OU": "Kubernetes CA"
       }]
   }
   EOF
   
   $ mkdir -p etcd
   $ cat <<EOF | tee etcd/ca-csr.json
   {
       "CN": "etcd-ca",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{`# 需要根据实际情况进行替换`
           "C":  "CN",
           "ST": "YunNan",
           "L":  "KunMing",
           "O":  "system",
           "OU": "Etcd CA"
       }]
   }
   EOF
   
   $ cat <<EOF | tee front-proxy-csr.json
   {
       "CN": "kubernetes-front-proxy-ca",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{`# 需要根据实际情况进行替换`
           "C":  "CN",
           "ST": "YunNan",
           "L":  "KunMing",
           "O":  "system",
           "OU": "Front Proxy CA"
       }]
   }
   EOF
   ```

2. 根据`CSRJSON`生成私钥和证书请求文件`.csr`。

   ```shell
   $ cfssl genkey ca-csr.json | cfssljson -bare ca
   $ cfssl genkey etcd/ca-csr.json | cfssljson -bare etcd/ca
   $ cfssl genkey front-proxy-csr.json | cfssljson -bare front-proxy-ca
   ```

   > 证书请求`.csr`文件中同时包含了公钥以及`CSRJSON`文件中的信息。

3. 使用根CA对中间CA的公钥（证书请求）进行签名，签名之后生成中间CA。

   ```shell
   $ cfssl sign \
     -ca $CERT/global_root_ca.crt `# 该选项为根CA的证书路径`\
     -ca-key $CERT/global_root_ca.key `# 该选项为根CA的私钥路径`\
     -config $CERT/cfssl-config.json `# 最开始创建的cfssl工具配置文件`\
     -profile ca `# 指定要用配置文件中的哪个 profile`\
     ca.csr `# 该文件为上一步生成的证书请求文件`\
     | cfssljson -bare ca
   
   $ cfssl sign \
     -ca $CERT/global_root_ca.crt \
     -ca-key $CERT/global_root_ca.key \
     -config $CERT/cfssl-config.json \
     -profile ca \
     etcd/ca.csr \
     | cfssljson -bare etcd/ca
   
   $ cfssl sign \
     -ca $CERT/global_root_ca.crt \
     -ca-key $CERT/global_root_ca.key \
     -config $CERT/cfssl-config.json \
     -profile ca \
     front-proxy-ca.csr \
     | cfssljson -bare front-proxy-ca
   ```

   > 由于这些是中间CA，所以后续通过这些中间CA签发的证书都最好和对应的中间CA捆绑在一起使用，以便系统只需要信任一个*全局根CA*就能对这些证书进行链式验证（这应该就是**证书链**一词的由来）。

4. 重命名证书和私钥

   为了和k8s文档中的名称统一，这里将证书和私钥重命名与k8s文档一致（不需要进行格式转换，直接重命名即可）：

   ```shell
   $ mv ca.pem ca.crt && mv ca-key.pem ca.key
   $ mv etcd/ca.pem etcd/ca.crt && mv etcd/ca-key.pem etcd/ca.key
   $ mv front-proxy-ca.pem front-proxy-ca.crt && mv front-proxy-ca-key.pem front-proxy-ca.key
   ```

5. 删除不再需要的中间文件

      ```shell
      $ rm -rf *.{json,csr,pem} && rm -rf etcd/*.{json,csr,pem}
      ```

> 至此，控制平面*CA证书*已经生成完，但是默认情况下，apiserver 主动请求节点的 kubelet 时（如`kubectl logs ...`），需要对 kubelet 提供的证书进行验证，所以如果后面 kubelet 服务端证书也使用*全局根CA*或着*全局根CA*签发的中间CA签发的，那么部署*apiserver*时可以指定使用*全局根CA*来验证*kubelet*提供的证书（这是体现了使用*全局根CA*的好处），否则的话这里还需要生成一对*CA*，以便在部署*kubelet*时使用。

### 创建密钥对

除了上面的 CA 之外，还需要获取用于服务账户管理的密钥对，也就是 `sa.key` 和 `sa.pub`，其用法如下：

| 私钥路径 | 公钥路径 | 命令                    | 参数                               |
| -------- | -------- | ----------------------- | ---------------------------------- |
| sa.key   |          | kube-controller-manager | --service-account-private-key-file |
|          | sa.pub   | kube-apiserver          | --service-account-key-file         |

生成方式为：

```sh
$ openssl genrsa -out sa.key 2048 # 生成私钥
$ openssl rsa -in sa.key -out sa.pub -pubout # 从私钥中提取公钥
```

## 其他证书

如果使用*kubeadm*管理集群，创建上面几对CA和密钥即可，其他证书可以委托给`kubeadm`创建（kubeadm会自动创建缺少的部分），但如果采用二进制方式安装的话就需要全部手动创建完成。

所需证书如下：

| 默认 CN                       | 父级 CA                   | O (位于 Subject 中) | 证书类型       | 主机 (SAN)                                                   |
| ----------------------------- | ------------------------- | ------------------- | -------------- | ------------------------------------------------------------ |
| kube-etcd                     | etcd-ca                   |                     | server, client | `<hostname>`, `<Host_IP>`, `localhost`, `127.0.0.1`,`0:0:0:0:0:0:0:1` |
| kube-etcd-peer                | etcd-ca                   |                     | server, client | `<hostname>`, `<Host_IP>`, `localhost`, `127.0.0.1`,`0:0:0:0:0:0:0:1` |
| kube-etcd-healthcheck-client  | etcd-ca                   |                     | client         |                                                              |
| kube-apiserver-etcd-client    | etcd-ca                   | system:masters      | client         |                                                              |
| kube-apiserver                | kubernetes-ca             |                     | server         | `<hostname>`, `<Host_IP>`, `<advertise_IP>`, `kubernetes`、`kubernetes.default`、`kubernetes.default.svc`、 `kubernetes.default.svc.cluster`、`kubernetes.default.svc.cluster.local` |
| kube-apiserver-kubelet-client | kubernetes-ca             | system:masters      | client         |                                                              |
| front-proxy-client            | kubernetes-front-proxy-ca |                     | client         |                                                              |

> 证书类型如下：
>
> | kind   | 密钥用途                       |
> | ------ | ------------------------------ |
> | server | 数字签名、密钥加密、服务端认证 |
> | client | 数字签名、密钥加密、客户端认证 |

### ETCD相关证书

虽然这里生成的证书是用于K8s，但其他情况下部署ETCD并且需要使用证书时，用到的也是这些证书，只不过`CN`名称可能会使用其他的。

> 实际上`CN`名在任何情况下都可以随意定义，因为作为服务端证书时，`CN`一般表示该证书的名称，但是如果作为客户端证书时，`CN`除了用作证书名称外，还可能代表一个用户的用户名（前提是*ETCD服务端*启用认证），这时候只需要在*ETCD服务端*创建出对应的用户并配置好相关权限即可。

#### 服务端证书

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee etcd/server-csr.json
   {
       "CN": "kube-etcd",
       "hosts": [
           "$NODE_NAME",
           "$NODE_HOST",`# 主机名不够的情况可以添加`
           "localhost",
           "127.0.0.1",
           "0:0:0:0:0:0:0:1"
       ],
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{
           "C": "CN",
           "ST": "YunNan",
           "L": "KunMing"
       }]
   }
   EOF
   
   $ cat <<EOF | tee etcd/peer-csr.json
   {
       "CN": "kube-etcd-peer",
       "hosts": [
           "$NODE_NAME",
           "$NODE_HOST",`# 主机名不够的情况可以添加`
           "localhost",
           "127.0.0.1",
           "0:0:0:0:0:0:0:1"
       ],
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{
           "C": "CN",
           "ST": "YunNan",
           "L": "KunMing"
       }]
   }
   EOF
   ```

2. 签发证书

   ```shell
   $ cfssl gencert \
     -ca etcd/ca.crt `# ETCD中间CA`\
     -ca-key etcd/ca.key `# ETCD中间CA对应的私钥`\
     -config $CERT/cfssl-config.json  `# cfssl配置文件`\
     -profile kubernetes `# 该Profile在配置文件中表示 server + client`\
     etcd/server-csr.json \
     | cfssljson -bare etcd/server
   
   $ cfssl gencert \
     -ca etcd/ca.crt \
     -ca-key etcd/ca.key \
     -config $CERT/cfssl-config.json \
     -profile kubernetes \
     etcd/peer-csr.json \
     | cfssljson -bare etcd/peer
   ```
   
3. 捆绑证书并重命名私钥

   ```shell
   $ cat etcd/server.pem etcd/ca.crt > etcd/server.crt && mv etcd/server-key.pem etcd/server.key
   $ cat etcd/peer.pem etcd/ca.crt > etcd/peer.crt && mv etcd/peer-key.pem etcd/peer.key
   ```
   
4. 删除不再需要的中间文件

   ```sh
   $ rm -rf etcd/*.{pem,csr,json}
   ```

#### 客户端证书

这里创建*apiserver*连接*etcd*所使用的客户端证书以及健康检查使用的客户端证书，其他需要连接*etcd*的客户端也可以使用相同的方式生成客户端证书。

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee etcd/healthcheck-client-csr.json
   {
       "CN": "kube-etcd-healthcheck-client",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{
           "C": "CN",
           "ST": "YunNan",
           "L": "KunMing",
           "O": "system:masters"
       }]
   }
   EOF
   
   $ cat <<EOF | tee apiserver-etcd-client-csr.json
   {
       "CN": "kube-apiserver-etcd-client",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{
           "C": "CN",
           "ST": "YunNan",
           "L": "KunMing"
       }]
   }
   EOF
   ```

2. 签发证书

   ```shell
   $ cfssl gencert \
     -ca etcd/ca.crt \
     -ca-key etcd/ca.key \
     -config $CERT/cfssl-config.json \
     -profile client \
     etcd/healthcheck-client-csr.json \
     | cfssljson -bare etcd/healthcheck-client
   
   $ cfssl gencert \
     -ca etcd/ca.crt \
     -ca-key etcd/ca.key \
     -config $CERT/cfssl-config.json \
     -profile client \
     apiserver-etcd-client-csr.json \
     | cfssljson -bare apiserver-etcd-client
   ```

3. 捆绑证书并重命名私钥

   ```shell
   $ cat etcd/healthcheck-client.pem etcd/ca.crt > etcd/healthcheck-client.crt && mv etcd/healthcheck-client-key.pem etcd/healthcheck-client.key
   $ cat apiserver-etcd-client.pem etcd/ca.crt > apiserver-etcd-client.crt && mv apiserver-etcd-client-key.pem apiserver-etcd-client.key
   ```

4. 删除不再需要的中间文件

   ```sh
   $ rm -rf etcd/*.{pem,csr,json}
   ```

### apiserver相关证书

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee apiserver-csr.json
   {
       "CN": "kube-apiserver",
       "hosts": [
           "$NODE_NAME",
           "$NODE_HOST",`# 主机名（域名）和IP不够的情况可以添加`
           "$APISERVER_SERVICE",`# apiserver 服务IP，一般为网段的第一个`
           "kubernetes",
           "kubernetes.default",
           "kubernetes.default.svc",
           "kubernetes.default.svc.cluster.local",
           "localhost",
           "127.0.0.1",
           "0:0:0:0:0:0:0:1"
       ],
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{
           "C": "CN",
           "ST": "YunNan",
           "L": "KunMing",
           "O": "system:masters"
       }]
   }
   EOF
   
   $ cat <<EOF | tee apiserver-kubelet-client-csr.json
   {
       "CN": "kube-apiserver-kubelet-client",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{
           "C": "CN",
           "ST": "YunNan",
           "L": "KunMing"
       }]
   }
   EOF
   ```

2. 签发证书

   ```shell
   $ cfssl gencert \
     -ca ca.crt `# kubernetes-ca`\
     -ca-key ca.key `# kubernetes-ca对应的私钥`\
     -config $CERT/cfssl-config.json  `# cfssl配置文件`\
     -profile server \
     apiserver-csr.json \
     | cfssljson -bare apiserver
   
   $ cfssl gencert \
     -ca ca.crt \
     -ca-key ca.key \
     -config $CERT/cfssl-config.json \
     -profile client \
     apiserver-kubelet-client-csr.json \
     | cfssljson -bare apiserver-kubelet-client
   ```

3. 捆绑证书并重命名私钥

   ```shell
   $ cat apiserver.pem ca.crt > apiserver.crt && mv apiserver-key.pem apiserver.key
   $ cat apiserver-kubelet-client.pem ca.crt > apiserver-kubelet-client.crt && mv apiserver-kubelet-client-key.pem apiserver-kubelet-client.key
   ```

4. 删除不再需要的中间文件

   ```sh
   $ rm -rf *.{pem,csr,json}
   ```

### front-proxy-client证书

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee front-proxy-client-csr.json
   {
       "CN": "front-proxy-client",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [{
           "C": "CN",
           "ST": "YunNan",
           "L": "KunMing"
       }]
   }
   EOF
   ```

2. 签发证书

   ```shell
   $ cfssl gencert \
     -ca front-proxy-ca.crt `# kubernetes-front-proxy-ca`\
     -ca-key front-proxy-ca.key `# kubernetes-front-proxy-ca对应的私钥`\
     -config $CERT/cfssl-config.json  `# cfssl配置文件`\
     -profile client \
     front-proxy-client-csr.json \
     | cfssljson -bare front-proxy-client
   ```

3. 捆绑证书并重命名私钥

   ```shell
   $ cat front-proxy-client.pem front-proxy-ca.crt > front-proxy-client.crt && mv front-proxy-client-key.pem front-proxy-client.key
   ```

4. 删除不再需要的中间文件

   ```sh
   $ rm -rf *.{pem,csr,json}
   ```

## 生成用户（客户端）配置文件

可参考[官方文档 - 为用户帐户配置证书](https://kubernetes.io/zh-cn/docs/setup/best-practices/certificates/#configure-certificates-for-user-accounts)。

### 生成各个用户的证书

接上面的操作，假设当前位于`/etc/kubernetes/pki/`目录下，现在将各个用户的证书临时生成在`./client/`目录下：

```sh
$ mkdir -p ./client/ && cd ./client/
```

生存各个用户的证书：

```shell
$ cat <<EOF | tee admin-csr.json`# 凭据名称: default-admin`
{
    "CN": "kubernetes-admin",
    "key": { "algo": "rsa", "size": 2048 },
    "names": [{
        "C": "CN",
        "ST": "YunNan",
        "L": "KunMing",
        "O": "system:masters"
    }]
}
EOF

# 该证书以及对应的配置文件为可选，如果提供则 CN 的格式必须符合要求；如果缺省，可以先通过 --bootstrap-kubeconfig 初始化，后由 apiserver 给加入的节点生成证书
$ cat <<EOF | tee kubelet-csr.json`# 凭据名称: default-auth`
{
    "CN": "system:node:$NODE_NAME",`# 格式为"system:node:<nodeName>",<nodeName> 的值 必须 与 kubelet 向 apiserver 注册时提供的节点名称的值完全匹配，这样创建出来的证书启动kubelet时会自动将kubelet所在节点加入集群，否则加入成功`
    "key": { "algo": "rsa", "size": 2048 },
    "names": [{
        "C": "CN",
        "ST": "YunNan",
        "L": "KunMing",
        "O": "system:nodes"
    }]
}
EOF

$ cat <<EOF | tee kube-proxy-csr.json
{
    "CN": "system:kube-proxy",
    "key": { "algo": "rsa", "size": 2048 },
    "names": [{
        "C": "CN",
        "ST": "YunNan",
        "L": "KunMing",
        "O": "system:nodes"
    }]
}
EOF

$ cat <<EOF | tee controller-manager-csr.json`# 凭据名称: default-controller-manager`
{
    "CN": "system:kube-controller-manager",
    "key": { "algo": "rsa", "size": 2048 },
    "names": [{
        "C": "CN",
        "ST": "YunNan",
        "L": "KunMing"
    }]
}
EOF

$ cat <<EOF | tee scheduler-csr.json`# 凭据名称: default-scheduler`
{
    "CN": "system:kube-scheduler",
    "key": { "algo": "rsa", "size": 2048 },
    "names": [{
        "C": "CN",
        "ST": "YunNan",
        "L": "KunMing"
    }]
}
EOF

$ for item in 'admin' 'kubelet' 'kube-proxy' 'controller-manager' 'scheduler'
do
  cfssl gencert \
      -ca ../ca.crt `# kubernetes-ca`\
      -ca-key ../ca.key `# kubernetes-ca对应的私钥`\
      -config $CERT/cfssl-config.json  `# cfssl配置文件`\
      -profile client \
      "$item"-csr.json \
      | cfssljson -bare $item
done

# 将密钥重命名为推荐的名称并删除不再需要的中间文件
$ for item in 'admin' 'kubelet' 'kube-proxy' 'controller-manager' 'scheduler'
do
  mv $item.pem     $item.crt
  mv $item-key.pem $item.key
done
$ rm -rf *.{pem,csr,json}
```

### 生成配置文件

```shell
# 接上面的操作，假设当前位于目录: /etc/kubernetes/pki/client/
$ cd ../../
$ for item in 'admin' 'kubelet' 'kube-proxy' 'controller-manager' 'scheduler'
do
  touch $item.conf
  KUBECONFIG=$item.conf kubectl config set-cluster $NODE_NAME --server=https://$NODE_HOST:6443 --certificate-authority ./pki/ca.crt --embed-certs
  KUBECONFIG=$item.conf kubectl config set-credentials $NODE_NAME --client-key ./pki/client/"$item".key --client-certificate ./pki/client/"$item".crt --embed-certs
  KUBECONFIG=$item.conf kubectl config set-context $NODE_NAME --cluster $NODE_NAME --user $NODE_NAME
  KUBECONFIG=$item.conf kubectl config use-context $NODE_NAME
done
```

# 部署K8s各组件

## etcd

由于k8s使用etcd存储数据，所以需要先部署etcd。etcd如果开启认证，则需要在etcd中创建`apiserver`使用的证书`CN`对应的用户并授权（一般将其添加到`root`组）。部署成功后，需要确保后面用于启动`apiserver`的证书密钥能正常使用etcd（可以使用`etcdctl`验证，因为`apiserver`也是etcd的一个客户端）。

etcd不一定要部署在Master上，只要能访问就行，具体部署方式可以参考[etcd笔记](/note/db/etcd)。

如果采用静态Pod部署，则可以参考以下**kubeadm**生成的部署清单：

```sh
$ cat <<EOF | tee /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://$NODE_HOST:2379
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://$NODE_HOST:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt`# 服务端证书`
    - --client-cert-auth=true`# 启用客户端证书认证。客户端必须提供证书`
    - --data-dir=/var/lib/etcd
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://$NODE_HOST:2380
    - --initial-cluster=node-pc-ubuntu=https://$NODE_HOST:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://$NODE_HOST:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://$NODE_HOST:2380
    - --name=node-pc-ubuntu
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.k8s.io/etcd:3.5.4-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health?exclude=NOSPACE&serializable=true
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health?serializable=false
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
EOF
```

## 下载二进制文件

清单如下表，根据下表可知，一般下载**Node**即可，不过这里采用二进制方式部署，所以需要下载**Server**。

| Server                                                       | Node                                                         | Client                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------- |
| apiextensions-apiserver<br/>kubeadm<br/>kube-aggregator<br/>kube-apiserver<br/>kube-apiserver.docker_tag<br/>kube-apiserver.tar<br/>kube-controller-manager<br/>kube-controller-manager.docker_tag<br/>kube-controller-manager.tar<br/>kubectl<br/>kubectl-convert<br/>kubelet<br/>kube-log-runner<br/>kube-proxy<br/>kube-proxy.docker_tag<br/>kube-proxy.tar<br/>kube-scheduler<br/>kube-scheduler.docker_tag<br/>kube-scheduler.tar<br/>mounter | kubeadm<br/>kubectl<br/>kubectl-convert<br/>kubelet<br/>kube-log-runner<br/>kube-prox | kubectl<br />kubectl-convert |

下载后将可执行文件复制或链接到`/usr/local/bin/`。

## Kubernetes control plane

一般来说，控制平面所在的那个节点叫做`Master`，该节点可以不用`kubelet`，甚至可以没有容器环境。

### 部署kube-apiserver

直接启动：

```shell
$ cat <<EOF | tee start-kube-apiserver.sh
# 以下命令其实可以通过kubeadm部署后在 /etc/kubernetes/manifests/kube-apiserver.yaml 中得到
kube-apiserver \\
--advertise-address=$NODE_HOST `# 集群通告地址，即告诉其他客户端自己的地址，该地址一般为宿主机地址`\\
--allow-privileged=true `# 是否启用授权`\\
--authorization-mode=RBAC,Node `# 认证授权，启用RBAC授权和节点自管理`\\
--client-ca-file=/etc/kubernetes/pki/ca.crt \\
--enable-admission-plugins=NodeRestriction `# 准入控制模块`\\
--enable-bootstrap-token-auth=true `# 是否启用TLS bootstrap机制`\\
--etcd-cafile=$ETCD_CAFILE `# etcd集群CA证书`\\
--etcd-certfile=$ETCD_CERTFILE `# etcd集群认证证书`\\
--etcd-keyfile=$ETCD_KEYFILE `# etcd集群认证证书私钥`\\
--etcd-prefix=$ETCD_PREFIX `# 要在 etcd 中所有资源路径之前添加的前缀。默认值："/registry"。一般情况不用添加，但是由于用的是共享etcd，所有为了避免冲突，加上一个前缀`\\
--etcd-servers=$ETCD_SERVERS `# etcd集群地址`\\
--kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt `# apiserver访问kubelet客户端证书`\\
--kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key `# apiserver访问kubelet客户端证书私钥`\\
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname \\
--proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt `# 启动聚合层相关配置`\\
--proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key `# 启动聚合层相关配置`\\
--requestheader-allowed-names=front-proxy-client `# 启动聚合层相关配置`\\
--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt `# 启动聚合层相关配置`\\
--requestheader-extra-headers-prefix=X-Remote-Extra- `# 启动聚合层相关配置`\\
--requestheader-group-headers=X-Remote-Group `# 启动聚合层相关配置`\\
--requestheader-username-headers=X-Remote-User `# 启动聚合层相关配置`\\
--secure-port=6443 `# https安全端口`\\
--service-account-issuer=https://kubernetes.default.svc.cluster.local `# 1.20+版本必须加的参数`\\
--service-account-key-file=/etc/kubernetes/pki/sa.pub `# 包含 PEM 编码的 x509 RSA 或 ECDSA 私钥或公钥的文件，用于验证 ServiceAccount 令牌。 指定的文件可以包含多个key，不同的文件可以多次指定flag。 如果未指定，则使用 --tls-private-key-file。 提供 --service-account-signing-key-file 时必须指定`\\
--service-account-signing-key-file=/etc/kubernetes/pki/sa.key `# 包含服务帐户令牌颁发者的当前私钥的文件的路径。 发行者将使用此私钥签署已发行的 ID 令牌`\\
--service-cluster-ip-range=$SERVICE_CLUSTER_IP_RANGE `# Service虚拟IP地址段`\\
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt  `# apiserver https证书`\\
--tls-private-key-file=/etc/kubernetes/pki/apiserver.key `# apiserver https证书私钥`\\
`# 以下选项为kubeadm创建的默认配置也没有的` \\
--bind-address=0.0.0.0 `# 监听地址,这里监听全部地址，如果需要更高安全性可以只监听maste节点ip地址`\\
`# --token-auth-file=/opt/kubernetes/cfg/token.csv` `# 如果设置，该文件将用于通过令牌方式验证客户端。bootstrap token文件`\\
--audit-log-maxage=30 `# 审计日志相关`\\
--audit-log-maxbackup=3 `# 审计日志相关`\\
--audit-log-maxsize=100 `# 审计日志相关`\\
--audit-log-path=/opt/kubernetes/logs/k8s-audit.log `# 审计日志相关`
EOF
```

静态Pod部署：

```sh
$ cat <<EOF | tee /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: $NODE_HOST:6443
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=$NODE_HOST
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-prefix=/k8s/registry-ubuntu
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.1.0.0/16
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: registry.k8s.io/kube-apiserver:v1.25.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: $NODE_HOST
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: $NODE_HOST
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: $NODE_HOST
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
EOF
```

验证`apiserver`是否正常：

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                        ERROR
controller-manager   Unhealthy   Get "https://127.0.0.1:10257/healthz": dial tcp 127.0.0.1:10257: connect: connection refused   
scheduler            Unhealthy   Get "https://127.0.0.1:10259/healthz": dial tcp 127.0.0.1:10259: connect: connection refused   
etcd-0               Healthy     {"health":"true","reason":""}
```

> 由于前面部署了etcd，所以如果这里成功相应并显示etcd为`Healthy`状态则说明`kube-apiserver`部署成功。
>
> 如果没有成功，并且是通过静态Pod方式部署的，那建议先直接通过命令启动，等问题排除之后再转为Pod部署。

### 部署kube-controller-manager

直接启动：

```shell
$ cat <<EOF | tee start-kube-controller-manager.sh
# 以下命令其实可以通过kubeadm部署后在 /etc/kubernetes/manifests/kube-controller-manager.yaml 中得到
kube-controller-manager \\
--allocate-node-cidrs=true `# 是否应在云提供商上分配和设置Pod的CIDR`\\
--authentication-kubeconfig=/etc/kubernetes/controller-manager.conf \\
--authorization-kubeconfig=/etc/kubernetes/controller-manager.conf \\
--bind-address=0.0.0.0 `# 监听--secure-port端口的IP地址(default 0.0.0.0)`\\
--client-ca-file=/etc/kubernetes/pki/ca.crt \\
--cluster-cidr=$CLUSTER_CIDR `# 集群中Pod的CIDR范围，要求--allocate-node-cidrs为true`\\
--cluster-name=kubernetes \\
--cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt `# 用于颁发集群范围的证书。 如果指定，则不能指定更具体的 --cluster-signing-* 标志`\\
--cluster-signing-key-file=/etc/kubernetes/pki/ca.key  `# --cluster-signing-cert-file CA证书对应密钥`\\
--controllers=*,bootstrapsigner,tokencleaner \\
--kubeconfig=/etc/kubernetes/controller-manager.conf `# 连接apiserver配置文件`\\
--leader-elect=true `# 当该组件启动多个时，自动选举（HA）`\\
--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt \\
--root-ca-file=/etc/kubernetes/pki/ca.crt `# 根CA，可以制定签发 --cluster-signing-cert-file 的CA，也可以与之相同`\\
--service-account-private-key-file=/etc/kubernetes/pki/ca.key `# 用于签署服务帐户令牌`\\
--service-cluster-ip-range=$SERVICE_CLUSTER_IP_RANGE `# 集群service的cidr范围，需要--allocate-node-cidrs设置为true`\\
--use-service-account-credentials=true `# 为每个控制器使用单独的服务帐户凭据`\\
`# 以下选项为kubeadm创建的默认配置也没有的` \\
--cluster-signing-duration=87600h0m0s
EOF
```

静态Pod部署：

```sh
$ cat <<EOF | tee /etc/kubernetes/manifests/kube-controller-manager.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.2.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.1.0.0/16
    - --use-service-account-credentials=true
    image: registry.k8s.io/kube-controller-manager:v1.25.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10257
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-controller-manager
    resources:
      requests:
        cpu: 200m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10257
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      name: flexvolume-dir
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /etc/kubernetes/controller-manager.conf
      name: kubeconfig
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      type: DirectoryOrCreate
    name: flexvolume-dir
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /etc/kubernetes/controller-manager.conf
      type: FileOrCreate
    name: kubeconfig
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
EOF
```

验证组件是否正常

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                        ERROR
scheduler            Unhealthy   Get "https://127.0.0.1:10259/healthz": dial tcp 127.0.0.1:10259: connect: connection refused   
controller-manager   Healthy     ok                                                                                             
etcd-0               Healthy     {"health":"true","reason":""}
```

### 部署kube-scheduler

直接启动：

```shell
$ cat <<EOF | tee start-kube-scheduler.sh
# 以下命令其实可以通过kubeadm部署后在 /etc/kubernetes/manifests/kube-scheduler.yaml 中得到
kube-scheduler \\
--authentication-kubeconfig=/etc/kubernetes/scheduler.conf \\
--authorization-kubeconfig=/etc/kubernetes/scheduler.conf \\
--bind-address=0.0.0.0 `# 侦听 --secure-port 端口的 IP 地址。 集群的其余部分和 CLI/Web 客户端必须可以访问关联的接口。 如果为空白或未指定地址（0.0.0.0 或 ::），将使用所有接口。 （默认 0.0.0.0）`\\
--kubeconfig=/etc/kubernetes/scheduler.conf \\
--leader-elect=true `# 当该组件启动多个时，自动选举（HA）`\\
EOF
```

静态Pod部署：

```sh
$ cat <<EOF | tee /etc/kubernetes/manifests/kube-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    image: registry.k8s.io/kube-scheduler:v1.25.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
EOF
```

验证组件是否正常:

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
$ kubectl --kubeconfig /etc/kubernetes/admin.conf create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```

### 部署kubelet

kubelet的部署方式有两种，一种是直接生成kubelet使用的证书，并根据该证书创建对应的 kubeconfig 文件；另一种是使用 Bootstrap Kubeconfig 配置加入集群，然后由集群自动签发证书。由于第二种方式用于认证的信息为一个token，相比第一种直接使用证书的情况安全性有所降低，所以第二种情况还需要明确在集群中批准才能真正加入成功。

#### 生成kubelet初次加入集群引导kubeconfig文件

只有`--kubeconfig`不可用且使用`--bootstrap-kubeconfig`创建时需要。

`--bootstrap-kubeconfig`和`--kubeconfig`的区别：由于`--kubeconfig`配置文件中的密钥并不是所有节点通用的，因为证书的CN中包含了对应的主机名，这使得需要单独为每一个Worker节点创建对应的证书。而`--bootstrap-kubeconfig`则提供了一个临时连接`apiserver`的配置（里面使用了一个临时`token`），用于请求`apiserver`为当前节点颁发相应的证书，并将证书存储到`--kubeconfig`配置文件中。

```shell
$ KUBE_CONFIG="/etc/kubernetes/bootstrap-kubelet.conf"

$ kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --embed-certs=true \
  --server=https://$NODE_HOST:6443 \
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
$ sudo mkdir /var/lib/kubelet && cat << EOF | sudo tee /var/lib/kubelet/config.yaml
# 以下命令其实可以通过kubeadm部署后在 /var/lib/kubelet/config.yaml 中得到
# see https://kubernetes.io/zh-cn/docs/reference/config-api/kubelet-config.v1beta1/
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s # default: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s # default: 0s
    cacheUnauthorizedTTL: 30s # default: 0s
cgroupDriver: systemd # 操控宿主系统上控制组（CGroup） 的驱动程序（cgroupfs 或 systemd）。默认值："cgroupfs"
clusterDNS: # 集群 DNS 服务器的 IP 地址的列表。 如果设置了，kubelet 将会配置所有容器使用这里的 IP 地址而不是宿主系统上的 DNS 服务器来完成 DNS 解析。
  - $CLUSTER_DNS
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
failSwapOn: false # 节点上启用交换分区时是否拒绝启动
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
resolvConf: /run/systemd/resolve/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m # 设置除长期运行的请求（pull、 logs、exec和attach）之外所有运行时请求的超时时长。kubeadmDefault: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
# 其他
#address: 0.0.0.0 # 允许访问的地址
#port: 10250 # 监听的端口
#readOnlyPort: 10255
#serializeImagePulls: false # 以并行方式拉取镜像
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi # 当可用内存低于 100Mi 时, kubelet 将会开始驱逐 Pod
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
EOF
```

#### 启动kubelet

##### 直接启动

```sh
$ cat <<EOF | tee start-kubelet.sh
KUBELET_KUBECONFIG_ARGS="--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
KUBELET_CONFIG_ARGS="--config=/var/lib/kubelet/config.yaml"
KUBELET_KUBEADM_ARGS="--container-runtime=remote --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --hostname-override=$NODE_NAME --pod-infra-container-image=registry.k8s.io/pause:3.8"
kubelet --cert-dir=/etc/kubernetes/pki \$KUBELET_KUBECONFIG_ARGS  \$KUBELET_CONFIG_ARGS \$KUBELET_KUBEADM_ARGS
EOF
```

##### 使用服务启动

```sh
$ cat <<EOF | tee /var/lib/kubelet/kubeadm-flags.env
KUBELET_KUBEADM_ARGS="--container-runtime=remote --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --hostname-override=$NODE_NAME --pod-infra-container-image=registry.k8s.io/pause:3.8"
EOF

$ cat <<EOF | tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network-online.target
After=network-online.target

[Service]
User=root
# 用于 TLS 引导程序的 KubeConfig 文件为 /etc/kubernetes/bootstrap-kubelet.conf（一般采用 kubeadm init 生成的都没有该文件）， 但仅当 /etc/kubernetes/kubelet.conf 不存在时才能使用，即两个文件存在其中一个即可，如果都存在，则使用后者。
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# 这是 "kubeadm init" 和 "kubeadm join" 运行时生成的文件，
# 动态地填充 KUBELET_KUBEADM_ARGS 变量
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# 这是一个文件，用户在不得已下可以将其用作替代 kubelet args（对于 DEB：/etc/default/kubelet , 对于 RPM: /etc/sysconfig/kubelet），KUBELET_EXTRA_ARGS 应该从此文件中获取。
# 用户最好使用 .NodeRegistration.KubeletExtraArgs 对象在配置文件中替代。
# KUBELET_EXTRA_ARGS 在标志链中排在最后，并且在设置冲突时具有最高优先级。
EnvironmentFile=-/etc/default/kubelet
ExecStart=/usr/local/bin/kubelet $KUBELET_OPTS $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF
```

#### 批准kubelet证书申请并加入集群

只有`--kubeconfig`不可用且使用`--bootstrap-kubeconfig`创建时需要。

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf get csr
NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           REQUESTEDDURATION   CONDITION
node-csr-KLFsUNn9eyQ_JaQjwgZmeokMm7JwwheNyHKSoPIROz0   38s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Pending
$ kubectl certificate approve node-csr-KLFsUNn9eyQ_JaQjwgZmeokMm7JwwheNyHKSoPIROz0
```

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
clusterCIDR: $CLUSTER_CIDR # 集群中 Pod 的 CIDR 范围。配置后，从该范围之外发送到服务集群 IP 的流量被伪装，从 Pod 发送到外部 LoadBalancer IP 的流量将被重定向到相应的集群 IP。 对于双协议栈集群，接受一个逗号分隔的列表， 每个 IP 协议族（IPv4 和 IPv6）至少包含一个 CIDR。
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
+ clusterIP: $CLUSTER_DNS
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
    name: kube-apiserver-kubelet-client # 用户为 apiserver-kubelet-client 证书的 CN
EOF

$ kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f apiserver-to-kubelet-rbac.yaml
clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet created
clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver created
```

### 删除master污点

如果启动kubelet时就已经去掉的话，这里查不到值。

```shell
# 查看污点
$ kubectl --kubeconfig /etc/kubernetes/admin.conf describe nodes <node-name> | grep Taints
Taints:             node.kubernetes.io/not-ready:NoSchedule

# 删除污点（最后的'-'表示减去/删除的意思）
$ kubectl --kubeconfig /etc/kubernetes/admin.conf taint node local node.kubernetes.io/not-ready:NoSchedule-
node/local untainted
```

### 安装网络插件（calico）

[官网](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)

下载部署文件

```shell
$ curl -LO https://docs.projectcalico.org/manifests/calico.yaml
```

取消`CALICO_IPV4POOL_CIDR`注释，并将`alue`改为POD网段

```shell
vim +4434 calico.yaml
...
- name: CALICO_IPV4POOL_CIDR
  value: $CLUSTER_CIDR
...
```

部署

```shell
$ kubectl --kubeconfig /etc/kubernetes/admin.conf apply -f calico.yaml
```

# 参考

[k8s 1.24.x 二进制部署最新版本](https://blog.51cto.com/flyfish225/5396121)

[PKI 证书和要求](https://kubernetes.io/zh-cn/docs/setup/best-practices/certificates/)
