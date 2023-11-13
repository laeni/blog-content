---
title: MyBatis 常用笔记记录
author: Laeni
tags: DB
date: 2023-11-11
updated: 2023-11-11
---

当搜索“MyBatis 的优势”之类的词时总会“灵活性”这个词。确实，因为用 MyBatis，你可以仅查询一张表中的部分字段、轻松关联多张表。由于可以关联多张表，所以只需要将查询结果共同用一个类对应接收即可，不需要每张表创建一个类，这简直不要太爽。如果你也觉得这样很爽，那你可能大部分时间都只是自己写，很少去看他人写的代码。然而，这种·约束性很弱的写法，说好听点是“灵活”，说难听点就是“无规范章法的乱来”，这种代码自己写着可能很爽，但别人维护起来一定很痛苦。

这不，这段时间接手了一堆 MyBatis 项目，经常出现一个类涵盖了几张表的字段、将近百个属性、SQL 动不动上百行、表动不动就是六七张关联、各种子查询、SQL 中经常出现带业务特性的条件拼接、插入更新时要仔细将 SQL 和传给 MyBatis 的 DTO 参数对比才知道该 DTO 中的属性哪些用到了哪些没用（因为在给 MyBatis 的参数中，即使不用也可以往里面塞，这具有很强的迷惑性），关键是还不写注释，偶而出现的注释都是无意义的，出现异常只是简单打印一个笼统的错误描述，原始堆栈信息直接丢掉。对业务又不熟悉的情况下，看得我都怀疑是不是应该转行。

吐槽归吐槽，事情该干还的干，在这里记录一些 MyBatis 相关的常用易错易望笔记。

# `in` 查询

MyBatis 似乎不支持自动生成 in 条件，所以要么自己将条件拼接好后通过`$`变量替换，要么使用 MyBatis 的 `foreach` 结构。一般使用后者，因为前者如果忘记转义的话可能发生 SQL 注入攻击。

用法参考：<https://blog.csdn.net/u011781521/article/details/79669180>。

## foreach

interface:

```java
List<Object> findAllByIdIn(@Param("ids") List<Long> ids);
```

mapper:

```xml
<select id="findAllByIdIn" resultType="map">
    select *
    from TABLE
    where ID in (
        <foreach collection="ids" item="id" separator=",">#{ruleCfgId}</foreach>
    )
</select>
```

