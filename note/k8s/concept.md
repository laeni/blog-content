---
title: 'K8s相关概念或名词解释'
author: 'Laeni'
tags: Kubernetes, DataSource
date: '2022-06-16'
updated: '2022-06-16'
---

## Status（状态）

Terminating - 终止中
NotReady - 未就绪
SchedulingDisabled - 已经禁止调度计划，不再将新的POD调度到该Node上
cordon - 禁止调度，将一个Node标记为`SchedulingDisabled`
Pending - 悬而未决的，一般表示一个POD正在等待调度到Node上，可能由于现有Pod不满足要求

## Qos

Guaranteed - 放心
BestEffort - 最佳努力