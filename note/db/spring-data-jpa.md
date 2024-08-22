---
title: Spring Data JPA
author: Laeni
tags: 'Java, Spring, SpringData, JPA, DataSource'
date: 2022-11-03
updated: 2024-08-22
---

# 多数据源配置

JPA 中多数据源配置的核心在于每个数据都有对应的`EntityManagerFactory`和`LocalContainerEntityManagerFactoryBean`两个Bean，然后再通过`@EnableJpaRepositories`注解指定每个数据源对应的`EntityManagerFactory`和`LocalContainerEntityManagerFactoryBean`即可。具体实现可以参考[官方示例项目 - spring-projects/spring-data-examples](https://github.com/spring-projects/spring-data-examples/tree/main/jpa/multiple-datasources)。

如果需要为每个数据源定制JPA或者Hibernate配置的话，可以参考`JpaBaseConfiguration`及其已有实现进行配置，比如针对某个数据源`DdlAuto`、根据不同的数据源指定不同的方言和 Ping SQL等。这时可能需要按需配置多个`HibernateProperties`，并根据它创建`JpaVendorAdapter`来构建`EntityManagerFactoryBuilder`（当然也可以直接构建`LocalContainerEntityManagerFactoryBean`并Set`JpaVendorAdapter`）。

## 示例

### 数据源和JPA配置都不相同

- `application.yml`

  ```yml
  spring:
    datasource:
      db1:
        driverClassName: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://***:3306/***?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&useServerPrepStmts=true&cachePrepStmts=true&autoReconnect=true&failOverReadOnly=false&autoReconnectForPools=true&noAccessToProcedureBodies=true&useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: ***
        minimum-idle: 1
        maximum-pool-size: 5
        auto-commit: true
        idle-timeout: 30000
        pool-name: HikariCP-DB1
        max-lifetime: 1800000
        connection-timeout: 30000
        connection-test-query: SELECT 1
      db2:
        driverClassName: oracle.jdbc.OracleDriver
        jdbc-url: jdbc:oracle:thin:@(DESCRIPTION =(ADDRESS_LIST =(ADDRESS = (PROTOCOL = TCP)(HOST = ***)(PORT = 1521)))(CONNECT_DATA =(SERVICE_NAME = ***)))
        username: FSPF_BASIC
        password: ***
        minimum-idle: 2
        maximum-pool-size: 5
        auto-commit: true
        idle-timeout: 30000
        pool-name: Hikari-DB2
        max-lifetime: 1800000
        connection-timeout: 30000
        connection-test-query: SELECT 1 FROM DUAL
    jpa:
      db1:
        database: MYSQL
        generate-ddl: false
        show-sql: true
        open-in-view: false
        database-platform: org.hibernate.dialect.MySQL57Dialect
        hibernate.ddl-auto: validate
      db2:
        database: ORACLE
        generate-ddl: false
        show-sql: true
        open-in-view: false
        database-platform: org.hibernate.dialect.Oracle8iDialect
        hibernate.ddl-auto: validate
        # 如果有需要可以配置 Hibernate 来指定默认 Schema
        properties.hibernate.default_schema: FSPF_POSP_SYN
  ```

- `DataSourceConfiguration.java` - 数据源配置

  由于需要复用`org.springframework.boot.autoconfigure.orm.jpa.JpaBaseConfiguration`这部分代码，而该类的构造参数需要`DataSource`、`JpaProperties`等实例，所以需要单独配置这些 Bean。

  ```java
  @Slf4j
  @Configuration
  public class DataSourceConfiguration {
      //#region DB1
      /**
       * 数据源前缀 - DB1.
       */
      static final String DB_PREFIX_1 = "db1";
  
      @Bean(value = DB_PREFIX_1 + "_dataSource", destroyMethod = "close")
      @ConfigurationProperties(prefix = "spring.datasource." + DB_PREFIX_1)
      public HikariDataSource dataSource1() {
          return new HikariDataSource();
      }
  
      @Bean(DB_PREFIX_1 + "_jpaProperties")
      @ConfigurationProperties(prefix = "spring.jpa." + DB_PREFIX_1)
      public JpaProperties jpaProperties1() {
          return new JpaProperties();
      }
  
      @Bean(DB_PREFIX_1 + "_hibernateProperties")
      @ConfigurationProperties(prefix = "spring.jpa." + DB_PREFIX_1 + ".hibernate")
      public HibernateProperties hibernateProperties1() {
          return new HibernateProperties();
      }
      //#endregion
  
      //#region DB2
      /**
       * 数据源前缀 - DB2.
       */
      static final String DB_PREFIX_2 = "db2";
  
      @Bean(value = DB_PREFIX_2 + "_dataSource", destroyMethod = "close")
      @ConfigurationProperties(prefix = "spring.datasource." + DB_PREFIX_2)
      public HikariDataSource dataSource2() {
          return new HikariDataSource();
      }
  
      @Bean(DB_PREFIX_2 + "_jpaProperties")
      @ConfigurationProperties(prefix = "spring.jpa." + DB_PREFIX_2)
      public JpaProperties jpaProperties2() {
          return new JpaProperties();
      }
  
      @Bean(DB_PREFIX_2 + "_hibernateProperties")
      @ConfigurationProperties(prefix = "spring.jpa." + DB_PREFIX_2 + ".hibernate")
      public HibernateProperties hibernateProperties2() {
          return new HibernateProperties();
      }
      //#endregion
  }
  ```

- `HibernateJpaConfigurationX.java` - HibernateJpa 配置

  此部分配置代码基本是来自`org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaConfiguration`，大部分都是通过重写的方式修改 Bean 的名称，否则会因为同名 Bean 的问题导致冲突。

  **`HibernateJpaConfiguration1.java`**

  ```java
  @Configuration
  @EnableJpaRepositories(
          entityManagerFactoryRef = DataSourceConfiguration.DB_PREFIX_1 + "_entityManagerFactory",
          transactionManagerRef = DataSourceConfiguration.DB_PREFIX_1 + "_jpaTransactionManager",
          // TODO Repository 所在包名
          basePackages = "com.xxx1.repository"
  )
  @EnableConfigurationProperties({HibernateProperties.class})
  public class HibernateJpaConfiguration1 extends JpaBaseConfiguration {
      private final DataSource dataSource;
      private final HibernateProperties hibernateProperties;
      private final List<HibernatePropertiesCustomizer> hibernatePropertiesCustomizers;
  
      HibernateJpaConfiguration1(@Qualifier(DataSourceConfiguration.DB_PREFIX_1 + "_dataSource") DataSource dataSource,
                                 @Qualifier(DataSourceConfiguration.DB_PREFIX_1 + "_jpaProperties") JpaProperties properties,
                                 @Qualifier(DataSourceConfiguration.DB_PREFIX_1 + "_hibernateProperties") HibernateProperties hibernateProperties,
                                 ConfigurableListableBeanFactory beanFactory,
                                 ObjectProvider<JtaTransactionManager> jtaTransactionManager,
                                 ObjectProvider<HibernatePropertiesCustomizer> hibernatePropertiesCustomizers) {
          super(dataSource, properties, jtaTransactionManager);
          this.dataSource = dataSource;
          this.hibernateProperties = hibernateProperties;
          this.hibernatePropertiesCustomizers = determineHibernatePropertiesCustomizers(beanFactory,
                  hibernatePropertiesCustomizers.orderedStream().collect(Collectors.toList()));
      }
  
      /**
       * 多数据源下,如果"spring.jpa"配置不相同时必须为每个数据源进行配置,
       * 否则所有数据源将使用同一份配置.
       */
      @Override
      @Bean(DataSourceConfiguration.DB_PREFIX_1 + "_jpaVendorAdapter")
      public JpaVendorAdapter jpaVendorAdapter() {
          // 这里虽然是直接 super 的,但依然是不一样的,内部使用了通过 super(_, properties, _) 构造方法传入的参数.
          return super.jpaVendorAdapter();
      }
  
      @Override
      @Bean(DataSourceConfiguration.DB_PREFIX_1 + "_entityManagerFactoryBuilder")
      public EntityManagerFactoryBuilder entityManagerFactoryBuilder(@Qualifier(DataSourceConfiguration.DB_PREFIX_1 + "_jpaVendorAdapter") JpaVendorAdapter jpaVendorAdapter,
                                                                     ObjectProvider<PersistenceUnitManager> persistenceUnitManager,
                                                                     ObjectProvider<EntityManagerFactoryBuilderCustomizer> customizers) {
          return super.entityManagerFactoryBuilder(jpaVendorAdapter, persistenceUnitManager, customizers);
      }
  
      @Override
      @Bean(DataSourceConfiguration.DB_PREFIX_1 + "_entityManagerFactory")
      public LocalContainerEntityManagerFactoryBean entityManagerFactory(@Qualifier(DataSourceConfiguration.DB_PREFIX_1 + "_entityManagerFactoryBuilder") EntityManagerFactoryBuilder factoryBuilder) {
          Map<String, Object> vendorProperties = getVendorProperties();
          customizeVendorProperties(vendorProperties);
  
          return factoryBuilder
                  .dataSource(this.dataSource)
                  // TODO 实体所在包名
                  .packages("com.xxx1.entity")
                  .properties(vendorProperties)
                  .jta(isJta())
                  .build();
      }
  
      @Bean(DataSourceConfiguration.DB_PREFIX_1 + "_jpaTransactionManager")
      public JpaTransactionManager jpaTransactionManager(@Qualifier(DataSourceConfiguration.DB_PREFIX_1 + "_entityManagerFactory") LocalContainerEntityManagerFactoryBean entityManagerFactory) {
          return new JpaTransactionManager(Objects.requireNonNull(entityManagerFactory.getObject()));
      }
  
      @Override
      protected AbstractJpaVendorAdapter createJpaVendorAdapter() {
          return new HibernateJpaVendorAdapter();
      }
  
      @Override
      protected Map<String, Object> getVendorProperties() {
          return new LinkedHashMap<>(this.hibernateProperties.determineHibernateProperties(getProperties().getProperties(),
                  new HibernateSettings().ddlAuto(() -> "none").hibernatePropertiesCustomizers(this.hibernatePropertiesCustomizers)));
      }
  
      private List<HibernatePropertiesCustomizer> determineHibernatePropertiesCustomizers(ConfigurableListableBeanFactory beanFactory, List<HibernatePropertiesCustomizer> hibernatePropertiesCustomizers) {
          List<HibernatePropertiesCustomizer> customizers = new ArrayList<>();
          customizers.add((properties) -> properties.put(AvailableSettings.BEAN_CONTAINER, new SpringBeanContainer(beanFactory)));
          customizers.addAll(hibernatePropertiesCustomizers);
          return customizers;
      }
  }
  ```

  **`HibernateJpaConfiguration2.java`**

  ```java
  @Configuration
  @EnableJpaRepositories(
          entityManagerFactoryRef = DataSourceConfiguration.DB_PREFIX_2 + "_entityManagerFactory",
          transactionManagerRef = DataSourceConfiguration.DB_PREFIX_2 + "_jpaTransactionManager",
          // TODO Repository 所在包名
          basePackages = "com.xxx2.repository"
  )
  @EnableConfigurationProperties({HibernateProperties.class})
  public class HibernateJpaConfiguration2 extends JpaBaseConfiguration {
      private final DataSource dataSource;
      private final HibernateProperties hibernateProperties;
      private final List<HibernatePropertiesCustomizer> hibernatePropertiesCustomizers;
  
      HibernateJpaConfiguration2(@Qualifier(DataSourceConfiguration.DB_PREFIX_2 + "_dataSource") DataSource dataSource,
                                 @Qualifier(DataSourceConfiguration.DB_PREFIX_2 + "_jpaProperties") JpaProperties properties,
                                 @Qualifier(DataSourceConfiguration.DB_PREFIX_2 + "_hibernateProperties") HibernateProperties hibernateProperties,
                                 ConfigurableListableBeanFactory beanFactory,
                                 ObjectProvider<JtaTransactionManager> jtaTransactionManager,
                                 ObjectProvider<HibernatePropertiesCustomizer> hibernatePropertiesCustomizers) {
          super(dataSource, properties, jtaTransactionManager);
          this.dataSource = dataSource;
          this.hibernateProperties = hibernateProperties;
          this.hibernatePropertiesCustomizers = determineHibernatePropertiesCustomizers(beanFactory,
                  hibernatePropertiesCustomizers.orderedStream().collect(Collectors.toList()));
      }
  
      /**
       * 多数据源下,如果"spring.jpa"配置不相同时必须为每个数据源进行配置,
       * 否则所有数据源将使用同一份配置.
       */
      @Override
      @Bean(DataSourceConfiguration.DB_PREFIX_2 + "_jpaVendorAdapter")
      public JpaVendorAdapter jpaVendorAdapter() {
          // 这里虽然是直接 super 的,但依然是不一样的,内部使用了通过 super(_, properties, _) 构造方法传入的参数.
          return super.jpaVendorAdapter();
      }
  
      @Override
      @Bean(DataSourceConfiguration.DB_PREFIX_2 + "_entityManagerFactoryBuilder")
      public EntityManagerFactoryBuilder entityManagerFactoryBuilder(@Qualifier(DataSourceConfiguration.DB_PREFIX_2 + "_jpaVendorAdapter") JpaVendorAdapter jpaVendorAdapter,
                                                                     ObjectProvider<PersistenceUnitManager> persistenceUnitManager,
                                                                     ObjectProvider<EntityManagerFactoryBuilderCustomizer> customizers) {
          return super.entityManagerFactoryBuilder(jpaVendorAdapter, persistenceUnitManager, customizers);
      }
  
      @Override
      @Bean(DataSourceConfiguration.DB_PREFIX_2 + "_entityManagerFactory")
      public LocalContainerEntityManagerFactoryBean entityManagerFactory(@Qualifier(DataSourceConfiguration.DB_PREFIX_2 + "_entityManagerFactoryBuilder") EntityManagerFactoryBuilder factoryBuilder) {
          Map<String, Object> vendorProperties = getVendorProperties();
          customizeVendorProperties(vendorProperties);
  
          return factoryBuilder
                  .dataSource(this.dataSource)
                  // TODO 实体所在包名
                  .packages("com.xxx2.entity")
                  .properties(vendorProperties)
                  .jta(isJta())
                  .build();
      }
  
      @Bean(DataSourceConfiguration.DB_PREFIX_2 + "_jpaTransactionManager")
      public JpaTransactionManager jpaTransactionManager(@Qualifier(DataSourceConfiguration.DB_PREFIX_2 + "_entityManagerFactory") LocalContainerEntityManagerFactoryBean entityManagerFactory) {
          return new JpaTransactionManager(Objects.requireNonNull(entityManagerFactory.getObject()));
      }
  
      @Override
      protected AbstractJpaVendorAdapter createJpaVendorAdapter() {
          return new HibernateJpaVendorAdapter();
      }
  
      @Override
      protected Map<String, Object> getVendorProperties() {
          return new LinkedHashMap<>(this.hibernateProperties.determineHibernateProperties(getProperties().getProperties(),
                  new HibernateSettings().ddlAuto(() -> "none").hibernatePropertiesCustomizers(this.hibernatePropertiesCustomizers)));
      }
  
      private List<HibernatePropertiesCustomizer> determineHibernatePropertiesCustomizers(ConfigurableListableBeanFactory beanFactory, List<HibernatePropertiesCustomizer> hibernatePropertiesCustomizers) {
          List<HibernatePropertiesCustomizer> customizers = new ArrayList<>();
          customizers.add((properties) -> properties.put(AvailableSettings.BEAN_CONTAINER, new SpringBeanContainer(beanFactory)));
          customizers.addAll(hibernatePropertiesCustomizers);
          return customizers;
      }
  }
  ```

## 多数据源注意事项

由于有多个数据源，所以在使用`@Transactional`注解时，Spring 可能不知道该注解所注释的方法需要在哪个数据源的事务中执行，为了解决此问题有两种方式可以根据实际情况选择不同的方式。

1. 通过`org.springframework.transaction.annotation.Transactional#transactionManager`（`javax.transaction.Transactional`不支持指定）明确指定需要使用的事务管理器。

2. 使用`@Primary`注解将其中一个`JpaTransactionManager`（事务管理器）标记为主事务管理器（默认使用的事务管理器）。在大部分多数据源项目中，只有一个数据源是需要写入的，其他数据源都是只读，这种情况下可以将这个需要写入的数据源对应的事务管理器标记为主要的。如果有多个数据源经常需要写入时不推荐使用这种方式，因为这种方式在不明确指定需要使用的事务管理器的情况下（默认使用主事务管理器）也不会报错，这可能会因为疏忽大意导致某些方法事务是不应该在主事务管理器中开启的。

   即使通过`@Primary`指定了默认事务管理器，也可以通过前面的方式明确更换为其他的事务管理器。
