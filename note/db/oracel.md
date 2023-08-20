# 常用操作

## 表空间

### 查看表空间物理文件

```sql
select file_name,tablespace_name from dba_data_files;
```

### 创建表空间

```sql
-- 创建一个最大容量为10G的表空间，物理文件为 /u01/app/oracle/oradata/dev/XXX.DBF
create tablespace TBS_IPS datafile '/u01/app/oracle/oradata/dev/XXX.DBF' SIZE 10g;
```

## 安全

### 授权角色给用户

```sql
GRANT <ROLE1>[, ROLE2[, ROLE3...]] TO <username>;
```

#### 常见角色

- CONNECT - 允许用户连接数据库
- RESOURCE - 允许用户创建表等对象

