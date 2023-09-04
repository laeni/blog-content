---
title: Spring 版本升级注意事项
tags: 'SpringBoot, SpringCloud, Spring'
author: Laeni
date: '2022-12-01'
updated: '2023-07-17'
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

# SpringMVC → SpringBoot 2.7.x

## Spring 配置

1. 启动类改造

   ```java
   @EnableScheduling
   @EnableTransactionManagement
   @EnableAsync(proxyTargetClass = true)
   @ImportResource("classpath:applicationConfigContext.xml")
   @SpringBootApplication(scanBasePackages = {"com.ips.rcms", "com.ips.pfas", "com.iboxchain.pub.xlog"})
   public class RcmsApplication extends SpringBootServletInitializer {
   
       public static void main(String[] args) {
           SpringApplication.run(RcmsApplication.class);
       }
   
       @Override
       protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
           return builder.sources(RcmsApplication.class);
       }
   }
   ```

   该启动类有两个方法，如果是直接启动则使用`main`方法；如果是打成*war*包放*Tomcat*等容器中启动则需要继承`SpringBootServletInitializer`并覆写`configure`方法。

2. 删除不再使用的 XML 配置`context:annotation-config`

   ```xml
   <beans>
       <context:annotation-config/>
   </beans>
   ```

3. 转换`ReloadableResourceBundleMessageSource`Bean。

   可能有如下配置:

   ```
   <bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
       <property name="useCodeAsDefaultMessage" value="true"/>
       <property name="basenames">
           <array>
               <value>classpath:/msg/message</value>
           </array>
       </property>
       <property name="defaultEncoding" value="UTF-8"/>
   </bean>
   ```

   删除上面的Bean定义后转换为如下配置:

   ```properties
   spring.messages.basename=classpath:/msg/message
   ```

4. 转换`CommonsMultipartResolver`Bean。

   可能有如下配置:

   ```
   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
       <property name="maxUploadSize" value="200000000"/>
   </bean>
   ```
   
   删除上面的Bean定义后转换为如下配置:
   
   ```properties
   spring.servlet.multipart.max-file-size=200MB
   ```

## Quartz

1. 删除`<task:annotation-driven ... />`

## MVC

1. 将`WebMvcConfigurationSupport`配置使用`WebMvcConfigurer`*Bean*代替。

   删除：

   ```java
   @Configuration
   class DefaultMvcConfiguration extends WebMvcConfigurationSupport {
   
       @Override
       protected void configurePathMatch(PathMatchConfigurer configurer) {
           // 新版本 Spring 默认关闭且🙈推荐使用，但是老项目需要兼容支持
           configurer.setUseSuffixPatternMatch(true)
                   // 设置是否自动后缀留级匹配模式，如“/user”是否匹配“/user/”，为true是即匹配
                   .setUseTrailingSlashMatch(true);
       }
   
   }
   ```
   
   新增：
   
   ```java
   @Bean
   WebMvcConfigurer webMvcConfigurer() {
       ......
   }
   ```
   
1. 添加路径匹配配置

   ```yml
   spring:
     mvc:
       pathmatch:
         use-suffix-pattern: true
         matching-strategy: ANT_PATH_MATCHER
   ```
   
   > 虽然新版已经不建议使用后缀匹配模式，但是根据惯例，SpringMVC项目很喜欢使用诸如`xxx.do`或者`xxx.htm`的后端接口，如果Controller路径上不包含这些后缀会导致*404*问题。
   
1. 删除`<mvc:annotation-driven />`

1. 静态资源映射配置。

   SpringMVC中可能会做如下配置：

   ```xml
   <mvc:resources mapping="/bootstrap/**" location="/bootstrap/"/>
   <mvc:resources mapping="/commonJS/**" location="/commonJS/"/>
   <mvc:resources mapping="/images/**" location="/images/"/>
   <mvc:resources mapping="/jquery/**" location="/jquery/"/>
   <mvc:resources mapping="/js/**" location="/js/"/>
   ```
   
   SpringBoot虽然也可以像上面一样做路径映射，但是更好的做法应该是将静态资源移动到SpringBoot约定的静态资源目录（参见: `spring.web.resources.static-locations`的默认值）下：
   
   ```sh
   mkdir resources/static
   mv webapp/bootstrap webapp/commonJS webapp/images webapp/jquery webapp/js resources/static/
   ```

## 数据库

将

```xml
<beans>
    <tx:annotation-driven ... />
</beans>
```

替换为*SpringBoot*注解

```java
@EnableTransactionManagement
@SpringBootApplication
public class Application {
    ...
}
```



## 去除 `web.xml`

SpringBoot 已经不在需要`web.xml`，但是原始文件可能存在很多配置，所以先一步一步对`web.xml`进行转换，直到该文件为空后删除。

1. 删除标签`<display-name>`、`<context-param>`（一般情况下直接删除即可）。

2. 删除*listener*-`ContextLoaderListener`（一般情况下直接删除即可）。

   ```xml
   <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>
   ```

3. 删除*servlet*-`DispatcherServlet`（一般情况下直接删除即可）。

   ```xml
   <servlet>
       <servlet-name>dispatcher</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <init-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:/applicationConfigContext.xml</param-value>
       </init-param>
       <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
       <servlet-name>dispatcher</servlet-name>
       <url-pattern>/</url-pattern>
   </servlet-mapping>
   ```

4. 处理`CharacterEncodingFilter`。

   ```xml
   <filter>
       <filter-name>encodingFilter</filter-name>
       <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
       <init-param>
           <param-name>encoding</param-name>
           <param-value>UTF-8</param-value>
       </init-param>
       <init-param>
           <param-name>forceEncoding</param-name>
           <param-value>true</param-value>
       </init-param>
   </filter>
   <filter-mapping>
       <filter-name>encodingFilter</filter-name>
       <url-pattern>/*</url-pattern>
       <dispatcher>REQUEST</dispatcher>
       <dispatcher>FORWARD</dispatcher>
   </filter-mapping>
   ```

   以上配置对应SpringBoot配置：

   ```yaml
   server:
     servlet:
       encoding:
         charset: UTF-8
         force: true
   ```

   > SpringBoot默认已经将字符集设置为`UTF-8`，所以如果不需要更改为其他字符集可以省略`server.servlet.encoding.charset`配置；一般情况也不需要强制使用指定编码，所以`server.servlet.encoding.force`也可以省略。即大部分情况可以直接删除该*filter*即可。

5. 其他*servlet*转换为*SpringBoot*方式配置。

   例如有如下*servlet*：

   ```xml
   <servlet>
       <servlet-name>PackServlet</servlet-name>
       <servlet-class>net.sf.packtag.servlet.PackServlet</servlet-class>
   </servlet>
   <servlet-mapping>
       <servlet-name>PackServlet</servlet-name>
       <url-pattern>*.pack</url-pattern>
   </servlet-mapping>
   ```

   转换为*SpringBoot*方式：

   ```java
   import net.sf.packtag.servlet.PackServlet;
   
   @Configuration
   class MigrateToSpringBootConfiguration {
       @Bean
       ServletRegistrationBean<PackServlet> packServlet(){
           ServletRegistrationBean<PackServlet> bean = new ServletRegistrationBean<>(new PackServlet());
           bean.setName("PackServlet");
           bean.setUrlMappings(Collections.singletonList("*.pack"));
           //bean.setLoadOnStartup(10);
           return bean;
       }
   }
   ```

   > 注意事项：
   >
   > 1. 有些*servlet*可能会自动注册。
   > 2. 如果*servlet*在实例化时使用`ApplicationContext.getBean`获取*Bean*，则在目标*Bean*还不存在时可能会报错，这种情况建议将*servlet*依赖的*Bean*改为构造函数注入，然后在*SpringBoot*配置时注入该*Bean*后再实体化*servlet*。

6. 其他*servlet*转换为*SpringBoot*方式配置。

   *filter*和*servlet*一样，也有*SpringBoot*的方式注册。

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





## 去activiti

activiti 是 HTTP 工具类，用于多个服务间的相互调用，用 Feign 取代。

