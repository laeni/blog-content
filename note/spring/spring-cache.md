---
title: Spring Cache 入门
tags: 'Spring Cache, Spring'
author: Laeni
date: '2021-08-27'
updated: '2021-08-28
---

先从缓存中读取数据，如果没有再从慢速设备上读取实际数据（数据也会存入缓存）。缓存什么经常读取且不经常修改的数据、缓存昂贵（CPU/IO）的且对于相同的请求有相同的计算结果的数据。

本文使用的版本：`spring-boot-starter-cache:2.1.6.RELEASE` & `spring-boot-starter-data-redis:2.1.6.RELEASE`

## 基本概念

### TTL（Time To Live ）

存活期，即从缓存中创建时间点开始直到它到期的一个时间段（不管在这个时间段内有没有访问都将过期）。

### TTI（Time To Idle）

空闲期，即一个数据多久没被访问将从缓存中移除的时间。

### Eviction policy（移除策略）

移除策略，即如果缓存满了，从缓存中移除数据的策略。
 常见：

- FIFO（First In First Out）：先进先出算法，即先放入缓存的先被移除；
- LRU（Least Recently Used）：最久未使用算法，使用时间距离现在最久的那个被移除；
- LFU（Least Frequently Used）：最近最少使用算法，一定时间段内使用次数（频率）最少的那个被移除；
- 带过期时间：
  - volatile-lru：从已设置过期时间的数据集中淘汰**最近最少使用**的数据
  - volatile-ttl：从已设置过期时间的数据集中淘汰**将要过期**的数据
  - volatile-random：从已设置过期时间的数据集中淘汰**随机**的数据
  - allkeys-lru
  - allkeys-random

何时移除：

- 访问Key A时，移除举例最远的过期Key
- 定时任务
- CPU空闲时

## 常用API注解

自Spring 3.1起，提供了类似于@Transactional注解事务的注解Cache支持，且提供了Cache抽象，在此之前一般通过AOP实现。

使用Spring Cache的好处：

- 提供基本的Cache抽象，方便切换各种底层Cache；
- 通过注解Cache可以实现类似于事务一样，缓存逻辑透明的应用到我们的业务代码上，且只需要更少的代码就可以完成；
- 提供事务回滚时也自动回滚缓存；
- 支持比较复杂的缓存逻辑；

对于Spring Cache抽象，主要从以下几个方面学习：

- Cache API及默认提供的实现
- Cache注解
- 实现复杂的Cache逻辑

### Cache - API

```java
package org.springframework.cache;
public interface Cache {
    /**
     * 缓存的名字（用于缓存做数据隔离和分类，如用户缓存、token缓存）.
     */
    String getName();
    /**
     * 返回底层缓存提供程序，如 RedisCacheWriter.
     * 暂时没发现实际用处，可能只是提供获取原生缓存的bean，以便需要扩展一些缓存操作或统计之类的东西.
     */
    Object getNativeCache();
    /**
     * 返回 key 对应的 “值的包装器”，然后调用其get方法获取值.
     * 由于需要兼容存储空值的情况，将返回值包装了一层.
     */
    ValueWrapper get(Object key);
    /**
     * 直接返回 key 对应的值.
     * 如果允许缓存null值时，该方法无法区分返回的null表示“值就是null”还是“没有值”，该情况下需要使用“get(Object key)”变体.
     */
    <T> T get(Object key, Class<T> type);
    /**
     * 直接返回 key 对应的值,如果该值不存在则使用 valueLoader 获取新值(使用 valueLoader.call() 来调用 @Cacheable 注解的方法得到值)加入缓存并返回该值.
     * 当@Cacheable注解的sync属性配置为true时使用此方法。因此方法内需要保证回源到数据库的同步性。避免在缓存失效时大量请求回源到数据库
     */
    <T> T get(Object key, Callable<T> valueLoader);
    /**
     * 将 @Cacheable 注解方法返回的数据放入缓存中
     */
    void put(Object key, Object value);
    /**
     * 当缓存中不存在key时才放入缓存。
     * 相当于 get 和 put 的结合，如果不存在 key 对应的数据则加入该数据并返回，如果 key 存在则 key 对应的原始数据
     */
    ValueWrapper putIfAbsent(Object key, @Nullable Object value);
    /**
     * 从缓存中移除key对应的缓存.
     */
    void evict(Object key);
    /**
     * 删除缓存中的所有数据。
     * 需要注意的是，具体实现中只 name 对应缓存的全部数据，不要影响应用内的其他缓存.
     */
    void clear();

    /**
     * 由于缓存的值本身可能为null，所以需要使用包装器进行区分，如果不存在对应值则返回null，      * 但是如果存在值为null的缓存则应该返回一个包装器对象，但该包装器包装的值是null
     */
    interface ValueWrapper {
        Object get(); // 得到真实的value
    }
}
```

常用实现：

- AbstractValueAdaptingCache
  - CaffeineCache - 基于 Caffeine 实现。Caffeine是使用Java8对Guava缓存的重写版本，在Spring Boot 2.0中将取代Guava
  - ConcurrentMapCache - 基于核心 JDK java.util.concurrent 包的简单实现。一般用于测试或简单的缓存场景，通常与 SimpleCacheManager 结合使用或通过ConcurrentMapCacheManager 动态使用
  - JCacheCache - 基于 javax.cache.Cache 实例之上的实现（目前默认不存在 javax.cache 包）
  - RedisCache - 基于 Redis 实现，
- EhCacheCache - 基于 Ehcache 实现（类似于Redis）
- NoOpCache - 无缓存（适合禁用缓存或没有缓存的时候）
- TransactionAwareCacheDecorator - 缓存装饰器将其put 、 evict和clear操作与 Spring 管理的事务同步（通过 Spring 的TransactionSynchronizationManager ，仅在成功事务的提交后阶段执行实际的缓存 put/evict/clear 操作。如果没有事务处于活动状态，则put , evict和clear操作将像往常一样立即执行。不支持例如 putIfAbsent 的操作（会立即执行）。

> 除了这些默认的Cache之外，我们可以写自己的Cache实现。
>
> 而且即使不用之后的Spring Cache注解，我们也尽量使用Spring Cache API进行Cache的操作，如果要替换底层Cache也是非常方便的，比如单元测试时可能不需要缓存等。

### CacheManager - API

一般我们在应用中会同时使用多个`Cache`，因此Spring还提供了`CacheManager`，用于缓存的管理：

```java
package org.springframework.cache;
public interface CacheManager {
    /**
     * 根据Cache名字获取Cache.
     * <p>
     * 一般情况下，名字不需要提前定义好，而是执行该方法的时候会进行检查，
     * 如果不存在就创建一个该名字对应的 Cache 即可，这样在使用的时候只需要专注于业务，使用不同的名字进行缓存的隔离即可。
     * 因此在设计缓存Key的时候要把缓存名字考虑进去，否则无法起到隔离效果。
     * <p>
     * 具体实现中 name 一般作为 key 的一部分,用于和其他名字的缓存做隔离，而底层不需要创建很多真实的缓存。
     */
    Cache getCache(String name);
    /**
     * 得到所有Cache的名字.
     */
    Collection<String> getCacheNames();
}
```

常用实现：

- AbstractCacheManager - 通用 CacheManager 的抽象基类。
  - SimpleCacheManager - 简单的缓存管理器仅仅简单存储经常存在的缓存集合。一般用于测试或简单的缓存声明。
  - AbstractTransactionSupportingCacheManager - 支持 Spring 事务的 CacheManager 实现的基类。默认不开启。它的子类都支持事物，如果不是其子类的要支持事物，可以使用**TransactionAwareCacheManagerProxy**代理来支持。
    - JCacheCacheManager
    - EhCacheCacheManager
    - RedisCacheManager
- CompositeCacheManager - 用于组合CacheManager，即可以从多个CacheManager中轮询得到相应的Cache
- CaffeineCacheManager
- NoOpCacheManager - 无缓存。常用于禁用缓存或在没有实际缓存后备支持的情况下声明支持缓存。
- TransactionAwareCacheManagerProxy - 用于将不支持事务的缓存包装为支持事务的缓存。
- ConcurrentMapCacheManager

> Cache还支持Spring事务，即如果事务回滚了，Cache的数据也会移除掉。

### CacheManagerCustomizers - API

直译为**缓存管理自定义**，常用于修改`CacheManager`的配置，即如果需要系统默认缓存的配置时可以使用。但是**一般用不到**。

假如需要修改`RedisCacheManager`，那么就可以定义一个`CacheManagerCustomizer Bean`，在其中定义修改逻辑。这样当系统创建好`RedisCacheManager`实例时，会把该实例依次传给`CacheManagerCustomizer`列表进行处理。至于为什么一般用不到，原因可以参考文末的注意事项。

### KeyGenerator - API

如果在Cache注解上没有指定key的话@CachePut(value = "user")，会使用KeyGenerator进行生成一个key

```java
public interface KeyGenerator {  
    Object generate(Object target, Method method, Object... params);  
}  

@Override  
public Object generate(Object target, Method method, Object... params) {  
    if (params.length == 0) {  
        return SimpleKey.EMPTY;  
    }  
    if (params.length == 1 && params[0] != null) {  
        return params[0];  
    }  
    return new SimpleKey(params);  
```

我们也可以自定义自己的key生成器，然后通过xml风格的<cache:annotation-driven key-generator=""/>或注解风格的CachingConfigurer中指定keyGenerator。

### Cache注解

#### @EnableCaching - 启用Cache注解

```java
/**
 * 继承 CachingConfigurerSupport 进行配置，并按需重写父类方法。
 */
@Configuration
@EnableCaching // 使用@EnableCaching启用Cache注解支持
public class CacheConfiguration extends CachingConfigurerSupport {  
    /**
     * 一般提供一个简单key生成器即可,如果不提供,则默认没有明确指定key的情况下,会认为忽略所有参数生成key,
     * 即每次调用方法生成的key都是同一个（官方说的，实际效果没测试）.
     * <p>
     * 如果需要复杂的话key,一般是使用{@code spEl}表达式定义key,
     * </br>
     * 此外还可以定义多个{@linkplain KeyGenerator}Bean,并指定不同的名字,在注解中明确指定{@linkplain KeyGenerator}来生成Key.
     */
    @Bean @Override
    public KeyGenerator keyGenerator() {
        return new SimpleKeyGenerator();
    }
}  
```

#### @Cacheable - 查询/添加

应用到读取数据的方法上，即可缓存的方法，如查找方法：先从缓存中读取，如果没有再调用方法获取数据，然后把数据添加到缓存中

```java
/**
 * 主要应用到查询数据的方法上（或类中的所有方法），表示该方法的返回结果可以被缓存。
 * 默认值情况下使用方法参数来计算 Key，但可以通过 Key 属性提供 SpEL 表达式，或者自定义 KeyGenerator 实现来自定义计算 Key。
 * 如果在缓存中找不到 Key 对应的值，则调用目标方法并将返回值存储在 Key 关联的缓存中。
 * 请注意，Java8 的Optional返回类型是自动处理的，如果存在，其内容将存储在缓存中。
 * 此注解可用作元注释创建自定义注解（参见后文“自定义缓存注解”）
 */
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {
	/**
	 * Alias for {@link #cacheNames}.
	 */
	@AliasFor("cacheNames")
	String[] value() default {};

	/**
	 * 所使用的缓存名称。
	 * 一般用于不同类型的数据隔离，且一般不需要预先定义需要使用的名称，在使用时会自动根据名称动态创建。
	 */
	@AliasFor("value")
	String[] cacheNames() default {};

	/**
	 * 用于动态计算 Key 的(SpEL) 表达式。
	 * <p>默认为 {@code ""}, 意味着所有方法参数都被视为一个键，除非已配置自定义{@link #keyGenerator} 。
	 * <p>SpEL 表达式针对提供以下元数据的专用上下文进行评估:
	 * <ul>
	 * <li>{@code #root.method}, {@code #root.target}, 和 {@code #root.caches} 用于引用 {@link java.lang.reflect.Method method}、目标对象和使用的缓存.</li>
	 * <li>方法名称 ({@code #root.methodName}) 目标类 ({@code #root.targetClass}) 的表达式也可用.
	 * <li>方法参数可以通过索引访问。 例如，可以通过 {@code #root.args[1]}、 {@code #p1} 或 {@code #a1} 访问第二个参数。
     * 也可以按名称访问参数.</li>
	 * </ul>
	 */
	String key() default "";

	/**
	 * 要使用的自定义 KeyGenerator 的 bean name。
	 * <p>与 {@link #key} 属性互斥。
	 */
	String keyGenerator() default "";

	/**
	 * 自定义 {@link CacheManager} 的 bean name，可用于创建默认 {@link CacheResolver}.
	 * <p>与 {@link #cacheResolver} 属性互斥。
	 */
	String cacheManager() default "";

	/**
	 * 要使用的自定义 {@link CacheResolver} bean name。
	 */
	String cacheResolver() default "";

	/**
	 * 是有使用缓存。只有满足条件时才使用缓存数据，条件可以通过 SpEL 表达式书写条件。
	 * <p>在方法执行前评估条件，只有评估通过了才查询缓存。
	 * <p>默认情况下总是使用缓存.
	 * <p> SpEL 的使用可以参考 key 属性。
	 */
	String condition() default "";

	/**
	 * 否决缓存(是否不更新缓存)。在缓存前需要先评估该条件，满足条件时将跳过缓存，支持SpEL表达式。
	 * <p>在方法执行后评估条件，只有评估 条件为 false 才更新缓存。
	 * <p>由于在方法被调用后才执行，因此可以通过 SpEL 表达式引用 result 。
	 * <p>默认情况下永远不会否决缓存。
	 * #result 用于引用方法调用的结果。 对于支持的包装器，例如Optional ， #result指的是实际对象，而不是包装器，
	 * 实际上，如果将 #result 作为实参传递时，形参可以是包装器，也可以是实际对象。
	 * 其他 SpEL 的使用方式和 condition 及 key 属性的用法相同。
	 */
	String unless() default "";

	/**
	 * 是否同步调用被注释的方法。
	 * 如果多个线程试图加载同一个键的值，则同步底层方法的调用。
	 * <p>
	 * 一般实现中，如果为false，调用的是Cache.get(key) 或 Cache.get(Object key, Class<T> type) 方法；
	 * 如果为true，调用的是Cache.get(key, Callable)方法
	 * <p>
     * 同步会导致一些限制：
	 * <ol>
	 * <li>不支持 {@link #unless()}</li>
	 * <li>只能指定一个缓存</li>
	 * <li>不能组合其他缓存相关的操作</li>
	 * </ol>
	 * 这实际上是一个提示，您使用的实际缓存提供程序可能不以同步方式支持它。
     * 需要查看实际使用的缓存文档以获取有关实际语义的更多详细信息.
	 */
	boolean sync() default false;
}

// 使用示例
@Cacheable(value = "app", key = "'datasouce:app:' + #appId", sync = true)
public Optional<App> findByAppId(@NotNull String appId) {
    return appRepository.findByAppId(appId);
}
// @Cacheable将在执行方法之前( #result还拿不到返回值)判断condition,如果返回true，则查缓存；
@Cacheable(value = "user", key = "#id", condition = "#id lt 10")  
public User conditionFindById(final Long id)  
```

#### @CachePut - 添加/修改

一般用在写数据的方法上，如新增/修改方法，调用方法时会自动把相应的数据放入缓存

```java
public @interface CachePut {  
    String[] value();              //缓存的名字，可以把数据写到多个缓存  
    String key() default "";       //缓存key，如果不指定将使用默认的KeyGenerator生成，后边介绍  
    String condition() default ""; //满足缓存条件的数据才会放入缓存，condition在调用方法之前和之后都会判断  
    String unless() default "";    //用于否决缓存更新的，不像condition，该表达只在方法执行之后判断，此时可以拿到返回值result进行判断了  
}

public @interface CachePut {
	/** Alias for {@link #cacheNames}. */
	@AliasFor("cacheNames")
	String[] value() default {};

	/** 用法请参考 @Cacheable 注解 */
	@AliasFor("value")
	String[] cacheNames() default {};

	/** 
	 * 用法请参考 @Cacheable 注解.
	 * 在这里可以使用 #resul 获取结果。
     */
	String key() default "";

	/** 用法请参考 @Cacheable 注解 */
	String keyGenerator() default "";

	/** 用法请参考 @Cacheable 注解 */
	String cacheManager() default "";

	/** 用法请参考 @Cacheable 注解 */
	String cacheResolver() default "";

	/** 
	 * 用法请参考 @Cacheable 注解.
	 * 只有满足条件的数据才缓存。
     */
	String condition() default "";

	/** 
	 * 用法请参考 @Cacheable 注解.
	 * 否决缓存(是否不更新缓存)，如果满足条件则不缓存数据。
	 * 在这里可以使用 #resul 获取结果。
     */
	String unless() default "";
}

// 使用示例
@CachePut(value = "user", key = "#user.id")
public User save(User user) {
    users.add(user);
    return user;
}
// 如下@CachePut将在执行完方法后（#result就能拿到返回值了）判断condition
// 如果返回true，则放入缓存；
@CachePut(value = "user", key = "#id", condition = "#result.username ne 'zhang'")
public User conditionSave(final User user)
//根据运行流程，如下@CachePut将在执行完方法后（#result就能拿到返回值了）判断unless
//如果返回false，则放入缓存；（即跟condition相反）
@CachePut(value = "user", key = "#user.id", unless = "#result.username eq 'zhang'")
public User conditionSave2(final User user)
```

#### @CacheEvict - 删除

常用于删除数据时顺便将对应的缓存删除。但是不局限于删除方法，因为某些时候查询方法也可能会用到（比如查询到缓存后发现缓存数据已经不在适用，因此需要将缓存删除）；再比如刷新等场景。

```java
/**
 * 主要应用到查询数据的方法上（或类中的所有方法），表示该方法的返回结果可以被缓存。
 * 默认值情况下使用方法参数来计算 Key，但可以通过 Key 属性提供 SpEL 表达式，或者自定义 KeyGenerator 实现来自定义计算 Key。
 * 如果在缓存中找不到 Key 对应的值，则调用目标方法并将返回值存储在 Key 关联的缓存中。
 * 请注意，Java8 的Optional返回类型是自动处理的，如果存在，其内容将存储在缓存中。
 * 此注解可用作元注释创建自定义注解（参见后文“自定义缓存注解”）
 */
public @interface CacheEvict {
	/** Alias for {@link #cacheNames}. */
	@AliasFor("cacheNames")
	String[] value() default {};

	/** 用法请参考 @Cacheable 注解 */
	@AliasFor("value")
	String[] cacheNames() default {};

	/** 用法请参考 @Cacheable 注解 */
	String key() default "";

	/** 用法请参考 @Cacheable 注解 */
	String keyGenerator() default "";

	/** 用法请参考 @Cacheable 注解 */
	String cacheManager() default "";

	/** 用法请参考 @Cacheable 注解 */
	String cacheResolver() default "";

	/** 
	 * 用法请参考 @Cacheable 注解.
	 * 表示只有满足条件时才删除缓存，默认情况下总是删除。
     */
	String condition() default "";

	/**
	 * 是否删除缓存中的所有缓存数据。
	 * <p>默认情况下，仅删除关联键下的值。
	 * <p>请注意，不允许将此参数设置为 true 并指定key 。即为 ture 时与 Key 互斥。
	 */
	boolean allEntries() default false;

	/**
	 * 是否应该在调用方法之前删除缓存。
	 * <p>如果将此属性设置为true ，无论方法结果如何（即，是否抛出异常）都会导致缓存被删除。
	 * <p>默认为false ，意味着被注释的方法成功调用后才删除（即，仅当调用没有抛出异常时才删除缓存）。
	 */
	boolean beforeInvocation() default false;
}

// 使用示例
@CacheEvict(value = "user", key = "#user.id") //移除指定key的数据
public User delete(User user) {
    users.remove(user);
    return user;
}
@CacheEvict(value = "user", allEntries = true) //移除所有数据
public void deleteAll() {
    users.clear();
}
// @CacheEvict， beforeInvocation=false表示在方法执行之后调用（#result能拿到返回值了）
// 且判断condition，如果返回true，则移除缓存
@CacheEvict(value = "user", key = "#user.id", beforeInvocation = false, condition = "#result.username ne 'zhang'")
public User conditionDelete(final User user)
```

#### @Caching - 组合使用

有时候我们可能组合多个Cache注解使用:

- 比如用户新增成功后，我们要添加id-->user；username--->user；email--->user的缓存；此时就需要@Caching组合多个注解标签了。
- 如用户新增成功后，添加id-->user；username--->user；email--->user到缓存；

```java
/**
 * 多个缓存注释（不同或相同类型）的组注释。
 * 此注解可用作元注释创建自定义注解（参见后文“自定义缓存注解”）
 */
public @interface Caching {

	Cacheable[] cacheable() default {};

	CachePut[] put() default {};

	CacheEvict[] evict() default {};
}

// 使用示例
@Caching(
        put = {
                @CachePut(value = "user", key = "#user.id"),
                @CachePut(value = "user", key = "#user.username"),
                @CachePut(value = "user", key = "#user.email")
        }
)
public User save(User user)
```

当组合使用时，调用流程如下：（ **未验证** ）

```ruby
1、首先执行@CacheEvict（如果beforeInvocation=true且condition 通过），如果allEntries=true，则清空所有  
2、接着收集@Cacheable（如果condition 通过，且key对应的数据不在缓存），放入cachePutRequests（也就是说如果cachePutRequests为空，则数据在缓存中）  
3、如果cachePutRequests为空且没有@CachePut操作，那么将查找@Cacheable的缓存，否则result=缓存数据（也就是说只要当没有cache put请求时才会查找缓存）  
4、如果没有找到缓存，那么调用实际的API，把结果放入result  
5、如果有@CachePut操作(如果condition 通过)，那么放入cachePutRequests  
6、执行cachePutRequests，将数据写入缓存（unless为空或者unless解析结果为false）；  
7、执行@CacheEvict（如果beforeInvocation=false 且 condition 通过），如果allEntries=true，则清空所有  
```

#### @CacheConfig - 设置默认

用于在类级别上设置默认值。

```java
public @interface CacheConfig {
	/** 用法同其他注解 */
	String[] cacheNames() default {};
	/** 用法同其他注解 */
	String keyGenerator() default "";
	/** 用法同其他注解 */
	String cacheManager() default "";
	/** 用法同其他注解 */
	String cacheResolver() default "";
}
```

#### 自定义缓存注解

比如之前的那个@Caching组合，会让方法上的注解显得整个代码比较乱，此时可以使用自定义注解把这些注解组合到一个注解中，如：

```java
@Caching(  
        put = {  
                @CachePut(value = "user", key = "#user.id"),  
                @CachePut(value = "user", key = "#user.username"),  
                @CachePut(value = "user", key = "#user.email")  
        }  
)  
@Target({ElementType.METHOD, ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Inherited  
public @interface UserSaveCache {  
}  
```

这样我们在方法上使用如下代码即可，整个代码显得比较干净。

```java
@UserSaveCache  
public User save(User user)


@Caching(  
        put = {  
                @CachePut(value = "user", key = "#user.id"),  
                @CachePut(value = "user", key = "#user.username"),  
                @CachePut(value = "user", key = "#user.email")  
        }  
)  
public User save(User user)  

@Caching(  
        put = {  
                @CachePut(value = "user", key = "#user.id"),  
                @CachePut(value = "user", key = "#user.username"),  
                @CachePut(value = "user", key = "#user.email")  
        }  
)  
public User update(User user)   

@Caching(  
        evict = {  
                @CacheEvict(value = "user", key = "#user.id"),  
                @CacheEvict(value = "user", key = "#user.username"),  
                @CacheEvict(value = "user", key = "#user.email")  
        }  
)  
public User delete(User user)  

@CacheEvict(value = "user", allEntries = true)  
 public void deleteAll()  

@Caching(  
        cacheable = {  
                @Cacheable(value = "user", key = "#id")  
        }  
)  
public User findById(final Long id)  

@Caching(  
         cacheable = {  
                 @Cacheable(value = "user", key = "#username")  
         }  
 )  
 public User findByUsername(final String username)  

@Caching(  
          cacheable = {  
                  @Cacheable(value = "user", key = "#email")  
          }  
  )  
  public User findByEmail(final String email) 
```

#### 提供的SpEL上下文数据

Spring Cache提供了一些供我们使用的SpEL上下文数据，下表直接摘自Spring官方文档：

![img](https:////upload-images.jianshu.io/upload_images/5682416-07835d0333714e6a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 实现复杂的Cache逻辑

往缓存放数据/移除数据是有条件的，而且条件可能很复杂，考虑使用SpEL表达式

```java
@CacheEvict(value = "user", key = "#user.id", condition = "#root.target.canCache() and #root.caches[0].get(#user.id).get().username ne #user.username", beforeInvocation = true)  
public void conditionUpdate(User user) 
```

> 或更复杂的**直接调用目标对象的方法**进行操作
>  这个比较厉害，可以直接调用目标对象的方法

```java
@Caching(  
        evict = {  
                @CacheEvict(value = "user", key = "#user.id", condition = "#root.target.canEvict(#root.caches[0], #user.id, #user.username)", beforeInvocation = true)  
        }  
)  
public void conditionUpdate(User user){...}

public boolean canEvict(Cache userCache, Long id, String username) {  
    User cacheUser = userCache.get(id, User.class);  
    if (cacheUser == null) {  
        return false;  
    }  
    return !cacheUser.getUsername().equals(username);  
}  
```

如上方式唯一不太好的就是缓存条件判断方法也需要暴露出去；而且缓存代码和业务代码混合在一起，不优雅；因此把canEvict方法移到一个Helper静态类中就可以解决这个问题了：

```java
@CacheEvict(value = "user", key = "#user.id", condition = "T(com.sishuok.spring.service.UserCacheHelper).canEvict(#root.caches[0], #user.id, #user.username)", beforeInvocation = true)  
public void conditionUpdate(User user)
```

> 对于：id--->user；username---->user；email--->user；更好的方式可能是：id--->user；username--->id；email--->id；保证user只存一份；如：

```java
@CachePut(value="cacheName", key="#user.username", cacheValue="#user.username")  
public void save(User user)   

@Cacheable(value="cacheName", key="#user.username", cacheValue="#caches[0].get(#caches[0].get(#username).get())")  
public User findByUsername(String username) 
```

## Redis+Caffeine 实现二级缓存

![img](F:\Objects\cn.laeni\blog-content\note\spring\spring-cache.assets\2018227134516693.png)



## 使用注意事项

1. 在`JPA Repository`接口使用`@Cacheable`等注解时，不能使用`key`属性自定义`缓存Key`，因为`SpEL`表达式在这种情况下取不到对应的参数值，但是可以使用`keyGenerator`。而实际上一般都是在`repository`层上再封装一层`service`层，在`service`层上使用缓存注解。
2. 如果自定义`CacheManager Bean`将导致`XxxCacheConfiguration`失效从而导致`spring.cache.xxx`等特定于缓存的配置失效，所以如果自定义`CacheManager`时需要自行注入`spring.cache.xxx`相关配置。
3. 有时候不得不自定义`CacheManager`，比如实现二级缓存、采用JSON序列化（采用JSON序列化时需要为需要缓存的数据定制专用的`CacheManager`来设置专用的序列化器，否则从字符串转为`Object`时一般会转为`Map`，而不是实际需要的类型，导致后续进行强转时出问题），这时候可以参考官方`org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration`配置，但是无需管`CacheManagerCustomizers`，因为`CacheManagerCustomizers`的作用是自定义配置`CacheManager`，但是由于我们本身就是在自定义`CacheManager`，所以无需再通过`CacheManagerCustomizers`进行配置。

## 参考

- [【简书】Spring Cache](https://www.jianshu.com/p/33c019de9115)
- [【博客园】Caffeine用法](https://www.cnblogs.com/fnlingnzb-learner/p/11025565.html)

