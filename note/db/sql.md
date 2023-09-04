---
title: SQL经典案例
author: 'Laeni'
tags: DataSource
date: '2022-11-30'
updated: '2022-11-30'
---

- [查询每个班前三名](https://blog.csdn.net/weixin_39428938/article/details/98214175/)

  ```sql
  create table test_score
  (
      CLASS VARCHAR(10),
      NAME  VARCHAR(10),
      SCORE INTEGER
  );
  
  select class, name, score
  from test_score a
  where (select count(*) from test_score where class = a.class and a.score < score) < 3;
  ```

