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

5. `class Xxx implements WebBindingInitializer`改造。

   假设有如下配置:

   ```java
   @Component
   public class GlobalBindingInitializer implements WebBindingInitializer {
   	@Override
   	public void initBinder(WebDataBinder binder) {
   		// ......
   	}
   }
   ```

   或者`MultiActionController`子类的形式:

   ```java
   class _BaseController extends MultiActionController {
       @Override
   	protected void initBinder(HttpServletRequest request, ServletRequestDataBinder binder) throws Exception {
           // ......
       }
   }
   ```
   
   将上面的配置修改为：
   
   ```java
   @ControllerAdvice
   public class GlobalBindingInitializer {
   	@InitBinder
   	public void initBinder(WebDataBinder binder) {
   		// ......
   	}
   }
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

2. 添加路径匹配配置

   ```yml
   spring:
     mvc:
       pathmatch:
         use-suffix-pattern: true
         matching-strategy: ANT_PATH_MATCHER
   ```

   > 虽然新版已经不建议使用后缀匹配模式，但是根据惯例，SpringMVC项目很喜欢使用诸如`xxx.do`或者`xxx.htm`的后端接口，如果Controller路径上不包含这些后缀会导致*404*问题。

3. 删除`<mvc:annotation-driven />`

4. 静态资源映射配置。

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
