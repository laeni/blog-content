---
title: SpringBoot 版本升级注意事项
tags: 'SpringBoot, SpringCloud, Spring'
author: Laeni
date: '2022-12-01'
updated: '2022-12-01'
---

简单列举平时常用的一些改造点，以及注意事项。

# SpringBoot 2.2.5 → 2.7.3

## POM

升级相关依赖，其中至少包含如下Spring依赖。

```xml
- <parent>
-     <groupId>org.springframework.boot</groupId>
-     <artifactId>spring-boot-starter-parent</artifactId>
-     <version>2.2.5.RELEASE</version>
- </parent>
<dependencies>
-    <!-- 如果有也建议去掉 -->
-     <dependency>
-         <groupId>org.springframework.cloud</groupId>
-         <artifactId>spring-cloud-starter-bootstrap</artifactId>
-     </dependency>
</dependencies>

+ <dependencyManagement>
+     <dependencies>
+         <!-- 如果有SpringCloud，需一并升级 -->
+         <dependency>
+             <groupId>org.springframework.cloud</groupId>
+             <artifactId>spring-cloud-dependencies</artifactId>
+             <version>2021.0.4</version>
+             <type>pom</type>
+             <scope>import</scope>
+         </dependency>
+         <dependency>
+             <groupId>org.springframework.boot</groupId>
+             <artifactId>spring-boot-starter-parent</artifactId>
+             <version>2.7.3</version>
+             <type>pom</type>
+             <scope>import</scope>
+         </dependency>
+     </dependencies>
+ </dependencyManagement>
```

## bootstrap.yml

新版默认将`bootstrap.yml`配置文件去掉了，该文件的内容需要与`application.yml`合并，如果有`spring-cloud-starter-bootstrap`依赖也一并去掉。

## JPA

如果 JPA 方法带有命名参数查询方法，需要挨个检查，并将`@Param`注解补充完整，因为老版本能根据字节码确认形参名称，而新版本不再解析字节码，而是根据`@Param`注解和 javac 文档注释来确认形参名称，但 javac 方式不靠谱，所以建议全部使用`@Param`注释明确标注。

反例：

```java
@Query("delete from Xxx where name = :name")
void deleteByName(String name);
```

正例：

```java
@Query("delete from Xxx where name = :name")
void deleteByName(@org.springframework.data.repository.query.Param("name") String name);
```

## SpringMvc

新版Spring默认使用`PathPatternParser`路径匹配器，而非`AntPathMatcher`，但是`PathPatternParser`匹配器不支持开启后缀模式，所以`spring.mvc.pathmatch.use-suffix-pattern`选项也一起弃用。

所以升级之后要么通过配置`spring.mvc.pathmatch.matching-strategy: ANT_PATH_MATCHER`继续使用`AntPathMatcher`，要么需要排查（比如根据历史日志）是否有类似`party1//party2`**（注意中间有双斜杠）**、`party.htm`、`party.xxx`，如果有这种情况，需要明确处理，否则可能`404`。

## 单元测试

- 包导入

  ```java
  - import org.junit.Test;
  + import org.junit.jupiter.api.Test;
  
  - import static org.junit.Assert.*;
  + import static org.junit.jupiter.api.Assertions.*;
  ```

- 测试启动类上删除注解`@RunWith(SpringRunner.class)`。

