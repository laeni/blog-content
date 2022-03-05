---
title: 'shardingsphere-jdbc 4.1版本 SpringBoot 版本配置数据分片'
author: 'Laeni'
tags: 'Java, DataSource, 分库分表, shardingsphere-jdbc'
date: '2022-03-04'
updated: '2022-03-05'
---

这里简单记录一下`SpringBoot`通过配置方式使用`shardingsphere-jdbc`进行数据分片，虽然目前已经更新到`5.1.0`版本，但是单凭一个`shardingsphere-jdbc-core`依赖就能让项目大小增加`56MB+`，再加上其依赖及其复杂，很容易发生依赖冲突，所以不考虑使用该版本。但是从功能看还是很期待，期待`6.x`版本会不会有所改善。

另外，虽然`shardingsphere-jdbc`能通过很多方式使用，但是建议使用`Spring Boot`方式进行配置，原因倒不是因为该方式简单，而是因为该项目不同版本之间API变化较大，所以能不硬编码就不硬编码。即使不使用`Spring Boot`的也可以使用`DataSource dataSource = YamlShardingDataSourceFactory.createDataSource(yamlFile);`方式从yaml配置创建数据源。

## 准备

在进行使用之前，需要对其相关的概念有一定理解，这样才能度对配置进行随机应变，具体可以参看[官网](https://shardingsphere.apache.org/document/4.1.1/cn/features/sharding/)。

## 依赖

这里直接使用[官方](https://shardingsphere.apache.org/document/4.1.1/cn/manual/sharding-jdbc/usage/sharding/#%E5%BC%95%E5%85%A5maven%E4%BE%9D%E8%B5%96-1)提供的`SpringBoot`版本的依赖。

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.x</version>
</dependency>
```

## 基于Spring boot的规则配置

### 真实数据源配置

由于最终是使用`shardingsphere-jdbc`生成的代理数据源，所以这里的数据源只是一个半成品。并且这里要注意，如果使用`MySQL`数据源的话，最好不要使用`druid`，因为`druid`直接创建出来SET属性的会有Bug，以至于不能使用配置方式创建数据源，比如[#3059](https://github.com/alibaba/druid/issues/3059)所描述的为问题。

> 目前我们组已经从`druid`转向`hikari`，相比之下，目前`hikari`更好用，比如支持不停机更新配置（使用Spring原生方式配置数据再加上配置中心尤为方便）、Spring 默认集成而不用引入更多依赖等。

```yaml
spring:
  shardingsphere:
    datasource:
      names: db0,db1 # 列出所有真实数据源名称，多数据源以逗号分隔. -- 这里暂不清楚下面已经配置为什么还要在这里再些一次，不过问题不大
      db0:
        type: com.zaxxer.hikari.HikariDataSource #数据库连接池类名称
        driver-class-name: com.mysql.cj.jdbc.Driver #数据库驱动类名
        jdbc-url: jdbc:mysql://...
        username: ...
        password: ...
        # 数据库连接池的其它属性
      db1:
        type: com.zaxxer.hikari.HikariDataSource #数据库连接池类名称
        driver-class-name: com.mysql.cj.jdbc.Driver #数据库驱动类名
        jdbc-url: jdbc:mysql://...
        username: ...
        password: ...
        # 数据库连接池的其它属性
```

### 分片配置

```yaml
# 准确为指定表配置相关的策略
spring:
  shardingsphere:
    sharding:
      tables:
        table_name: # 待分表的表明
          logic-table: tbl_personal_customer # [可选]逻辑表名 - 一般与前面的Key(表名)相同，至于什么情况下不同没有研究过
          # [必须]实际数据节点 - 必须把该表相关的所有节点表以`数据源名.表名`的格式全部列出来
          # 数据源名取值为创建数据时配置的名字(spring.shardingsphere.datasource.names)之一
          # 该字符串可以为`Groovy`表达式，且执行之后结果为数组，也可以是逗号分割的多个字符串，再或者是组合出现，如下示例所示：
          #    cs.table_name_0,cs.table_name_1,...,cs.table_name_9
          #    $->{['cs.table_name_0','cs.table_name_1',...,'cs.table_name_9']} // 与上面的等效
          #    cs.table_name_0,cs.table_name_$->{1..9} // 与上面的等效
          #    cs.table_name_$->{0..9} // 与上面的等效
          # 这里全部列出来的原因是：如果SQL条件中不包含分片列时需要访问全部分片表，此时需要用到
          actual-data-nodes: "$->{['cs.table_name_0','cs.table_name_1',...,'cs.table_name_9']}"
          database-strategy: # [只有需要分库时才配置]分库策略 - 算法以数组形式返回该次SQL实际对应的数据源名称
            # 一般情况下只会配置一种策略，是否可以同时配置多种策略尚未可知
            # 其他算法的适用场景及用法参加官网： https://shardingsphere.apache.org/document/4.1.1/cn/features/sharding/concept/sharding/
            complex: # 复合分片策略，对应复合分片算法
              shardingColumns: name,xxx # 由于计算分片的列
              algorithmClassName: com.xxx.DemoDataSuorceShardingAlgorithm # 算法对应类的全限定名称，需要实现 ComplexKeysShardingAlgorithm 接口
          table-strategy: # [只有需要分表时才配置]分表策略 - 算法以数组形式返回该次SQL实际对应的表名称 - 配置方式完全和分库策略一致
            complex:
              shardingColumns: xxx
              algorithmClassName: com.xxx.DemoTableShardingAlgorithm
          key-generator: # [可选]自增主键生成策略 - 通过在客户端生成自增主键替换以数据库原生自增主键的方式，做到分布式主键无重复
            column: id
            type: SNOWFLAKE
        table_name2: # 配置的另外一张表
          # ....

---

# 配置默认的一些策略，这是策略全部是可选的，并且一般不用配置
spring:
  shardingsphere:
    sharding:
      binding-tables: # 绑定表 - 如果指定的话,在某些查询种会有性能优化
        - t_order,t_order_item
        - tab_1,tab_2 # 另一组绑定表
      broadcast-tables: # 广播表
        - t_config
      default-data-source-name: ds0 # 默认数据源
      default-database-strategy: # 默认分库策略
        inline:
          algorithm-expression: ds$->{user_id % 2}
          sharding-column: user_id
      default-table-strategy: # 默认分表策略
        none:
      default-key-generator: # 默认主键生成策略
        type: SNOWFLAKE
        column: id
```

