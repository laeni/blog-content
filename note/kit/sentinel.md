---
title: Sentinel常用示例
tags: 'Sentinel, 流量控制, 限流, 熔断, 降级, 热点参数限流'
author: Laeni
date: '2021-08-25'
updated: '2021-08-25'
---

[Sentinel](https://github.com/alibaba/Sentinel)

## 热点参数限流

对于热点参数限流，官方说明如下：

> 何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：
>
> - 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
> - 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制
>
> 热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。
>
> ![Sentinel Parameter Flow Control](F:\Objects\cn.laeni\blog-content\note\kit\sentinel.assets\sentinel-hot-param-overview-1.png)
>
> Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。


