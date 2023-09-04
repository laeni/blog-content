---
title: 生成K8s所需的证书和密钥及用户配置文件
author: 'Laeni'
tags: 安全, Kubernetes, PKI, 证书
date: '2022-08-01'
updated: '2022-08-01'
---

根据[PKI证书和要求](https://kubernetes.io/zh-cn/docs/setup/best-practices/certificates/#configure-certificates-manually)创建符合K8s集群所需要的全部证书，目的是为了熟悉证书生成过程并且了解各个证书的作用。这里先生成根CA，然后再通过根CA创建K8s中需要的几个CA，最后通过这些CA来签发对应的证书。

# 证书文件结构预览

在各种情况下推荐手动生成的证书。

## kubeadm 工具部署集群

```sh
/etc/kubernetes/pki$ tree
├── ca.crt
├── ca.key
├── etcd
│   ├── ca.crt
│   └── ca.key
├── front-proxy-ca.crt
└── front-proxy-ca.key
```

> 使用**kubeadm**时，建议仅创建必须的根CA，其他证书自动生成。

## 二进制部署集群

1. Etcd配套证书

2. Kube-APIserver配套证书

3. kube-scheduler配套证书

4. kube-controller-manager配套证书

5. kube-proxy 配套证书

6. service account配套证书

7. admin配套证书

8. kubelet配套证书

# 生成根CA

参见[cfssl工具帮助文档](/note/security/cfssl)创建根CA，创建根CA很简单，但是注意根CA的有效期一般比其他的要稍微长一点。在K8s中，实际上并不需要本根CA，但是如果存在的话，在某些时候可以简化使用，比如信任根CA后使用`etcdctl`时可以不指定`--cacert`选项。

生成根CA后返回这里继续，生成之后当前目录应该存在证书（`ca.pem`）和私钥（`ca-key.pem`），后面的k8s相关证书将在k8s目录下进行：

```shell
$ mkdir k8s && cd k8s/
```

# 创建中间 CA

从安全角度严格来讲，K8s需要`kubernetes-ca`,`etcd-ca`和`kubernetes-front-proxy-ca`三个中间CA，当然如果为了简单则可以指创建一个中间CA或者直接使用根CA也是可以的。

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee ca-csr.json
   {
     "CN": "kubernetes-ca",
     "key": { "algo": "rsa", "size": 2048 },
     "names": [
       {
         "C":  "CN",
         "ST": "YunNan",
         "L":  "KunMing",
         "O":  "system",
         "OU": "Etcd CA"
       }
     ]
   }
   EOF
   
   $ mkdir -p etcd
   $ cat <<EOF | tee etcd/ca-csr.json
   {
     "CN": "etcd-ca",
     "key": { "algo": "rsa", "size": 2048 },
     "names": [
       {
         "C":  "CN",
         "ST": "YunNan",
         "L":  "KunMing",
         "O":  "system",
         "OU": "Etcd CA"
       }
     ]
   }
   EOF
   
   $ cat <<EOF | tee front-proxy-csr.json
   {
     "CN": "kubernetes-front-proxy-ca",
     "key": { "algo": "rsa", "size": 2048 },
     "names": [
       {
         "C":  "CN",
         "ST": "YunNan",
         "L":  "KunMing",
         "O":  "system",
         "OU": "Front Proxy CA"
       }
     ]
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
     -ca ../ca.pem `# 该选项为根CA的证书路径`\
     -ca-key ../ca-key.pem `# 该选项为根CA的私钥路径`\
     -config ../cfssl-config.json `# 最开始创建的cfssl工具配置文件`\
     -profile ca `# 指定要用配置文件中的哪个 profile`\
     ca.csr `# 该文件为上一步生成的证书请求文件`\
     | cfssljson -bare ca
   
   $ cfssl sign \
     -ca ../ca.pem \
     -ca-key ../ca-key.pem \
     -config ../cfssl-config.json \
     -profile ca \
     etcd/ca.csr \
     | cfssljson -bare etcd/ca
   
   $ cfssl sign \
     -ca ../ca.pem \
     -ca-key ../ca-key.pem \
     -config ../cfssl-config.json \
     -profile ca \
     front-proxy-ca.csr \
     | cfssljson -bare front-proxy-ca
   ```

   > 由于这些是中间CA，所以后续通过这些中间CA签发的证书都最好和对应的中间CA捆绑在一起使用，以便能直接通过根CA进行验证。

4. 重命名证书和私钥

   为了和k8s文档中的名称统一，这里将证书和私钥重命名与k8s文档一致（不需要进行格式转换，直接重命名即可）：

   ```shell
   $ mv ca.pem ca.crt && mv ca-key.pem ca.key
   $ mv etcd/ca.pem etcd/ca.crt && mv etcd/ca-key.pem etcd/ca.key
   $ mv front-proxy.pem front-proxy-ca.crt && mv front-proxy-ca-key.pem front-proxy-ca.key
   ```

   至此，所有中间证书就全部生成完成，可以将不再使用的中间文件删除：

   ```shell
   $ rm -rf *.{json,csr} && rm -rf etcd/*.{json,csr}
   ```

# 创建其他证书

一般情况，创建上面三对中间CA即可，其他证书可以委托给`kubeadm`创建，但也可以全部手动创建好。

## 创建ETCD相关证书

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee etcd/server-csr.json
   {
       "CN": "kube-etcd",
       "hosts": [
           "*.laeni.cn",
           "10.10.1.1",
           "127.0.0.1",
           "localhost"
       ],
       "key": { "algo": "rsa", "size": 2048 },
       "names": [
           {
               "C": "CN",
               "ST": "YunNan",
               "L": "KunMing"
           }
       ]
   }
   EOF
   
   $ cat <<EOF | tee etcd/peer-csr.json
   {
       "CN": "kube-etcd-peer",
       "hosts": [
           "*.laeni.cn",
           "10.10.1.1",
           "127.0.0.1",
           "localhost"
       ],
       "key": { "algo": "rsa", "size": 2048 },
       "names": [
           {
               "C": "CN",
               "ST": "YunNan",
               "L": "KunMing"
           }
       ]
   }
   EOF
   
   $ cat <<EOF | tee etcd/healthcheck-client-csr.json
   {
       "CN": "kube-etcd-healthcheck-client",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [
           {
               "C": "CN",
               "ST": "YunNan",
               "L": "KunMing",
               "O": "system:masters"
           }
       ]
   }
   EOF
   
   $ cat <<EOF | tee apiserver-etcd-client-csr.json
   {
       "CN": "kube-apiserver-etcd-client",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [
           {
               "C": "CN",
               "ST": "YunNan",
               "L": "KunMing"
           }
       ]
   }
   EOF
   ```

2. 签发证书

   ```shell
   $ cfssl gencert \
     -ca etcd/ca.crt `# ETCD中间CA`\
     -ca-key etcd/ca.key `# ETCD中间CA对应的私钥`\
     -config ../cfssl-config.json  `# cfssl配置文件`\
     -profile kubernetes \
     etcd/server-csr.json \
     | cfssljson -bare etcd/server
   
   $ cfssl gencert \
     -ca etcd/ca.crt \
     -ca-key etcd/ca.key \
     -config ../cfssl-config.json \
     -profile kubernetes \
     etcd/peer-csr.json \
     | cfssljson -bare etcd/peer
   
   $ cfssl gencert \
     -ca etcd/ca.crt \
     -ca-key etcd/ca.key \
     -config ../cfssl-config.json \
     -profile client \
     etcd/healthcheck-client-csr.json \
     | cfssljson -bare etcd/healthcheck-client
   
   $ cfssl gencert \
     -ca etcd/ca.crt \
     -ca-key etcd/ca.key \
     -config ../cfssl-config.json \
     -profile client \
     apiserver-etcd-client-csr.json \
     | cfssljson -bare apiserver-etcd-client
   ```

3. 捆绑证书并重命名私钥

   ```shell
   $ cat etcd/server.pem etcd/ca.crt > etcd/server.crt && mv etcd/server-key.pem etcd/server.key
   $ cat etcd/peer.pem etcd/ca.crt > etcd/peer.crt && mv etcd/peer-key.pem etcd/peer.key
   $ cat etcd/healthcheck-client.pem etcd/ca.crt > etcd/healthcheck-client.crt && mv etcd/healthcheck-client-key.pem etcd/healthcheck-client.key
   $ cat apiserver-etcd-client.pem etcd/ca.crt > apiserver-etcd-client.crt && mv apiserver-etcd-client-key.pem apiserver-etcd-client.key
   ```

## 创建apiserver相关证书

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee apiserver-csr.json
   {
       "CN": "kube-apiserver",
       "hosts": [
           "kubernetes",
           "kubernetes.default",
           "kubernetes.default.svc",
           "kubernetes.default.svc.cluster.local",
           "*.laeni.cn",
           "`一般需要将主机名加上`",
           "10.10.1.1",
           "10.10.1.2",
           "127.0.0.1",
           "localhost"
       ],
       "key": { "algo": "rsa", "size": 2048 },
       "names": [
           {
               "C": "CN",
               "ST": "YunNan",
               "L": "KunMing",
               "O": "system:masters"
           }
       ]
   }
   EOF
   
   $ cat <<EOF | tee apiserver-kubelet-client-csr.json
   {
       "CN": "kube-apiserver-kubelet-client",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [
           {
               "C": "CN",
               "ST": "YunNan",
               "L": "KunMing"
           }
       ]
   }
   EOF
   ```

2. 签发证书

   ```shell
   $ cfssl gencert \
     -ca ca.crt `# kubernetes-ca`\
     -ca-key ca.key `# kubernetes-ca对应的私钥`\
     -config ../cfssl-config.json  `# cfssl配置文件`\
     -profile server \
     apiserver-csr.json \
     | cfssljson -bare apiserver
   
   $ cfssl gencert \
     -ca ca.crt \
     -ca-key ca.key \
     -config ../cfssl-config.json \
     -profile client \
     apiserver-kubelet-client-csr.json \
     | cfssljson -bare apiserver-kubelet-client
   ```

3. 捆绑证书并重命名私钥

   ```shell
   $ cat apiserver.pem ca.crt > apiserver.crt && mv apiserver-key.pem apiserver.key
   $ cat apiserver-kubelet-client.pem ca.crt > apiserver-kubelet-client.crt && mv apiserver-kubelet-client-key.pem apiserver-kubelet-client.key
   ```

## 创建front-proxy-client证书

1. 创建`CSRJSON`配置文件

   ```shell
   $ cat <<EOF | tee front-proxy-client-csr.json
   {
       "CN": "front-proxy-client",
       "key": { "algo": "rsa", "size": 2048 },
       "names": [
           {
               "C": "CN",
               "ST": "YunNan",
               "L": "KunMing"
           }
       ]
   }
   EOF
   ```

2. 签发证书

   ```shell
   $ cfssl gencert \
     -ca front-proxy-ca.crt `# kubernetes-front-proxy-ca`\
     -ca-key front-proxy-ca.key `# kubernetes-front-proxy-ca对应的私钥`\
     -config ../cfssl-config.json  `# cfssl配置文件`\
     -profile client \
     front-proxy-client-csr.json \
     | cfssljson -bare front-proxy-client
   ```

3. 捆绑证书并重命名私钥

   ```shell
   $ cat front-proxy-client.pem front-proxy.crt > front-proxy-client.crt && mv front-proxy-client-key.pem front-proxy-client.key
   ```

   至此，所有中间证书就全部生成完成，可以将不再使用的中间文件删除：

   ```shell
   $ rm -rf *.{json,csr} && rm -rf etcd/*.{json,csr}
   ```

# 将生成的所有证书链接或复制到k8s推荐的目录中

```shell
$ cd ..
$ sudo mkdir -p /etc/kubernetes
$ sudo ln -sf $(pwd)/k8s /etc/kubernetes/pki
```

# 生成用户（客户端）配置文件

## 生成各个用户的证书

```shell
$ mkdir client && cd client
$ cat <<EOF | tee admin-csr.json
{
    "CN": "kubernetes-admin",
    "key": { "algo": "rsa", "size": 2048 },
    "names": [
        {
            "C": "CN",
            "ST": "YunNan",
            "L": "KunMing",
            "O": "system:masters"
        }
    ]
}
EOF

$ cat <<EOF | tee kubelet-csr.json
{
    "CN": "system:node:local",`# 格式为"system:node:<nodeName>",<nodeName> 的值 必须 与 kubelet 向 apiserver 注册时提供的节点名称的值完全匹配`
    "key": { "algo": "rsa", "size": 2048 },
    "names": [
        {
            "C": "CN",
            "ST": "YunNan",
            "L": "KunMing",
            "O": "system:nodes"
        }
    ]
}
EOF

$ cat <<EOF | tee kube-proxy-csr.json
{
    "CN": "system:kube-proxy",`# 格式为"system:node:<nodeName>",<nodeName> 的值 必须 与 kubelet 向 apiserver 注册时提供的节点名称的值完全匹配`
    "key": { "algo": "rsa", "size": 2048 },
    "names": [
        {
            "C": "CN",
            "ST": "YunNan",
            "L": "KunMing",
            "O": "system:nodes"
        }
    ]
}
EOF

$ cat <<EOF | tee controller-manager-csr.json
{
    "CN": "system:kube-controller-manager",
    "key": { "algo": "rsa", "size": 2048 },
    "names": [
        {
            "C": "CN",
            "ST": "YunNan",
            "L": "KunMing"
        }
    ]
}
EOF

$ cat <<EOF | tee scheduler-csr.json
{
    "CN": "system:kube-scheduler",
    "key": { "algo": "rsa", "size": 2048 },
    "names": [
        {
            "C": "CN",
            "ST": "YunNan",
            "L": "KunMing"
        }
    ]
}
EOF

$ for item in 'admin' 'kubelet' 'kube-proxy' 'controller-manager' 'scheduler'
do
  cfssl gencert \
      -ca ../ca.crt `# kubernetes-ca`\
      -ca-key ../ca.key `# kubernetes-ca对应的私钥`\
      -config ../../cfssl-config.json  `# cfssl配置文件`\
      -profile client \
      "$item"-csr.json \
      | cfssljson -bare $item
done
```

## 生成配置文件

```shell
$ for item in 'admin' 'kubelet' 'kube-proxy' 'controller-manager' 'scheduler'
do
  touch $item.conf
  KUBECONFIG=$item.conf kubectl config set-cluster k8s-ubuntu --server=https://10.10.1.2:6443 --certificate-authority /etc/kubernetes/pki/ca.crt --embed-certs
  KUBECONFIG=$item.conf kubectl config set-credentials k8s-ubuntu --client-key /etc/kubernetes/pki/client/"$item"-key.pem --client-certificate /etc/kubernetes/pki/client/"$item".pem --embed-certs
  KUBECONFIG=$item.conf kubectl config set-context k8s-ubuntu --cluster k8s-ubuntu --user k8s-ubuntu
  KUBECONFIG=$item.conf kubectl config use-context k8s-ubuntu
  # 链接到 k8s 文档推荐的位置，以便其他人寻找
  sudo ln -s $(pwd)/$item.conf /etc/kubernetes/
done
```



