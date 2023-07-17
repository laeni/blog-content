---
title: SpringBoot 版本升级注意事项
tags: 'SpringBoot, SpringCloud, Spring'
author: Laeni
date: '2022-12-01'
updated: '2022-12-01'
---

简单列举平时常用的一些改造点，以及注意事项。

# SpringBoot 2.2.x → 2.7.x

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

# HX老系统改造

## 集成统一日志插件（SpringMVC）

1. `pom.xml`

   ```xml
   <dependency>
       <groupId>com.iboxchain.pub.xlog</groupId>
       <artifactId>public-xlog-core</artifactId>
       <version>1.3.1-RELEASE</version>
   </dependency>
   ```

2. `src/main/webapp/WEB-INF/web.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
   		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   		 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
   		 version="4.0">
   	...
   
   	<!-- region xlog(日志插件) -->
   	<filter>
   		<filter-name>cidFilter</filter-name>
   		<filter-class>com.iboxchain.pub.xlog.core.filter.CidFilter</filter-class>
   		<init-param>
   			<param-name>xlog.staticResourcePatterns</param-name>
   			<param-value>/webjars/**,/**/*.jpeg,/**/*.jpg,/**/*.png,/**/*.ico,/**/*.js,/**/*.css,/**/*.gif,/**/*.woff</param-value>
   		</init-param>
   		<init-param>
   			<param-name>xlog.needPrintError</param-name>
   			<param-value>false</param-value>
   		</init-param>
   		<init-param>
   			<!-- 选填，响应消息日志大小限制，默认10K -->
   			<param-name>xlog.logSizeLimit</param-name>
   			<param-value>10240</param-value>
   		</init-param>
   		<init-param>
   			<!-- 当前应用名称，必选项-->
   			<param-name>xlog.name</param-name>
   			<param-value>APP_NAME</param-value>
   		</init-param>
   	</filter>
   	<filter-mapping>
   		<filter-name>cidFilter</filter-name>
   		<url-pattern>/*</url-pattern>
   	</filter-mapping>
   	<!-- endregion -->
   
       ...
   </web-app>
   ```

   > 注意将`xlog.name`修改为 URL 前缀（一般为项目名），打印日志的时候会拼接在路径的最前面。

3. `src/main/resources/logback.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <configuration scan="false">
       <conversionRule conversionWord="clr" converterClass="com.ips.rcms.log.ColorConverter" />
       <conversionRule conversionWord="wEx" converterClass="com.ips.rcms.log.ExtendedWhitespaceThrowableProxyConverter" />
   
       <!-- region property -->
       <property name="log.base" value="../logs/rcms" />
       <property name="threshold" value="1" />
       <property name="queueSize" value="256" />
       <property name="FILE_LOG_PATTERN" value="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%t] [%level] [%logger{0}] [%X{transactionId}] [%X{spanId}] [%X{timeDiff}] [%X{serviceId}] [%X{protocol}] [%X{logType}] - %m%n" />
       <property name="CONSOLE_LOG_PATTERN" value="%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}" />
       <if condition='property("container").contains("true")'>
           <then>
               <property name="suffix" value="-${HOSTNAME}.log" />
           </then>
           <else>
               <property name="suffix" value=".log" />
           </else>
       </if>
       <!-- endregion -->
   
       <!-- region appender -->
       <!-- region org/springframework/boot/logging/logback/console-appender.xml -->
       <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
           <encoder>
               <pattern>${CONSOLE_LOG_PATTERN}</pattern>
               <charset>UTF-8</charset>
           </encoder>
       </appender>
       <!-- endregion -->
       <!-- 监控 -->
       <appender name="access" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <file>${log.base}/access/access${suffix}</file>
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
               <FileNamePattern>
                   ${log.base}/access/access${suffix}.%d{yyyy-MM-dd-HH}.%i
               </FileNamePattern>
               <TimeBasedFileNamingAndTriggeringPolicy
                       class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                   <MaxFileSize>100MB</MaxFileSize>
               </TimeBasedFileNamingAndTriggeringPolicy>
           </rollingPolicy>
           <encoder>
               <pattern>${FILE_LOG_PATTERN}</pattern>
               <charset>UTF-8</charset>
           </encoder>
       </appender>
       <!-- 应用 -->
       <appender name="app" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <file>${log.base}/app/app${suffix}</file>
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
               <FileNamePattern>
                   ${log.base}/app/app${suffix}.%d{yyyy-MM-dd-HH}.%i
               </FileNamePattern>
               <TimeBasedFileNamingAndTriggeringPolicy
                       class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                   <MaxFileSize>100MB</MaxFileSize>
               </TimeBasedFileNamingAndTriggeringPolicy>
           </rollingPolicy>
           <encoder>
               <pattern>${FILE_LOG_PATTERN}</pattern>
               <charset>UTF-8</charset>
           </encoder>
       </appender>
       <appender name="async-app" class="ch.qos.logback.classic.AsyncAppender">
           <!-- 默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志,如果不希望丢弃日志（既每次都是全量保存），那可以设置为0 -->
           <discardingThreshold>${threshold}</discardingThreshold>
           <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
           <queueSize>${queueSize}</queueSize>
           <!-- 添加附加的appender,最多只能添加一个 -->
           <appender-ref ref="app" />
       </appender>
       <!--接口请求与响应日志  -->
       <appender name="interface" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <file>${log.base}/interface/interface${suffix}</file>
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
               <FileNamePattern>
                   ${log.base}/interface/interface${suffix}.%d{yyyy-MM-dd-HH}.%i
               </FileNamePattern>
               <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                   <MaxFileSize>100MB</MaxFileSize>
               </TimeBasedFileNamingAndTriggeringPolicy>
           </rollingPolicy>
           <encoder>
               <pattern>${FILE_LOG_PATTERN}</pattern>
               <charset>UTF-8</charset>
           </encoder>
       </appender>
       <!-- remote日志 -->
       <appender name="remote" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <file>${log.base}/remote/remote${suffix}</file>
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
               <FileNamePattern>
                   ${log.base}/remote/remote${suffix}.%d{yyyy-MM-dd-HH}.%i
               </FileNamePattern>
               <TimeBasedFileNamingAndTriggeringPolicy
                       class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                   <MaxFileSize>100MB</MaxFileSize>
               </TimeBasedFileNamingAndTriggeringPolicy>
           </rollingPolicy>
           <encoder>
               <pattern>${FILE_LOG_PATTERN}</pattern>
               <charset>UTF-8</charset>
           </encoder>
       </appender>
       <appender name="async-remote" class="ch.qos.logback.classic.AsyncAppender">
           <!-- 默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志,如果不希望丢弃日志（既每次都是全量保存），那可以设置为0 -->
           <discardingThreshold>${threshold}</discardingThreshold>
           <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
           <queueSize>${queueSize}</queueSize>
           <!-- 添加附加的appender,最多只能添加一个 -->
           <appender-ref ref="remote" />
       </appender>
       <!-- 错误日志 -->
       <appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
               <level>WARN</level>
           </filter>
           <file>${log.base}/error/error${suffix}</file>
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
               <FileNamePattern>
                   ${log.base}/error/error${suffix}.%d{yyyy-MM-dd-HH}.%i
               </FileNamePattern>
               <TimeBasedFileNamingAndTriggeringPolicy
                       class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                   <MaxFileSize>100MB</MaxFileSize>
               </TimeBasedFileNamingAndTriggeringPolicy>
           </rollingPolicy>
           <encoder>
               <pattern>${FILE_LOG_PATTERN}</pattern>
               <charset>UTF-8</charset>
           </encoder>
       </appender>
       <appender name="async-error" class="ch.qos.logback.classic.AsyncAppender">
           <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
               <level>WARN</level>
           </filter>
           <discardingThreshold>0</discardingThreshold>
           <queueSize>${queueSize}</queueSize>
           <appender-ref ref="error" />
       </appender>
       <appender name="etl" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <file>${log.base}/etl/etl${suffix}</file>
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
               <FileNamePattern>
                   ${log.base}/etl/etl${suffix}.%d{yyyy-MM-dd-HH}.%i
               </FileNamePattern>
               <TimeBasedFileNamingAndTriggeringPolicy
                       class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                   <MaxFileSize>100MB</MaxFileSize>
               </TimeBasedFileNamingAndTriggeringPolicy>
           </rollingPolicy>
           <encoder>
               <pattern>${FILE_LOG_PATTERN}</pattern>
               <charset>UTF-8</charset>
           </encoder>
       </appender>
       <appender name="mq" class="ch.qos.logback.core.rolling.RollingFileAppender">
           <file>${log.base}/mq/mq${suffix}</file>
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
               <FileNamePattern>
                   ${log.base}/mq/mq${suffix}.%d{yyyy-MM-dd-HH}.%i
               </FileNamePattern>
               <TimeBasedFileNamingAndTriggeringPolicy
                       class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                   <MaxFileSize>100MB</MaxFileSize>
               </TimeBasedFileNamingAndTriggeringPolicy>
           </rollingPolicy>
           <encoder>
               <pattern>${FILE_LOG_PATTERN}</pattern>
               <charset>UTF-8</charset>
           </encoder>
       </appender>
       <!-- endregion -->
   
       <!-- region logger -->
       <logger name="interface" additivity="true">
           <appender-ref ref="interface" />
       </logger>
       <logger name="access" additivity="true">
           <appender-ref ref="access" />
       </logger>
       <!-- 一般除了 interface 和 access 之外可以根据实际情况分开打印 -->
       <logger name="remote" additivity="true">
           <appender-ref ref="async-remote" />
       </logger>
       <logger name="etl" additivity="true">
           <appender-ref ref="etl" />
       </logger>
       <logger name="mq" additivity="true">
           <appender-ref ref="mq" />
       </logger>
       <!-- 这里通过配置文件直接设置日志级别，但如果其他地方（如配置中心）提供该功能，则一般不在此处设置 -->
       <logger name="root" level="INFO" />
       <!-- endregion -->
   
       <root>
           <!-- 统一输出到 app 日志以便统一查看 -->
           <appender-ref ref="async-app" />
           <!-- 特殊情况（如容器或开发调试）需要将日志输出到控制台 -->
           <appender-ref ref="CONSOLE" />
           <!-- 为了快速观察是否有异常（比如巡检），将 WARN 级别以上的日志输出一份的单独的地方 -->
           <appender-ref ref="async-error" />
       </root>
   </configuration>
   ```
   
   > **日志配置经验法则：**
   >
   > 1. 通常，可能会将日志分类打印，但日志的分类不能为了分类而分类，一个好的分类方法是根据日志的作用（比如请求响应日志和access监控日志）分类，而根据日志的打印位置分类并无多达作用，应尽量避免（比如不应该通过模块来划分日志）。日志是为了定位问题的，而分开打印反而不利于问题定位；再者一般情况框架日志不多（如果很多，应该看是否重要，如果不重要应考虑关闭），所以打印日志时应该尽量打印到一起（由于查看日志时一般主要查看app日志，所以全部打印到app日志），如有特殊日志才考虑分开（如请求响应日志等）。
   > 2. logger 的主要目的时为了将某中类型的日志单独打印一份而设置的，单独打印后可以根据需要决定是否往 Root 继续打印（为了能统一查看日志，除非特殊情况，否则建议统一往 Root 输出一份），虽然 logger 可以配置输出级别，但是日志级别一般通过专有的配置进行设置，比如配置中心。
   > 3. 日志重复输出 - 当 logger 直接或间接记录到 Root Appender 并开启 additivity 时会存在重复输出的问题解决方案: 开启 additivity 时，Appender 不能含有直接或间接 Root Appender。





## 去activiti

activiti 是 HTTP 工具类，用于多个服务间的相互调用，用 Feign 取代。

