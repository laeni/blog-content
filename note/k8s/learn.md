# Kubernetes组成

Kubernetes集群由两大部分组成：控制平面和实际运行普通应用的工作节点（或负载节点），其中经常把控制平面所在的节点叫Master节点。

![img](./learn/1.jpg)

## 控制平面组件

### API Server

负责对外提供RESTful的kubernetes API 的服务，它是系统管理指令的统一接口（用户以及K8s自身的其他组件都是统一通过*apiserver*进行通行的）是各组件互相通讯的中转站，接受外部请求，并将信息写到ETCD中。kubectl（kubernetes提供的客户端工具，该工具内部是对kubernetes API的调用）就是直接和apiserver交互的。运行方式一般是在容器中运行。

### Controller Manager

如果 API Server 做的是前台的工作的话，那么 Controller Manager 就是负责后台的。它执行集群级功能，例如复制组件，跟踪Node节点，处理节点故障等等。其中每一个资源都对应一个控制器。而control manager就是负责管理这些控制器的。运行方式一般是在容器中运行。

### Schedule

负责Pod的调度，它根据各种条件（如可用的资源、节点的亲和性等）将容器调度到Node上运行。kubernetes目前提供了调度算法，同样也保留了接口。用户可以根据自己的需求自定义调度算法。运行方式一般是在容器中运行。

### Etcd

Etcd是一个高可用的键值存储系统，kubernetes使用它来存储集群的配置和各个资源的状态，实现了Restful的API。运行方式一般是在容器中运行。

## 负载节点组件

### Kublet

在kubernetes集群中，Kublet是**控制平面**在每个Node节点上面的agent，所以每个节点都要运行一个Kubelet服务进程，默认监听*10250*端口，接收并执行**控制平面**的指令，它负责管理该节点上的所有Pod及Pod中的容器（但是如果容器不是kubernetes创建的，它并不会管理）。每个Kubelet进程会向*控制平面*注册其所在节点的信息，定期向*控制平面*汇报该节点的资源使用情况，并通过**cAdvisor**监控节点和容器的资源。本质上，它负责使Pod的运行状态与期望的状态一致。运行方式一般是荣容器运行时一样直接运行在宿主机中。

在kubernetes集群中，每个Node节点上都运行一个Kubelet服务进程，默认监听10250端口，接收并执行Master发来的指令，管理Pod及Pod中的容器。每个Kubelet进程会在API Server上注册所在Node节点的信息，定期向Master节点汇报该节点的资源使用情况，并通过cAdvisor监控节点和容器的资源。

#### Kubelet内部结构

如下kubelet内部组件结构图所示，Kubelet由许多内部组件构成：

![img](./learn/2.jpg)

1. **Kubelet API：**包括 10250 端口的认证 API、4194 端口的 cAdvisor API、10255端口的只读API以及10248端口的健康检查 API。
2. **syncLoop：**从API或者manifest 目录接收 Pod 更新，发送到podWorkers处理，大量使用channel处理来处理异步请求。
3. **辅助的manager：**如cAdvisor、PLEG、Volume Manager等，处理syncLoop以外的其他工作。
4. **CRI**：容器执行引擎接口，负责与container runtime shim通信。
5. **容器执行引擎：**如dockershim、rkt 等（注：rkt 暂未完成CRI的迁移）。
6. **网络插件：**目前支持CNI和kubenet。

#### 了解Kubelet

##### 节点管理

节点管理主要是节点自注册和节点状态更新：

Kubelet可以通过设置启动参数`--register-node`来确定是否向API Server注册自己。如果Kubelet没有选择自注册模式，则需要用户自己配置Node资源信息，同时需要将API Server的位置告知Kubelet。自注册模式下，Kubelet在启动时通过 API Server 注册节点信息，并定时向 API Server 发送节点新消息，API Server在接收到新消息后，将信息写入Etcd中。

##### Pod管理

Kubelet以PodSpec的方式工作，PodSpec 是描述一个 Pod 的YAML或JSON对象。kubelet接受各种机制提供的PodSpecs（主要通过apiserver），并确保这些PodSpecs中描述的Pod正常健康运行。

向Kubelet提供PodSpecs的方法：

**文件：**启动参数--config指定的配置目录下的文件 (默认/ etc/kubernetes/manifests/)。该文件每 20 秒重新检查一次（可配置）。

**HTTP endpoint (URL)：**启动参数--manifest-url设置。每20秒检查一次这个端点（可配置）。

**API Server：**通过API Server监听Etcd目录，同步Pod清单。Kubelet 通过API Server Client(Kubelet启动时创建)使用Watch加List的方式监听"/registry/nodes/$当前节点名"和“/registry/pods”目录，将获取的信息同步到本地缓存中。Kubelet监听Etcd，所有针对Pod的操作都将会被Kubelet监听到。如果发现有新的绑定到本节点的Pod，则按照Pod清单的要求创建该Pod。如果发现本地的Pod被修改，则Kubelet会做出相应的修改，比如删除Pod中某个容器时，则通过Docker Client删除该容器。 如果发现删除本节点的Pod，则删除相应的Pod，并通过Docker Client删除Pod中的容器。

**HTTP server：**kubelet 侦听 HTTP请求，并响应简单的API以提交新的Pod清单。

##### Pod 启动流程

![img](/home/laeni/Documents/Workspace/cn.laeni/blog-content/note/k8s/learn/3.jpg)

启动流程如下：

1）、用户通过Kubectl或其他API客户端提交Pod spec给API Server；

2）、API Server尝试着将Pod对象的相关信息存入Etcd中，待写入操作执行完成，API Server即会返回确认信息至客户端；

3）、API Server开始反映Etcd中的状态变化；

4）、所有的Kubernetes组件均使用Watch机制来跟踪检查API Server上的相关变动；

5）、Kube-scheduler通过其Watch觉察到API Server创建了新的Pod对象但尚未绑定至任何工作节点

6）、Kube-scheduler为Pod对象挑选一个工作节点并将结果信息更新至API Server；

7）、调度结果信息由API Server更新至Etcd，而且API Server也开始反映此Pod对象的调度结果；

8）、Pod被调度到目标工作节点上的kubelet尝试在当前节点上调用Docker启动容器，并将容器的结果状态回送至API Server；

9）、API Server将Pod状态信息存入Etcd中；

10）、在Etcd确认写入操作成功完成后，API Server将确认信息发送至相关的Kubelet，时间将通过它被接受。

**3、容器健康检查**

Pod通过两类探针检查容器的健康状态:

**(1) LivenessProbe（存活探针）探针**：用于判断容器是否健康，告诉Kubelet一个容器什么时候处于不健康的状态。如果LivenessProbe探针探测到容器不健康，则Kubelet 将删除该容器，并根据容器的重启策略做相应的处理。

**(2)ReadinessProbe（就绪探针）探针：**用于判断容器是否启动完成且准备接收请求。如果ReadinessProbe探针探测到失败，则Pod的状态将被修改。

Kubelet定期调用容器中的LivenessProbe探针来诊断容器的健康状况。LivenessProbe包含如下三种实现方式：

**ExecAction：**在容器内部执行一个命令，如果该命令的退出状态码为 0，则表明容器健康；

**TCPSocketAction：**通过容器的IP地址和端口号执行TCP检查，如果端口能被访问，则表明容器健康；

**HTTPGetAction：**通过容器的IP地址和端口号及路径调用HTTP GET方法，如果响应的状态码大于等于200且小于400，则认为容器状态健康。

LivenessProbe和ReadinessProbe探针包含在Pod定义的某个主容器中。

**4、Kubelet驱逐**

Kubelet会监控资源的使用情况，并使用驱逐机制防止计算和存储资源耗尽。在驱逐时，Kubelet将Pod的所有容器停止，并将PodPhase设置为Failed。

**5、cAdvisor资源监控**

cAdvisor是一个开源的分析容器资源使用率和性能特性的代理工具，集成到Kubele中，当Kubelet启动时会同时启动cAdvisor，且一个cAdvisor只监控一个Node节点的信息。cAdvisor自动查找所有在其所在节点上的容器，自动采集CPU、内存、文件系统和网络使用的统计信息。cAdvisor通过它所在节点机的Root容器，采集并分析该节点机的全面使用情况。

**6、Container Runtime**

容器运行时（Container Runtime）是Kubernetes最重要的组件之一，负责真正管理镜像和容器的生命周期。Kubelet通过 容器运行时接口（Container Runtime Interface，CRI) 与容器运行时交互，以管理镜像和容器。

基于CRI容器引擎包括：

1）Docker

2）RKT

3）Virtlet

### Kube-proxy

Kube-proxy实现了Kubernetes中的服务发现和反向代理功能。Kube-proxy支持TCP和UDP连接转发，默认基于Round Robin算法将客户端流量转发到与Service对应的一组后端Pod。服务发现方面，Kube-proxy使用Etcd的Watch机制监控集群中Service和Endpoint对象数据的动态变化，并且维护一个Service到Endpoint的映射关系，从而保证了后端Pod的IP地址变化不会对访问者造成影响，另外，Kube-proxy还支持Session保持。

另外，还有重要的网络部分，它是保证计算机与计算机、资源与资源之间的纽带，实现网络互通，数据传输，例如：Flannel、Calico都是不错的实现方案。

### Container Runtime

容器运行时，如Docker，最主要的功能是下载镜像和运行容器。

# Kubernetes的扩展性

Kubernetes开放了容器运行时接口（CRI）、容器网络接口（CNI）和容器存储接口（CSI），这些接口让Kubernetes的扩展性变得最大化，而Kubernetes本身则专注于容器调度。

- CRI（Container Runtime Interface）：容器运行时接口，提供计算资源，CRI隔离了各个容器引擎之间的差异，而通过统一的接口与各个容器引擎之间进行互动。
- CNI（Container Network Interface）：容器网络接口，提供网络资源，通过CNI接口，Kubernetes可以支持不同网络环境。
- CSI（Container Storage Interface）：容器存储接口，提供存储资源，通过CSI接口，Kubernetes可以支持各种类型的存储。

# Kubernetes中的基本对象
![img](https://support.huaweicloud.com/basics-cce/zh-cn_image_0258869759.png)

- Pod

  Pod是Kubernetes创建或部署的最小单位。一个Pod封装一个或多个容器（container）、存储资源（volume）、一个独立的网络IP以及管理控制容器运行方式的策略选项。

- Deployment

  Deployment是对Pod的服务化封装。一个Deployment可以包含一个或多个Pod，每个Pod的角色相同，所以系统会自动为Deployment的多个Pod分发请求。

- StatefulSet

  StatefulSet是用来管理有状态应用的对象。和Deployment相同的是，StatefulSet管理了基于相同容器定义的一组Pod。但和Deployment不同的是，StatefulSet为它们的每个Pod维护了一个固定的ID。这些Pod是基于相同的声明来创建的，但是不能相互替换，无论怎么调度，每个Pod都有一个永久不变的ID。

- Job

  Job是用来控制批处理型任务的对象。批处理业务与长期伺服业务（Deployment）的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job管理的Pod根据用户的设置把任务成功完成就自动退出（Pod自动删除）。

- CronJob

  CronJob是基于时间控制的Job，类似于Linux系统的crontab，在指定的时间周期运行指定的任务。

- DaemonSet

  DaemonSet是这样一种对象（守护进程），它在集群的每个节点上运行一个Pod，且保证只有一个Pod，这非常适合一些系统层面的应用，例如日志收集、资源监控等，这类应用需要每个节点都运行，且不需要太多实例，一个比较好的例子就是Kubernetes的kube-proxy。

- Service

  Service是用来解决Pod访问问题的。Service有一个固定IP地址，Service将访问流量转发给Pod，而且Service可以给这些Pod做负载均衡。

- Ingress

  Service是基于四层TCP和UDP协议转发的，Ingress可以基于七层的HTTP和HTTPS协议转发，可以通过域名和路径做到更细粒度的划分。

- ConfigMap

  ConfigMap是一种用于存储应用所需配置信息的资源类型，用于保存配置数据的键值对。通过ConfigMap可以方便的做到配置解耦，使得不同环境有不同的配置。

- Secret

  Secret是一种加密存储的资源对象，您可以将认证信息、证书、私钥等保存在Secret中，而不需要把这些敏感数据暴露到镜像或者Pod定义中，从而更加安全和灵活。

- PersistentVolume（PV）

  PV指持久化数据存储卷，主要定义的是一个持久化存储在宿主机上的目录，比如一个NFS的挂载目录。

- PersistentVolumeClaim（PVC）

  Kubernetes提供PVC专门用于持久化存储的申请，PVC可以让您无需关心底层存储资源如何创建、释放等动作，而只需要申明您需要何种类型的存储资源、多大的存储空间。

# 资源对象

## Namespace

Namespace是对一组资源和对象的抽象集合。Namespace 的主要作用是资源分组、资源隔离和权限隔离。Kubernetes 默认创建了两个 Namespace，分别是default和kube-system。当不指定Namespace时，默认为`default` ，可以通过`--namespace`选项特定的命名空间，也可以在 kubeconfig 配置文件中修改默认命名空间，还可以通过`--all-namespaces`选项指定为全部命名空间。

### 创建Namespace

- 根据资源描述文件创建

   1. 定义 Namespace 资源描述

      ```yaml
      apiVersion: v1 
      kind: Namespace 
      metadata: 
        name: custom-namespace 
      ```

   2. 使用kubectl命令创建

      ```sh
      $ kubectl create -f custom-namespace.yaml
      namespace/custom-namespace created 
      ```

- 您还可以使用kubectl create namespace命令创建。

   ```sh
   $ kubectl create namespace custom-namespace 
   namespace/custom-namespace created 
   ```

在指定Namespace下创建资源。

```
$ kubectl create -f nginx.yaml -n custom-namespace 
pod/nginx created 
```



这样在custom-namespace下，就创建了一个名为nginx的Pod。

### Namespace的隔离说明

Namespace只能做到组织上划分，对运行的对象来说，它不能做到真正的隔离。举例来说，如果两个Namespace下的Pod知道对方的IP，而Kubernetes依赖的底层网络没有提供Namespace之间的网络隔离的话，那这两个Pod就可以互相访问。

### 常用命令

1. 查询到当前集群下的Namespace。

   ```sh
   $ kubectl get ns
   NAME               STATUS   AGE
   default            Active   36m
   kube-system        Active   36m
   ...
   ```

2. 查询`kube-system`命令空间下的 Pod。

   ```
   $ kubectl get po --namespace=kube-system
   NAME                                      READY   STATUS    RESTARTS   AGE
   coredns-7689f8bdf-295rk                   1/1     Running   0          9m11s
   ...
   ```

## Pod

Pod是Kubernetes 创建或部署的最小工作单元。每个Pod包含一个或多个容器（container）、存储资源（volume）、一个独立的网络IP以及管理控制容器运行方式的策略选项。Pod 中的容器会作为一个整体被调度到一个Node上运行。一个Pod由一个叫“Pause”的根容器和一个或多个用户自定义的容器组成。

![img](https://pics7.baidu.com/feed/730e0cf3d7ca7bcbc374de54bba32365f724a8ea.jpeg@f_auto?token=a65f4bd47ccc45569c24c14e9c6d02e2&s=713688734BEEC4CE026C126A02004074)

Pod使用主要分为两种方式：

- Pod中运行一个容器。这是Kubernetes最常见的用法，您可以将Pod视为单个封装的容器，但是Kubernetes是直接管理Pod而不是容器。
- Pod中运行多个需要耦合在一起工作、需要共享资源的容器。通常这种场景下应用包含一个主容器和几个辅助容器（SideCar Container），如下图所示，例如主容器为一个web服务器，从一个固定目录下对外提供文件服务，而辅助容器周期性的从外部下载文件存到这个固定目录下。
  ![img](https://support.huaweicloud.com/basics-cce/zh-cn_image_0258392378.png)

实际使用中很少直接创建Pod，而是使用Kubernetes中称为Controller的抽象层来管理Pod实例，例如Deployment和Job。Controller可以创建和管理多个Pod，提供副本管理、滚动升级和自愈能力。通常，Controller会使用Pod Template来创建相应的Pod。

### 资源结构

Pod 资源由`TypeMeta`、`ObjectMeta`、`PodSpec`和`PodStatus`四部分组成（参见[type Pod struct](https://github.com/kubernetes/kubernetes/blob/a0b69ecd01edc68f9eb88658edcb9f82daf27883/staging/src/k8s.io/api/core/v1/types.go#L3966)）：

1. `TypeMeta`只有`Kind`和`APIVersion`两个属性，且直接内联在根路径下，表明资源的类型及其版本。该部分在所有资源中都存在。
2. `ObjectMeta`在`metadata`字段下，存储资源的基本信息，如资源的名称、资源所属命名空间和资源的标签等。该部分在所有资源中都存在。
3. `PodSpec`在`spec`字段下，表示该一个 Pod 期望达到的状态。
4. `PodStatus`在`status`字段下，表示 Pod 的实际状态，我们在创建 Pod 资源描述文件时不填写该字段。

一个 Pod 资源描述文件可以如下：

```yaml
apiVersion: v1                      # Kubernetes的API Version
kind: Pod                           # Kubernetes的资源类型
metadata:
  name: nginx                       # Pod的名称
spec:                               # Pod的具体规格（specification）
  containers:
  - image: nginx:alpine             # 使用的镜像为 nginx:alpine
    name: container-0               # 容器的名称
    resources:                      # 申请容器所需的资源
      limits:
        cpu: 100m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 200Mi
  imagePullSecrets:                 # 拉取镜像使用的证书
  - name: default-secret
```

### 常用Pod操作命令

```sh
$ kubectl create -f nginx.yaml   # 根据 nginx.yaml 资源描述文件创建 Pod
$ kubectl get pods               # 查询所有 Pod
$ kubectl get pod nginx -o yaml  # 查看名为 nginx 的 Pod，-o yaml表示以YAML格式返回，还可以使用-o json，以JSON格式返回
$ kubectl describe pod nginx     # 查看名为 nginx 的 Pod 的详情
$ kubectl delete po nginx pod2   # 删除名为 nginx 和 pod2 的 Pod
$ kubectl delete po --all        # 删除全部 Pod
$ kubectl delete po -l app=nginx # 删除标签为 app=nginx 的 Pod
```

### Pod容器的环境变量

可以通过配置`spec.containers[].env`字段设置容器的环境变量：

```yaml
apiVersion: v1
kind: Pod
...
spec:
    containers:
    - image: nginx:alpine
      name: container-0
      env:                            # 环境变量
      - name: env_key
        value: env_value
```

环境变量还可以引用ConfigMap和Secret，具体使用方法请参见[在环境变量中引用ConfigMap](https://support.huaweicloud.com/basics-cce/kubernetes_0020.html#kubernetes_0020__section18458337406)和[在环境变量中引用Secret](https://support.huaweicloud.com/basics-cce/kubernetes_0021.html#kubernetes_0021__section66761159194018)。

### 容器启动命令

一般制作镜像的时候就指定了容器启动命令，但是也可以覆盖默认的命令：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: container-0
    command:                     # 启动命令
    - top
    - "-b"
```

### 容器的生命周期

Kubernetes提供了[容器生命周期钩子](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/)，在容器的生命周期的特定阶段执行调用，比如容器在停止前希望执行某项操作，就可以注册相应的钩子函数。目前提供的生命周期钩子函数如下所示。

- 启动后处理（PostStart）：容器启动后触发。
- 停止前处理（PreStop）：容器停止前触发。

实际使用时，只需配置Pod的lifecycle.postStart或lifecycle.preStop参数，如下所示。

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: container-0
    lifecycle:
      postStart:                 # 启动后处理
        exec:
          command:
          - "/postStart.sh"
      preStop:                   # 停止前处理
        exec:
          command:
          - "/preStop.sh"
```

### 探针

#### 就绪探针（Readiness Probe）

[就绪探针（Readiness Probe）](https://support.huaweicloud.com/basics-cce/kubernetes_0026.html)

#### 存活探针（Liveness Probe）

Kubernetes提供了自愈的能力，具体就是能感知到容器崩溃，然后能够重启这个容器。但是有时候例如Java程序内存泄漏了，程序无法正常工作，但是JVM进程却是一直运行的，对于这种应用本身业务出了问题的情况，Kubernetes提供了Liveness Probe机制，通过检测容器响应是否正常来决定是否重启，这是一种很好的健康检查机制。

Kubernetes支持如下三种探测机制。

- HTTP GET：向容器发送HTTP GET请求，如果Probe收到2xx或3xx，说明容器是健康的。

  HTTP GET方式是最常见的探测方法，其具体机制是向容器发送HTTP GET请求，如果Probe收到2xx或3xx，说明容器是健康的，定义方法如下所示。

  ```
  apiVersion: v1
  kind: Pod
  ...
  spec:
    containers:
    - name: liveness
      image: nginx:alpine
      livenessProbe:           # liveness probe
        httpGet:               # HTTP GET定义
          path: /
          port: 80
  ```

- TCP Socket：尝试与容器指定端口建立TCP连接，如果连接成功建立，说明容器是健康的。

  TCP Socket 定义方法如下所示。

  ```
  apiVersion: v1
  kind: Pod
  ...
  spec:
    containers:
    - name: liveness
      image: nginx:alpine
      livenessProbe:           # liveness probe
        tcpSocket:
          port: 80
  ```

- Exec：Probe执行容器中的命令并检查命令退出的状态码，如果状态码为0则说明容器是健康的。

  Exec即执行具体命令，具体机制是Probe执行容器中的命令并检查命令退出的状态码，如果状态码为0则说明健康，定义方法如下所示。

  ```
  apiVersion: v1
  kind: Pod
  ...
  spec:
    containers:
    - name: liveness
      image: nginx:alpine
      args:
      - /bin/sh
      - -c
      - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
      livenessProbe:           # liveness probe
        exec:                  # Exec定义
          command:
          - cat
          - /tmp/healthy
  ```

  上面定义中，30秒后命令会删除/tmp/healthy，这会导致Liveness Probe判定Pod处于不健康状态，然后会重启容器。

##### 高级配置

如果查看 Pod 详情，回显中可能如下行所示：

```
Liveness: http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3
```

这一行表示Liveness Probe的具体参数配置，其含义如下：

- delay：延迟，delay=0s，表示在容器启动后立即开始探测，没有延迟时间
- timeout：超时，timeout=1s，表示容器必须在1s内进行响应，否则这次探测记作失败
- period：周期，period=10s，表示每10s探测一次容器
- success：成功，#success=1，表示连续1次成功后记作成功
- failure：失败，#failure=3，表示连续3次失败后会重启容器

以上存活探针表示：容器启动后立即进行探测，如果1s内容器没有给出回应则记作探测失败。每次间隔10s进行一次探测，在探测连续失败3次后重启容器。

这些是创建时默认设置的，也可以手动配置：

```
apiVersion: v1
kind: Pod
...
spec:
  containers:
  - name: liveness
    image: nginx:alpine
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10    # 容器启动后多久开始探测
      timeoutSeconds: 2          # 表示容器必须在2s内做出相应反馈给probe，否则视为探测失败
      periodSeconds: 30          # 探测周期，每30s探测一次
      successThreshold: 1        # 连续探测1次成功表示成功
      failureThreshold: 3        # 连续探测3次失败表示失败
```

initialDelaySeconds一般要设置大于0，这是由于很多情况下容器虽然启动成功，但应用就绪也需要一定的时间，需要等就绪时间之后才能返回成功，否则就会导致probe经常失败。

##### 存活探针注意事项

- 一个好的Liveness Probe应该检查应用内部所有关键部分是否健康，并使用一个专有的URL访问，例如/health，当访问/health 时执行这个功能，然后返回对应结果。
- 检查只能限制在应用内部，不能检查依赖外部的部分，例如当前端web server不能连接数据库时，这个就不能看成web server不健康。
- Liveness Probe不能占用过多的资源，且不能占用过长的时间，否则所有资源都在做健康检查，这就没有意义了。例如Java应用，就最好用HTTP GET方式，如果用Exec方式，JVM启动就占用了非常多的资源。

### Label

当资源变得非常多的时候，如何分类管理就非常重要了，Kubernetes提供了一种机制来为资源分类，那就是Label（标签）。Label非常简单，但是却很强大，Kubernetes中几乎所有资源都可以用Label来组织。

Label的具体形式是key-value的标记对，可以在创建资源的时候设置，也可以在后期添加和修改。

以Pod为例，当Pod变得多起来后，就显得杂乱且难以管理，如下图所示。
![img](https://support.huaweicloud.com/basics-cce/zh-cn_image_0262265504.png)

如果我们为Pod打上不同标签，那情况就完全不同了，如下图所示。
![img](https://support.huaweicloud.com/basics-cce/zh-cn_image_0262267029.png)

#### 添加Label

Label的形式为key-value形式，如下，为Pod设置了app=nginx和env=prod两个Label。

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:                     # 为Pod设置两个Label    
    app: nginx    
    env: prod
...
```

对已存在的Pod，可以直接使用kubectl label命令直接添加Label。

```
$ kubectl label pod nginx env=prod
pod/nginx labeled
```

Pod有了Label后，在查询Pod的时候带上`--show-labels`就可以看到Pod的Label。

```
$ kubectl get pod --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
nginx             1/1     Running   0          50s   app=nginx,env=prod
```

还可以使用`-L`只查询某些固定的Label。

```
$ kubectl get pod -L app,env 
NAME              READY   STATUS    RESTARTS   AGE   APP     ENV
nginx             1/1     Running   0          1m    nginx   prod
```

#### 修改Label

对于已存在的Label，如果要修改的话，需要在命令中带上`--overwrite`，如下所示。

```
$ kubectl label pod nginx env=debug --overwrite
pod/nginx labeled

$ kubectl get pod --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
nginx             1/1     Running   0          50s   app=nginx,creation_method=manual,env=debug
```

## Controller

负责集群内的Node、Pod副本、服务端点（Endpoint）、命名空间（Namespace）、服务账号（ServiceAccount）、资源定额（ResourceQuota）的管理，当某个Node意外宕机时，Controller Manager会及时发现并执行自动化修复流程，确保集群始终处于预期的工作状态。Kubernetes 提供了多种 Controller：Replication Controller（副本控制器）、Node Controller、ResourceQuota Controller、Namespace Controller、Endpoint Controller和Service Controller等。

## Deployment

通过在Deployment中描述你所期望的集群状态，Deployment Controller会将现在的集群状态在一个可控的速度下逐步更新成你所期望的集群状态。Deployment主要职责同样是为了保证pod的数量和健康，90%的功能与Replication Controller完全一样，可以看作新一代的Replication Controller。

## Job

用于调配pod对象运行一次性任务，容器中的进程在正常运行结束后不会对其进行重启，而是将pod对象置于completed状态。若容器中的进程因错误而终止，则需要依据配置确定重启与否，未运行完成的pod对象因其所在的节点故障而意外终止后会被重新调度。从程序的运行形态上来区分，我们可以将Pod分为两类：长时运行服务（jboss、mysql等）和一次性任务（数据计算、测试）。RC创建的Pod都是长时运行服务，而Job创建的Pod都是一次性任务。

## Service

Kubernetes中的核心要素Service提供了一套简化的服务代理和发现机制，天然适应微服务架构。Kubernetes分配给Service的固定IP是一个虚拟IP，并不是一个真实的IP，在外部是无法寻址的。真实的系统实现上，Kubernetes是通过Kube-proxy组件来实现的虚拟IP路由及转发。所以在之前集群部署的环节上，我们在每个Node上均部署了Proxy这个组件，从而实现了Kubernetes层级的虚拟转发网络、Service代理外部服务、Service内部负载均衡以及发布Service等功能。

## ConfigMap

很多生产环境中的应用程序配置较为复杂，可能需要多个config文件、命令行参数和环境变量的组合。并且，这些配置信息应该从应用程序镜像中解耦出来，以保证镜像的可移植性以及配置信息不被泄露。社区引入ConfigMap这个API资源来满足这一需求。ConfigMap包含了一系列的键值对，用于存储被Pod或者系统组件（如controller）访问的信息。这与secret的设计理念有异曲同工之妙，它们的主要区别在于ConfigMap通常不用于存储敏感信息，而只存储简单的文本信息。

