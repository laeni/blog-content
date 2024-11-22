---
title: Java 开发常见问题及解决方案
description:
author: Laeni
tags: Java, Spring
date: 2024-11-14
updated: 2024-11-14
---

# Spring

## Spring-Data-JPA

### 实体中的`@Column`等注解不生效

原因之一可能是`@Column`注解在同一个实体内存在的位置不统一，比如有的放在字段上，有的却放在 getter 方法上，这会导致放在字段上的注解无效。目前已知 **Spring Data JPA v1.3.0.RELEASE** 有此问题。
