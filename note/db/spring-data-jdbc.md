---
title: Spring Data JDBC
author: 'Laeni'
tags: 'Java, Spring, Spring Data, JDBC'
date: '2022-09-19'
updated: '2022-10-24'
hide: true
---

[Spring Data JDBC](https://spring.io/projects/spring-data-jdbc) 是 Spring Data 系列的一部分，它使实现基于 JDBC 的存储库变得容易。其处理对基于 JDBC 的数据访问层的增强支持，使构建使用数据访问技术的 Spring 驱动的应用程序变得更加容易。Spring Data JDBC 旨在从概念上更简单，所以它不提供缓存，延迟加载，延迟写入或 JPA 的许多其他功能。

> 目前Spring Data JDBC不支持多数据源，如果有多数据源需求建议改用JPA。

## 常用示例

### 一对一映射

`LegoSet`聚合根：

```java
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import org.springframework.data.annotation.AccessType;
import org.springframework.data.annotation.AccessType.Type;
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Column;

@Data
@AccessType(Type.PROPERTY)
@AllArgsConstructor(access = AccessLevel.PACKAGE)
public class LegoSet {
	private @Id Integer id;
	/**
	 * "lego_set_id"为 manual 表中的属性（外键），用于与本表的 id 进行关联.
	 */
	@Column("lego_set_id")
	private Manual manual;
}
```

`Manual`聚合：

```java
@Data
@Table("manual")
public class Manual {
	private @Id Long id;
	@Column("lego_set_id")
	private Long legoSetId;
	private String author, text;

	public Manual(String text, String author) {
		this.id = null;
		this.author = author;
		this.text = text;
	}
}
```

### 一对多映射 - Map

以下示例聚合根(LegoSet)和聚合(Model)为一对多的关系，并且将Model中的某个属性作为Map的键。

`LegoSet`聚合根：

```java
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.With;
import org.springframework.data.annotation.AccessType;
import org.springframework.data.annotation.AccessType.Type;
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.MappedCollection;

import java.time.Period;
import java.time.temporal.ChronoUnit;
import java.util.HashMap;
import java.util.Map;

@Data
@AccessType(Type.PROPERTY)
@AllArgsConstructor(access = AccessLevel.PACKAGE)
public class LegoSet {

	private @Id Integer id;

  /**
   * "@MappedCollection"指定如何将子表中的数据映射到本字段，
   * "idColumn"为外键，即本表中的主键，标识和本表的关联关系；
   * "keyColumn"表示将 Model 中的哪个字段用作这里Map中的键.
   */
	@MappedCollection(idColumn = "lego_set_id", keyColumn = "name")
  @AccessType(Type.FIELD) @With(AccessLevel.PACKAGE)
	private final Map<String, Model> models;

	public LegoSet() {
		this.models = new HashMap<>();
	}

	public void addModel(String name, String description) {
		models.put(name, new Model(name, description));
	}
}
```

`Model`聚合：

```java
@lombok.Getter
@lombok.ToString
// 如果省略 @Table#value ，则表名将为实体名（全小写）
@org.springframework.data.relational.core.mapping.Table("Model")
public class Model {
    private final String name, description;

    public Model(String name, String description) {
        this.name = name;
        this.description = description;
    }
}
```

