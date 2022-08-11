本笔记记录使用[Kind](https://kind.sigs.k8s.io/)创建K8s集群。由于安装简单不再记录，所以记录一些常用配置（默认情况下可以不需要配置），其他配置请参考[官方文档](https://kind.sigs.k8s.io/docs/user/configuration/)或者[Go结构体](https://pkg.go.dev/sigs.k8s.io/kind/pkg/apis/config/v1alpha4#Cluster)。

在合适的目录创建任意命名的YAML配置文件，如`config.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  # 虽然官方强烈建议将其保留为默认值 (127.0.0.1)，但是非生产环境，为了方便可以修改。
  apiServerAddress: "0.0.0.0"
  # 默认情况下，API 服务器侦听随机开放端口。
  # 你可以选择一个特定的端口，但在大多数情况下可能不需要。
  # 使用随机端口可以更轻松地启动多个集群。
  # 但是我们这里并不需要启动多个集群，并且希望他与k8s官网尽量靠近，所以指定为6443。
  apiServerPort: 6443
  # POD 子网
  podSubnet: "10.244.0.0/16"
  # 服务子网
  serviceSubnet: "10.96.0.0/16"
# 可以指定多个节点，并且详细配置每个节点，默认情况下只有一个名为"control-plane"的节点
#nodes:
#- role: control-plane
#- role: worker
#  # 可以通过设置容器镜像来设置特定的 Kubernetes 版本，可以在版本页面上找到可用的镜像标签
#  # 版本页面 - https://github.com/kubernetes-sigs/kind/releases
#  # 注意需要包含镜像摘要
#  image: kindest/node:v1.24.0@sha256:0866296e693efe1fed79d5e6c7af8df71fc73ae45e3679af05342239cdc5bc8e
```

创建集群时使用指定的配置文件：

```sh
$ kind create cluster --config=config.yaml
```

> 除非另有说明，否则传递给 CLI 的参数优先于配置文件中的等效参数。