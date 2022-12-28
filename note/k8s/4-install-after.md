# Kubernetes Metrics Server

```sh
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## 设计

指标服务器是 [Kubernetes 监视体系结构](https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/monitoring_architecture.md)中所述的核心指标管道中的一个组件。

有关详细信息，请参阅：

- [指标 API 设计](https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/resource-metrics-api.md)
- [指标服务器设计](https://github.com/kubernetes/design-proposals-archive/blob/main/instrumentation/metrics-server.md)



