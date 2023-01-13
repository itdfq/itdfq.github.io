---
title: Expected one result (or null) to be returned XXXX
top: false
cover: false
toc: true
mathjax: true
date: 2023-01-13 10:53:18
password:
summary: mysql进行查询的时候，发现实际查询和设置的返回类型不一致,导致出错
tags:
- mysql
- 原创
categories:
- sql
---
### nested exception is org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne()


#### 举例

假如说现在有个sql,表a 和 表b 是一对多的关系,现在需要查询出a表的id，且满足以下条件子表b的score需要大于3
然后现在这个查询就会出问题

```sql
select count(a.id) from a  left join b on a.item_id = b.item_id  where b.score>3 group by a.item_id
```
报错：
nested exception is org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne()

#### 原因： 
* group by 分组之后是多个列，但是count 函数返回的是单列 导致返回的条数出错
* 这种情况需要加个临时表，先计算出id,然后再count(1) 获取所有的id

#### 解决办法：

```sql
select count(1) from (select a.id from a group by a.item_id) tmp
```

注意：
使用mysql 临时表必须加上表名，不然会报错
SQL 错误 [1248] [42000]: Every derived table must have its own alias
每一个派生出来的表都必须有一个自己的别名