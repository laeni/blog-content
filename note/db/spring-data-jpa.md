---
title: Spring Data JPA
author: 'Laeni'
tags: 'Java, Spring, Spring Data, JPA'
date: '2022-11-03'
updated: '2022-11-03'
---

# 多数据源配置

JPA 中多数据源配置的核心在于每个数据都有对应的`EntityManagerFactory`和`LocalContainerEntityManagerFactoryBean`两个Bean，然后再通过`@EnableJpaRepositories`注解指定每个数据源对应的`EntityManagerFactory`和`LocalContainerEntityManagerFactoryBean`即可。具体实现可以参考[官方示例项目 - spring-projects/spring-data-examples](https://github.com/spring-projects/spring-data-examples/tree/main/jpa/multiple-datasources)。

如果需要为每个数据源定制JPA或者Hibernate配置的话，可以参考`JpaBaseConfiguration`及其已有实现进行配置，比如针对某个数据源`DdlAuto`、根据不同的数据源指定不同的方言和 Ping SQL等。这时可能需要按需配置多个`HibernateProperties`，并根据它创建`JpaVendorAdapter`来构建`EntityManagerFactoryBuilder`（当然也可以直接构建`LocalContainerEntityManagerFactoryBean`并Set`JpaVendorAdapter`）。



