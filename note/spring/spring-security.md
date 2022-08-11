---
title: Spring Security
tags: 'Spring Security, Spring'
author: Laeni
date: '2022-03-27'
updated: '2022-03-28
---

## 核心概念

| 类名                   | 概念                                                         |
| ---------------------- | ------------------------------------------------------------ |
| AuthenticationManager  | 用户认证的管理类，所有的认证请求（比如login）都会通过提交一个token给AuthenticationManager的authenticate()方法来实现。当然事情肯定不是它来做，具体校验动作会由AuthenticationManager将请求转发给具体的实现类来做。根据实现反馈的结果再调用具体的Handler来给用户以反馈。 |
| AuthenticationProvider | 认证的具体实现类，一个provider是一种认证方式的实现，比如提交的用户名密码我是通过和DB中查出的user记录做比对实现的，那就有一个DaoProvider；如果我是通过CAS请求单点登录系统实现，那就有一个CASProvider。按照Spring一贯的作风，主流的认证方式它都已经提供了默认实现，比如DAO、LDAP、CAS、OAuth2等。前面讲了AuthenticationManager只是一个代理接口，真正的认证就是由AuthenticationProvider来做的。一个AuthenticationManager可以包含多个Provider，每个provider通过实现一个support方法来表示自己支持那种Token的认证。AuthenticationManager默认的实现类是ProviderManager。 |
| UserDetailService      | 用户认证通过Provider来做，所以Provider需要拿到系统已经保存的认证信息，获取用户信息的接口spring-security抽象成UserDetailService。 |
| AuthenticationToken    | 用户认证通过Provider来做，所以Provider需要拿到系统已经保存的认证信息，获取用户信息的接口spring-security抽象成UserDetailService。 |
| AuthenticationToken    | 所有提交给AuthenticationManager的认证请求都会被封装成一个Token的实现，比如最容易理解的UsernamePasswordAuthenticationToken。 |
| SecurityContext        | 当用户通过认证之后，就会为这个用户生成一个唯一的SecurityContext，里面包含用户的认证信息Authentication。通过SecurityContext我们可以获取到用户的标识Principle和授权信息GrantedAuthrity。在系统的任何地方只要通过SecurityHolder.getSecruityContext()就可以获取到SecurityContext。 |

## 核心拦截器

所有的过滤器都会实现SpringSecurityFilter安全过滤器。以下表格中的过滤器按照执行顺序排列。

| 拦截器                                  | 释义                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| HttpSessionContextIntegrationFilter     | 位于过滤器顶端，第一个起作用的过滤器。用途一，在执行其他过滤器之前，率先判断用户的session中是否已经存在一个SecurityContext了。如果存在，就把SecurityContext拿出来，放到SecurityContextHolder中，供Spring Security的其他部分使用。如果不存在，就创建一个SecurityContext出来，还是放到SecurityContextHolder中，供Spring Security的其他部分使用。用途二，在所有过滤器执行完毕后，清空SecurityContextHolder，因为SecurityContextHolder是基于ThreadLocal的，如果在操作完成后清空ThreadLocal，会受到服务器的线程池机制的影响。 |
| LogoutFilter                            | 只处理注销请求，默认为/j_spring_security_logout。用途是在用户发送注销请求时，销毁用户session，清空SecurityContextHolder，然后重定向到注销成功页面。可以与rememberMe之类的机制结合，在注销的同时清空用户cookie。 |
| AuthenticationProcessingFilter          | 处理form登陆的过滤器，与form登陆有关的所有操作都是在此进行的。默认情况下只处理/j_spring_security_check请求，这个请求应该是用户使用form登陆后的提交地址此过滤器执行的基本操作时，通过用户名和密码判断用户是否有效，如果登录成功就跳转到成功页面（可能是登陆之前访问的受保护页面，也可能是默认的成功页面），如果登录失败，就跳转到失败页面。 |
| DefaultLoginPageGeneratingFilter        | 此过滤器用来生成一个默认的登录页面，默认的访问地址为/spring_security_login，这个默认的登录页面虽然支持用户输入用户名，密码，也支持rememberMe功能，但是因为太难看了，只能是在演示时做个样子，不可能直接用在实际项目中。 |
| BasicProcessingFilter                   | 此过滤器用于进行basic验证，功能与AuthenticationProcessingFilter类似，只是验证的方式不同。 |
| SecurityContextHolderAwareRequestFilter | 此过滤器用来包装客户的请求。目的是在原始请求的基础上，为后续程序提供一些额外的数据。比如getRemoteUser()时直接返回当前登陆的用户名之类的。 |
| RememberMeProcessingFilter              | 此过滤器实现RememberMe功能，当用户cookie中存在rememberMe的标记，此过滤器会根据标记自动实现用户登陆，并创建SecurityContext，授予对应的权限。 |
| AnonymousProcessingFilter               | 为了保证操作统一性，当用户没有登陆时，默认为用户分配匿名用户的权限。 |
| ExceptionTranslationFilter              | 此过滤器的作用是处理`FilterSecurityInterceptor`中抛出的异常，然后将请求重定向到对应页面，或返回对应的响应错误代码 |
| SessionFixationProtectionFilter         | 防御会话伪造攻击。                                           |
| FilterSecurityInterceptor               | 用户的权限控制都包含在这个过滤器中。<br />功能一：如果用户尚未登陆，则抛出AuthenticationCredentialsNotFoundException“尚未认证异常”。<br />功能二：如果用户已登录，但是没有访问当前资源的权限，则抛出AccessDeniedException“拒绝访问异常”。<br />功能三：如果用户已登录，也具有访问当前资源的权限，则放行。我们可以通过配置方式来自定义拦截规则 |

## 执行流程

```
                     (AuthenticationToken)
AuthenticationFilter ---------------------> AuthenticationManager --> AuthenticationProvider
```

## WebSecurityConfigurerAdapter configure()方法作用

配置SpringSecurity时一般集成`WebSecurityConfigurerAdapter `抽象类进行配置。以下方法按照执行顺序排列。

**void init(final WebSecurity web)**

**final HttpSecurity getHttp()**

**AuthenticationManager authenticationManager()**

**void configure(AuthenticationManagerBuilder auth)**

**UserDetailsService userDetailsService()**

**void configure(HttpSecurity http)**

**void configure(WebSecurity web)**

WEB安全配置。作用如下：

1. 一般常用于忽略某些请求，且这些请求一般是静态资源（动态资源一般通过`void configure(HttpSecurity http)`配置为“所有用户可用”）。

---

**UserDetailsService userDetailsServiceBean()**

仅用于将`UserDetailsService`实例公开为Bean。

**AuthenticationManager authenticationManagerBean()**

仅用于将`AuthenticationManager `实例公开为Bean。

## 集成JWT示例

### 登陆阶段流程

**1. 登陆阶段流程图。**
中间省略了Spring Security 的某些调用。仅用来描绘自己代码的逻辑。

![登陆阶段流程图](spring-security.assets/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2ODgyNzkz,size_16,color_FFFFFF,t_70.png)

## AuthenticationException 子类

| 异常                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| UsernameNotFoundException                                    | 无法通过用户名找到用户                                       |
| BadCredentialsException                                      | 凭据无效。要引发此异常，这意味着该帐户既没有被锁定也没有被禁用。 |
| AccountStatusException<br />\|    AccountExpiredException<br />\|    CredentialsExpiredException<br />\|    DisabledException<br />\|    LockedException | 用于由特定用户帐户状态 (锁定，禁用等) 引起的身份验证异常的基类。不断言凭据是否有效。<br />\|    帐户已过期。<br />\|    帐户的凭据已过期。<br />\|    帐户被禁用。<br />\|    帐户被锁定。 |
| AuthenticationCredentialsNotFoundException                   | 如果身份验证请求被拒绝，则抛出，因为`Authentication`对象中没有`SecurityContext`。 |
| InsufficientAuthenticationException                          | 如果由于凭据不够受信任而拒绝了身份验证请求，则抛出。         |
| AuthenticationServiceException<br />\|    InternalAuthenticationServiceException | 如果由于系统问题而无法处理身份验证请求，则抛出。例如，如果后端身份验证存储库不可用，这可能会引发。<br />\|    如果由于内部发生的系统问题而无法处理身份验证请求，则抛出。它不同于AuthenticationServiceException因为如果外部系统有内部错误或故障，它不会被抛出。这样可以确保我们可以处理与其他系统的错误明显不同的控制范围内的错误。这种区别的好处是，不受信任的外部系统不应该能够填满日志并导致过多的IO。但是，内部系统应该报告错误。<br/>例如，如果后端身份验证存储库不可用，这可能会引发。但是，如果使用OpenID提供程序验证OpenID响应时发生错误，则不会抛出该错误。 |
| ProviderNotFoundException                                    | 未找到提供者异常                                             |
| RememberMeAuthenticationException<br />\|    CookieTheftException<br />\|    InvalidCookieException | 记住我身份验证<br />\|    Cookie盗窃<br />\|    无效的Cookie |
| SessionAuthenticationException                               | 被一个会话认证策略指示身份验证对象对当前会话无效，通常是因为同一用户已超过了他们同时允许的会话数。 |
| NonceExpiredException                                        | 如果由于摘要随机码已过期而拒绝了身份验证请求，则抛出。       |



##　常见HTTP状态码

403 - Forbidden - Access Denied | 禁止 - 拒绝访问

## 常见问题

1. **通过配置文件配置的用户不生效、密码错误或者报错？**

   a. 先确认看有没有明确配置了`PasswordEncoder`Bean，如果没有配置，则Spring会采用默认的实现（`DelegatingPasswordEncoder`），且当配置的密码是明文是Spring会自动加上`{noop}`前缀，这样就能通过`DelegatingPasswordEncoder`进行校验了。但是一旦明确声明了`PasswordEncoder`，则不管在哪里配置，配置时必须使用密文，确保当前生效的`PasswordEncoder`能进行校验。

   b. 如果配置了`AuthenticationManager`、`AuthenticationProvider`和`UserDetailsService`等Bean时，配置文件配置的用户将不会生效，详情参见：`UserDetailsServiceAutoConfiguration`和`ReactiveUserDetailsServiceAutoConfiguration`等。

   c. 如果重写了`WebSecurityConfigurerAdapter#configure(AuthenticationManagerBuilder)`方法则
   
2. 用户不存在时打印堆栈

3. 通过`public void configure(HttpSecurity http)`忽略掉的请求还是会走过滤链路。

## 参考

[Spring Security权限控制 + JWT Token 认证](https://blog.csdn.net/qq_36882793/article/details/102839333)

