---
title: 'MySQL的锁'
author: '周迪'
tags: MySQL, DataSource
date: '2022-03-04'
updated: '2022-03-05'
---

## 锁的定义

### 什么是数据库的锁

**通用解释：**数据库的锁实际是一种包含了各种信息的**数据结构**，主要用于对数据库事物进行并发控制

[**MySQL解释：**](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_lock)

> The high-level notion of an object that controls access to a resource, such as a table, row, or internal data structure, as part of a **locking** strategy. For intensive performance tuning, you might delve into the actual structures that implement locks, such as **mutexes** and **latches**

**简要翻译：**

>锁是一种可以控制表、行记录或者内部数据结构等资源访问的高级概念对象，也是锁策略的一部分。如果要做精细的性能调优，则需要钻研锁的具体实现对应的数据结构，如 mutexes 和 latches

**加锁：**就是内存里面生成相应的锁结构（在InnoDB里，同一个事物，同一个页面，相同的锁类型，共享同一个锁结构）

**作为数据结构的示例：**

<img src="https://edcdn.czxtech.cn/pics/16a68cda54348429~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp" alt="image_1d9kcpbl5178l1i691bg21jk61lv09.png-101.8kB" style="zoom: 50%;" />

## 锁的分类

### 按模式

* 共享锁（Shared Locks）(S)
* 独占锁（Exclusive Locks）(X)

### 按范围

* 全局锁

> 全局读锁，整个库处于只读状态（一般用在做全库的逻辑备份时）
>
> 加锁：flush tables with read lock（FTWRL）
>
> 解锁：1、unlock tables; 2、客户端断开连接，自动释放;

* 表级锁
  * 表锁（lock table xxx read/write）
  * 元数据锁（MDL锁-访问锁的时候自动加上，Server实现）
  * 意向锁（InnoDB实现）
  * 自增锁（InnoDB实现）
* 行级锁（InnoDB实现）

### [InnoDB行级锁](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)

* 记录锁 Record Locks（lock_mode X locks rec but not gap）

> 锁特定的索引记录，要么是唯一索引 要么是主键

* 间隙锁 Gap Locks（lock_mode X locks gap before rec）

>锁索引记录中的区间（间隙），不止区间里的记录，左开右开，对索引的间隙加锁，防止其他事物插入数据
>
>产生条件：

* 临键锁 Next-Key Locks（lock_mode X）

> 间隙锁 和 记录锁合并称为临键锁；InnoDB行锁加锁的基本单位；（某些特定场合会退化为记录锁或者间隙锁）
>
> 在RR隔离级别下，就是通过临键锁来阻止**幻读**现象的发生

* 插入意向锁 Insert Intention Locks（lock_mode X locks gap before rec insert intention）

> 插入意向锁是一种针对insert操作的**间隙锁**，多个事物在同一个索引区间范围插入不冲突的记录时，不会阻塞

* 隐式锁 

>当事物需要加锁时，这个锁又不会发生冲突，则会直接跳过加锁环节的一种性能提升机制

## 案例

### InnoDB行锁实践

#### 影响因素

* 事物隔离级别（Repeatable Read）
* **索引类型**（聚簇索引、唯一二级索引、普通二级索引）
* **匹配类型**（精确匹配、范围匹配、未匹配）
* 具体语句类型（SELECT INSERT DELETE UPDATE）
* 是否被标记删除
* 数据库版本

#### 环境准备

* 设置隔离级别为RR

```sql
set session transaction isolation level repeatable read;
```

* 打开监视器

```sql
set global innodb_status_output_locks=ON;
```

* 开启日志

```sql
set global innodb_status_output=ON;
```

* 查看锁详情

```sql
show engine innodb status \G;
```

* 创建表

```sql
create table t_row_lock
(
    pk int not null comment '主键'
        primary key,
    ui int not null comment '唯一索引',
    i  int not null comment '普通索引',
    v  int not null comment '值',
    constraint ui
        unique (ui)
);

create index i
    on t_row_lock (i);
```

* 插入数据

```sql
INSERT INTO t_row_lock (pk, ui, i, v) VALUES (1, 1, 1, 1);
INSERT INTO t_row_lock (pk, ui, i, v) VALUES (5, 5, 5, 5);
INSERT INTO t_row_lock (pk, ui, i, v) VALUES (10, 10, 10, 10);
INSERT INTO t_row_lock (pk, ui, i, v) VALUES (15, 15, 15, 15);
INSERT INTO t_row_lock (pk, ui, i, v) VALUES (20, 20, 20, 20);
INSERT INTO t_row_lock (pk, ui, i, v) VALUES (25, 25, 25, 25);
```

#### 测试维度（索引类型 + 匹配类型）

##### 1. 聚簇索引 + 精确匹配

```sql
begin ;
select * from t_row_lock s where s.pk = 1 for update;
```

##### 2. 聚簇索引 + 范围匹配

```sql
begin ;
select * from t_row_lock s where s.pk > 1 for update;
```

##### 3. 聚簇索引 + 未匹配

```sql
begin ;
select * from t_row_lock s where s.pk = 2 for update;
```

##### 4. 唯一索引 + 精确匹配

```sql
begin ;
select * from t_row_lock s where s.ui = 1 for update;
```

##### 5. 唯一索引 + 范围匹配

```sql
begin ;
select * from t_row_lock s where s.ui > 1 and s.ui <= 10 for update;
```

##### 6. 唯一索引 + 未匹配

```sql
begin ;
select * from t_row_lock s where s.ui = 2 for update;
```

##### 7. 普通索引 + 精确匹配

```sql
begin ;
select * from t_row_lock s where s.i = 1 for update;
```

##### 8. 普通索引 + 范围匹配

```sql
begin ;
select * from t_row_lock s where s.i > 1 and s.i <= 10 for update;
```

##### 9. 普通索引 + 未匹配

```sql
begin ;
select * from t_row_lock s where s.i = 2 for update;
```

##### 10. 无索引

```sql
begin ;
select * from t_row_lock s where s.v = 1 for update;
```

#### 测试结果

##### 1. 聚簇索引 + 精确匹配

```sql
begin ;
select * from t_row_lock s where s.pk = 1 for update;
```

锁类型：记录锁

锁个数：1

涉及索引：PRIMARY

锁详情：索引PRIMARY上 记录锁1

##### 2. 聚簇索引 + 范围匹配

```sql
begin ;
select * from t_row_lock s where s.pk > 1 for update;
```

锁类型：临键锁

锁个数：6

涉及索引：PRIMARY

锁详情：索引PRIMARY上 临键锁(1,5] (5,10] (10,15] (15,20] (20,25] (25,supremum]

##### 3. 聚簇索引 + 未匹配

```sql
begin ;
select * from t_row_lock s where s.pk = 2 for update;
```

锁类型：间隙锁

锁个数：1

涉及索引：PRIMARY

锁详情：索引PRIMARY上 间隙锁(1,5)

##### 4. 唯一索引 + 精确匹配

```sql
begin ;
select * from t_row_lock s where s.ui = 1 for update;
```

锁类型：记录锁

锁个数：2

涉及索引：UI、PRIMARY

锁详情：索引UI上 记录锁 1，索引PRIMARY上 记录锁 1

##### 5. 唯一索引 + 范围匹配

```sql
begin ;
select * from t_row_lock s where s.ui > 1 and s.ui <= 10 for update;
```

锁类型：临键锁、记录锁

锁个数：5

涉及索引：UI、PRIMARY

锁详情：索引UI上 临键锁 (1,5] (5,10] (10,15]，索引PRIMARY上 记录锁 5、10

##### 6. 唯一索引 + 未匹配

```sql
begin ;
select * from t_row_lock s where s.ui = 2 for update;
```

锁类型：间隙锁

锁个数：1

涉及索引：UI

锁详情：索引UI上间隙锁 (1,5)

##### 7. 普通索引 + 精确匹配

```sql
begin ;
select * from t_row_lock s where s.i = 1 for update;
```

锁类型：临键锁、间隙锁、记录锁

锁个数：3

涉及索引：PRIMARY、I

锁详情：索引PRIMARY上 记录锁 1，索引I上 临键锁 (infimum,1] 间隙锁 (1,5) 

##### 8. 普通索引 + 范围匹配

```sql
begin ;
select * from t_row_lock s where s.i > 1 and s.i <= 10 for update;
```

锁类型：临键锁、记录锁

锁个数：5

涉及索引：PRIMARY、I

锁详情：索引PRIMARY上 记录锁 5 10，索引I上 临键锁 (1,5] (5,10] (10,15] 

##### 9. 普通索引 + 未匹配

```sql
begin ;
select * from t_row_lock s where s.i = 2 for update;
```

锁类型：间隙锁

锁个数：1

涉及索引：I

锁详情：索引I上 间隙锁 (1,5)

##### 10. 无索引

```sql
begin ;
select * from t_row_lock s where s.v = 1 for update;
```

锁类型：临键锁

锁个数：7

涉及索引：PRIMARY

锁详情：索引PRIMARY上 临键锁 (infimum,1] (1,5] (5,10] (10,15] (15,20] (20,25] (25,supermum]

## 总结

#### 加锁规则（InnoDB行锁）

1. [加锁基本单位：临键锁（Next-Key Locks）](https://dev.mysql.com/doc/refman/8.0/en/innodb-locks-set.html)
2. 只有访问到的对象（用到的索引）才会加锁
3. 唯一索引（包括聚簇索引）等值匹配，记录存在时，临键锁退化为记录锁
4. 索引上的等值匹配，向右遍历且最后一个值不满足等值条件的时候，临键锁退化为间隙锁
5. **（版本差异）**唯一索引上的范围查询会访问到不满足条件的第一个值为止**（语义问题）**
6. 根据加锁成本选择最终的锁
