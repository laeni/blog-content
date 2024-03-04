---
title: HX老系统改造
tags: Tomcat, 升级
author: Laeni
date: 2024-01-17
updated: 2023-01-17
hide: true
---

如需升级 Spring，可查看 [Spring 升级指南](../spring/upgrade)。

# 从 Java 7 升级 Java8

## 项目改造

## 服务器配置

1. 安装 JDK8

   ```sh
   sudo su - root
   mkdir -p /opt/appl/java && chown imipay:appl /opt/appl/java
   su - imipay
   cd /opt/appl/java
   curl -LO https://me-1252266447.cos.ap-chengdu.myqcloud.com/Package/java/OpenJDK8U-jdk_x64_linux_hotspot_8u332b09.tar.gz
   tar -xf OpenJDK8U-jdk_x64_linux_hotspot_8u332b09.tar.gz && rm -rf OpenJDK8U-jdk_x64_linux_hotspot_8u332b09.tar.gz
   ln -s jdk8u332-b09 jdk8
   ```

2. 安装 Tomcat

   ```sh
   cd /opt/appl/tomcat
   curl -LO https://me-1252266447.cos.ap-chengdu.myqcloud.com/Package/java/apache-tomcat-8.5.98.tar.gz
   tar -xf apache-tomcat-8.5.98.tar.gz && rm -rf apache-tomcat-8.5.98.tar.gz
   rm -rf /opt/appl/tomcat/apache-tomcat-8.5.98/webapps/*
   ```

3. 配置 Tomcat

   由于 HX 经常在一台服务器部署多个 Tomcat，并且开启 Tomcat 的 https 协议，这就需要根据老版本配置查看监听的 HTTP 端口、HTTPS 端口以及配置 HTTPS 所使用的信息。

   将证书移动到通过路径：

   ```sh
   sudo su - root
   mkdir -p /opt/appl/certs && chown imipay:appl /opt/appl/certs
   su - imipay
   cp /opt/appl/tomcat/xxx/xxx.jks
   ```

   根据老版本配置修改新版本的`conf/server.xml`。

   ![image-20240117095947372](./hx-upgrade.assets/image-20240117095947372.png)

   ![image-20240117100138261](./hx-upgrade.assets/image-20240117100138261.png)

   ![image-20240117100455039](./hx-upgrade.assets/image-20240117100455039.png)

# 集成统一日志插件（SpringMVC）

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
   			<!-- TODO 需要根据实际系统情况列出不打印日志的接口路径，这些接口路径一般是涉及二进制的，如图片等 -->
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
   			<param-name>xlog.name</param-name>
   			<!-- TODO 当前应用名称，建议填写 context-path 值，因为打印日志时会将其拼接到请求路径前面 -->
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

3. `com.xxx.log.ColorConverter`

   ```java
   package com.xxx.log;
   
   import ch.qos.logback.classic.Level;
   import ch.qos.logback.classic.spi.ILoggingEvent;
   import ch.qos.logback.core.pattern.CompositeConverter;
   
   import java.util.Collections;
   import java.util.HashMap;
   import java.util.Map;
   
   /**
    * 拷贝自SpringBoot.
    */
   public class ColorConverter extends CompositeConverter<ILoggingEvent> {
   
       private static final String ENCODE_JOIN = ";";
   
       private static final String ENCODE_START = "\033[";
   
       private static final String ENCODE_END = "m";
   
       private static final String RESET = "0;" + AnsiColor.DEFAULT;
   
       private static final Map<String, AnsiElement> ELEMENTS;
   
       static {
           Map<String, AnsiElement> ansiElements = new HashMap<>();
           ansiElements.put("faint", AnsiStyle.FAINT);
           ansiElements.put("red", AnsiColor.RED);
           ansiElements.put("green", AnsiColor.GREEN);
           ansiElements.put("yellow", AnsiColor.YELLOW);
           ansiElements.put("blue", AnsiColor.BLUE);
           ansiElements.put("magenta", AnsiColor.MAGENTA);
           ansiElements.put("cyan", AnsiColor.CYAN);
           ELEMENTS = Collections.unmodifiableMap(ansiElements);
       }
   
       private static final Map<Integer, AnsiElement> LEVELS;
   
       static {
           Map<Integer, AnsiElement> ansiLevels = new HashMap<>();
           ansiLevels.put(Level.ERROR_INTEGER, AnsiColor.RED);
           ansiLevels.put(Level.WARN_INTEGER, AnsiColor.YELLOW);
           LEVELS = Collections.unmodifiableMap(ansiLevels);
       }
   
       @Override
       protected String transform(ILoggingEvent event, String in) {
           AnsiElement element = ELEMENTS.get(getFirstOption());
           if (element == null) {
               // Assume highlighting
               element = LEVELS.get(event.getLevel().toInteger());
               element = (element != null) ? element : ELEMENTS.get("green");
           }
           return toAnsiString(in, element);
       }
   
       private String toAnsiString(String in, AnsiElement element) {
           StringBuilder sb = new StringBuilder();
           buildEnabled(sb, new Object[]{element, in});
           return sb.toString();
       }
   
       private static void buildEnabled(StringBuilder sb, Object[] elements) {
           boolean writingAnsi = false;
           boolean containsEncoding = false;
           for (Object element : elements) {
               if (element instanceof AnsiElement) {
                   containsEncoding = true;
                   if (!writingAnsi) {
                       sb.append(ENCODE_START);
                       writingAnsi = true;
                   }
                   else {
                       sb.append(ENCODE_JOIN);
                   }
               }
               else {
                   if (writingAnsi) {
                       sb.append(ENCODE_END);
                       writingAnsi = false;
                   }
               }
               sb.append(element);
           }
           if (containsEncoding) {
               sb.append(writingAnsi ? ENCODE_JOIN : ENCODE_START);
               sb.append(RESET);
               sb.append(ENCODE_END);
           }
       }
   
       interface AnsiElement {
           @Override
           String toString();
       }
       enum AnsiStyle implements AnsiElement {
           NORMAL("0"),
           BOLD("1"),
           FAINT("2"),
           ITALIC("3"),
           UNDERLINE("4");
   
           private final String code;
   
           AnsiStyle(String code) {
               this.code = code;
           }
   
           @Override
           public String toString() {
               return this.code;
           }
       }
       enum AnsiColor implements AnsiElement {
           DEFAULT("39"),
           BLACK("30"),
           RED("31"),
           GREEN("32"),
           YELLOW("33"),
           BLUE("34"),
           MAGENTA("35"),
           CYAN("36"),
           WHITE("37"),
           BRIGHT_BLACK("90"),
           BRIGHT_RED("91"),
           BRIGHT_GREEN("92"),
           BRIGHT_YELLOW("93"),
           BRIGHT_BLUE("94"),
           BRIGHT_MAGENTA("95"),
           BRIGHT_CYAN("96"),
           BRIGHT_WHITE("97");
   
           private final String code;
   
           AnsiColor(String code) {
               this.code = code;
           }
   
           @Override
           public String toString() {
               return this.code;
           }
   
       }
   }
   
4. `com.xxx.log.ExtendedWhitespaceThrowableProxyConverter`

   ```java
   package com.xxx.log;
   
   import ch.qos.logback.classic.pattern.ExtendedThrowableProxyConverter;
   import ch.qos.logback.classic.spi.IThrowableProxy;
   import ch.qos.logback.core.CoreConstants;
   
   public class ExtendedWhitespaceThrowableProxyConverter extends ExtendedThrowableProxyConverter {
   
       @Override
       protected String throwableProxyToString(IThrowableProxy tp) {
           return CoreConstants.LINE_SEPARATOR + super.throwableProxyToString(tp) + CoreConstants.LINE_SEPARATOR;
       }
   
   }
   ```

5. `src/main/resources/logback.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <configuration scan="false">
       <!-- TODO 这两个转换器复制自SpringBoot，比如控制台带颜色输出，如不需要可以不配置 -->
       <conversionRule conversionWord="clr" converterClass="com.xxx.log.ColorConverter" />
       <conversionRule conversionWord="wEx" converterClass="com.xxx.log.ExtendedWhitespaceThrowableProxyConverter" />
   
       <!-- region property -->
   		<!-- TODO 当前配置会将日志和 Tomcat 日志打印到一起，也是推荐配置 -->
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
   		<!-- TODO 根据自身系统实际情况增加或删除 -->
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
   		<!-- TODO 根据自身系统实际情况增加或删除 -->
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
       <logger name="remote" additivity="true">
           <appender-ref ref="async-remote" />
       </logger>
   		<!-- TODO 根据自身系统实际情况增加或删除 -->
       <logger name="etl" additivity="true">
           <appender-ref ref="etl" />
       </logger>
   		<!-- TODO 根据自身系统实际情况增加或删除 -->
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
           <!-- 为了快速观察是否有异常（比如巡检），将 WARN 级别以上的日志输出一份到单独的地方 -->
           <appender-ref ref="async-error" />
       </root>
   </configuration>
   ```
   
   > **日志配置经验法则：**
   >
   > 1. 通常，可能会将日志分类打印，但日志的分类不能为了分类而分类，一个好的分类方法是根据日志的作用（比如请求响应日志和access监控日志）分类，而根据日志的打印位置分类并无多达作用，应尽量避免（比如不应该通过模块来划分日志）。日志是为了定位问题的，而分开打印反而不利于问题定位；再者一般情况框架日志不多（如果很多，应该看是否重要，如果不重要应考虑关闭），所以打印日志时应该尽量打印到一起（由于查看日志时一般主要查看app日志，所以全部打印到app日志），如有特殊日志才考虑分开（如请求响应日志等）。
   > 2. logger 的主要目的时为了将某中类型的日志单独打印一份而设置的，单独打印后可以根据需要决定是否往 Root 继续打印（为了能统一查看日志，除非特殊情况，否则建议统一往 Root 输出一份），虽然 logger 可以配置输出级别，但是日志级别一般通过专有的配置进行设置，比如配置中心。
   > 3. 日志重复输出 - 当 logger 直接或间接记录到 Root Appender 并开启 additivity 时会存在重复输出的问题解决方案: 开启 additivity 时，Appender 不能含有直接或间接 Root Appender。

# 去activiti

activiti 是 HTTP 工具类，用于多个服务间的相互调用，用 Feign 取代。

# XML 格式的Spring MVC配置更改

可能有如下的XML格式配置:

```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
    <property name="useSuffixPatternMatch" value="true"/>
</bean>
```

需要将其替换为 Java 配置:

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer.setUseSuffixPatternMatch(true);
    }
}
```

> 可能由于历史原因不得不开启后缀匹配。

# 路径匹配规则配置

老项目可能通过一些奇怪的后缀来匹配，比如`web.xml`中可能有如下配置：

```xml
<!-- web.xml -->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:/spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>*.htm</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>*.json</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>*.html</url-pattern>
</servlet-mapping>
```

而**Controller**的路径可能是不带后缀的，当升级到 **Spring 5.x** 后，默认不进行后缀匹配，所以了兼容老版本用法，要么在每个 Controller 中都配置带后缀和不带后缀的路径，要么通过配置开启路径后缀匹配。

开启方法见[XML 格式的Spring MVC配置更改](#XML 格式的Spring MVC配置更改)。

此外，为了能够使用不带后缀的 Controller，建议将上述 `web.xml` 配置中路径匹配方式更改**匹配所有**。即：

```xml
<!-- web.xml -->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:/spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

这时候，如果项目有静态资源的，需要指定静态资源路径匹配前缀：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:resources mapping="/js/**" location="/js/"/>
    <mvc:resources mapping="/css/**" location="/css/"/>

</beans>
```
