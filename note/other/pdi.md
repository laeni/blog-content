# 注意事项

## 合并分支

分支合并前，所有分支的**名称**、**类型**、**顺序**和**个数**需完全相同。

## Database join

该组件，如果不勾选`Outer join`则当没查询到结果时会丢失原记录。

## Java 代码

- Set 值时，基本类型需要转换为包装类型

  ```java
  get(Fields.Out, "NAME").setValue(r, Boolean.valueOf(true));
  get(Fields.Out, "NAME").setValue(r, Long.valueOf(1));
  ```

- 数字类型处理。

  大部分情况下，设置`Intger`类值时，需要转为`Long`型再设置。

  > SQL 返回的数字为`Long`型。