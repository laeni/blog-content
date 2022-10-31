## 网络配置

### [转发 IPv4 并让 iptables 看到桥接流量](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#%E8%BD%AC%E5%8F%91-ipv4-%E5%B9%B6%E8%AE%A9-iptables-%E7%9C%8B%E5%88%B0%E6%A1%A5%E6%8E%A5%E6%B5%81%E9%87%8F)

```bash
# 配置启动时自动加载 overlay 和 br_netfilter 模块
# 通过运行 `lsmod | grep br_netfilter` 来验证 `br_netfilter` 模块是否已加载
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf # 配置文件名称可以随意，只要是 .conf 结尾即可
overlay
br_netfilter
EOF
# 马上加载模块而不重新启动
sudo modprobe overlay
sudo modprobe br_netfilter

# 设置所需的 sysctl 参数，让 Linux 节点的 iptables 能够正确查看桥接流量
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
# 应用 sysctl 参数而不重新启动
sudo sysctl --system
```

### 配置虚拟不可变的IP地址

由于笔记本电脑更换网络后本机ip地址会发生变化，变化后K8S需要重新初始化，所以需要给电脑加一个不会变化的静态ip或者使用域名。如果使用不可变ip地址，则使用系统支持的方式创建一个网桥并分配一个ip即可。

## 环境配置

### 配置 cgroup 驱动程序

`cgroupfs`和`systemd`都是[控制组（CGroup）](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-cgroup)驱动程序，如果容器运行时和`kubelet`分别使用不同的驱动程序时可能导致系统不稳定，所以需要统一使用一种驱动程序。

**说明：**

如果你从软件包（例如，RPM 或者 `.deb`）中安装 containerd，你可能会发现其中默认禁止了 CRI 集成插件。

你需要启用 CRI 支持才能在 Kubernetes 集群中使用 containerd。 要确保 `cri` 没有出现在 `/etc/containerd/config.toml` 文件中 `disabled_plugins` 列表内。如果你更改了这个文件，也请记得要重启 `containerd`。

如果你应用此更改，请确保重新启动 containerd：

```shell
sudo systemctl restart containerd
```

当使用 kubeadm 时，请手动配置 [kubelet 的 cgroup 驱动](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#configuring-the-kubelet-cgroup-driver)。

#### 常见的工具指定 cgroup 驱动程序

1. containerd

   结合 `runc` 使用 `systemd` cgroup 驱动，在 `/etc/containerd/config.toml` 中设置

   ```
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
     ...
     [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
       SystemdCgroup = true
   ```

   重启

   ```shell
   $ sudo systemctl daemon-reload
   $ sudo systemctl restart containerd
   ```

2. docker

   ```shell
   $ sudo mkdir /etc/docker
   $ cat <<EOF | sudo tee /etc/docker/daemon.json
   {
     "exec-opts": ["native.cgroupdriver=systemd"],
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "100m"
     },
     "storage-driver": "overlay2"
   }
   EOF
   ```

   重启

   ```shell
   $ sudo systemctl enable docker
   $ sudo systemctl daemon-reload
   $ sudo systemctl restart docker
   ```

3. kubeadm

   ```shell
   $ cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
   ...
   Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cgroup-driver=systemd"
   ...
   
   cat /var/lib/kubelet/kubeadm-flags.env
   ...
   KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2"
   ...
   ```

   重启

   ```shell
   $ sudo systemctl daemon-reload
   $ sudo systemctl restart kubelet
   ```

### [根据k8s要求配置containerd](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#containerd)

