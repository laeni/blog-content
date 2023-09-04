---
title: K8S常用运维命令备案
author: 'Laeni'
tags: Kubernetes
date: '2022-10-21'
updated: '2022-10-21'
---

# Node

设置节点不可调度：`kubectl cordon <node_name>`，恢复可调度：`kubectl uncordon <node_name>`。

pod驱逐：`kubectl drain <node_name> --delete-local-data --ignore-daemonsets --force`
    –delete-local-data: 即使pod使用了emptyDir也删除
    –ignore-daemonsets: 忽略deamonset控制器的pod，如果不忽略，deamonset控制器控制的pod被删除后可能马上又在此节点上启动起来,会成为死循环
   –force: 不加force参数只会删除该NODE上由ReplicationController, ReplicaSet, DaemonSet,StatefulSet or Job创建的Pod，加了后还会删除’裸奔的pod’(没有绑定到任何replication controller)

