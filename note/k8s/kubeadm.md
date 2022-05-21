

### 创建kubeadm配置文件

<details>
<summary>kubeadm.yaml</summary>
<pre>
```yaml
apiVersion: kubeadm.k8s.io/v1beta2
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
  advertiseAddress: 172.17.0.1
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: laeni
  # 如果需要在主节点运行其他pod则需要去除"污点"
  taints: []
---
apiServer:
  certSANs: ["127.0.0.1", "localhost", "172.17.0.1", "0.0.0.0"]
  extraArgs:
    enable-admission-plugins: "NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota"
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
# 默认证书存放路径
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: control-plane.minikube.internal:6443
controllerManager:
  extraArgs:
    allocate-node-cidrs: "true"
    leader-elect: "false"
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
    extraArgs:
      proxy-refresh-interval: "70000"
imageRepository: daocloud.io/daocloud
kind: ClusterConfiguration
kubernetesVersion: v1.21.0
networking:
  dnsDomain: cluster.local
  podSubnet: "10.244.0.0/16"
  serviceSubnet: 10.96.0.0/12
scheduler:
  extraArgs:
    leader-elect: "false"
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
authentication:
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
cgroupDriver: systemd
clusterDomain: "cluster.local"
# disable disk resource management by default
imageGCHighThresholdPercent: 100
evictionHard:
  nodefs.available: "0%"
  nodefs.inodesFree: "0%"
  imagefs.available: "0%"
failSwapOn: false
staticPodPath: /etc/kubernetes/manifests
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
clusterCIDR: "10.244.0.0/16"
metricsBindAddress: 172.17.0.1:10249
```
</pre>
</details>
