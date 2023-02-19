---
title: 'K8s 中 POD 的 DNS 策略及配置'
author: 'Laeni'
tags: 'k8s, dns, coredns'
date: '2023-02-18'
updated: '2023-02-18'
---

POD 具体使用哪个 DNS 服务器，主要由**DNS策略**（`dnsPolicy`）和**配置**（`dnsConfig`）决定，比如下面的资源：

```yaml
apiVersion: apps/v1
kind: Deployment
...
spec:
  ...
  template:
    ...
    spec:
      ...
      dnsPolicy: None
      dnsConfig:
        options:
          - name: ndots
            value: '5'
          - name: timeout
            value: '3'
        nameservers:
          - 10.2.3.4
        searches:
          - my.dns.search.suffix
```

其中有4种**DNS策略**，区别如下：

| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| ClusterFirst            | 使用集群DNS服务。这是**非主机网络模式**（`hostNetwork`为`false`）的默认值。由于集群DNS一般会级联宿主机使用的DNS服务，而宿主机DNS一般能解析外部域名。所以这种场景下，容器既能够解析service注册的集群内部域名，也能够解析发布到互联网上的外部域名。这时`dnsConfig`配置时可选的，但是如果存在，则会与不存在时生存的配置进行合并，相当于对域名解析配置进行**追加**。 |
| Default                 | 使用宿主机的DNS服务（*严格来讲是使用kubelet的“--resolv-conf”参数指向的域名解析文件，但该文件一般是宿主机的`/etc/resolv.conf`*）。这是**主机网络模式**（`hostNetwork`为`true`）的默认值。这种场景下，默认只能解析注册到互联网上的外部域名，无法解析集群内部域名。 |
| ClusterFirstWithHostNet | 由于主机网络模式（`hostNetwork`为`true`）的默认值是**Default**，所以如果想使用集群DNS服务，可使用本策略。 |
| None                    | 完全使用`dnsConfig`字段的自定义配置，即默认配置为空。且该模式下必须配置`dnsConfig`字段。 |

> 注意：非主机网络模式下的默认DNS策略为*ClusterFirst*，而非*Default*。并且以上任意策略都可以配置`dnsConfig`字段，配置后将与默认配置进行合并，合并规则如下。

# dnsConfig 配置合并规则

dnsConfig 下有`options`、`nameservers`和`searches`子项，但是他们合并规则是不一样的。

| 项目        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| options     | 如果冲突则优先取配置的。                                     |
| nameservers | 如果超限（最多**3个**）则优先取`dnsPolicy`对应规则生成的默认配置。 |
| searches    | 如果超限（最多**6个**）则优先取`dnsPolicy`对应规则生成的默认配置。 |

# 参考文档

- [【华为云】工作负载DNS配置说明](https://support.huaweicloud.com/usermanual-cce/cce_10_0365.html)