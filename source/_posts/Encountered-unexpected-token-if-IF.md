---
title: 'Encountered unexpected token: if IF ='
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-27 13:32:30
password:
summary: 使用mybatis遇到的bug
tags:
- 原创
- BUG
  categories:
- BUG
---
### 报错信息如下：
```text
when useing jsqlparser parser this sql then throw a exception:Encountered unexpected token: "if" "IF"
Caused by: net.sf.jsqlparser.JSQLParserException
at net.sf.jsqlparser.parser.CCJSqlParserUtil.parseStatements(CCJSqlParserUtil.java:137)
at com.baomidou.mybatisplus.core.parser.AbstractJsqlParser.parser(AbstractJsqlParser.java:60)
... 47 more
Caused by: net.sf.jsqlparser.parser.ParseException: Encountered unexpected token: "if" "IF"
at line 10, column 9.
```

### 原因
使用Mybatis 进行使用，采用JSQLParser 对sql进行解析，但是低版本的不支持IF("","","")函数，可以使用case when 进行代替

例如：
```sql
   IF(lvi.id is not null,1,0) as usefulVideo,
   IF(pspsv.id is not null,1,0) as uploadedVideo
```

可以替换为：
```sql
CASE
WHEN lvi.id is NULL THEN
0 ELSE 1
END AS usefulVideo,
CASE
WHEN pspsv.id is NULL THEN
0 ELSE 1
END AS uploadedVideo
```

### 参考文档：
* [jsqlparser无法解析 "if" "IF" "="等MySQL语法](https://www.jianshu.com/p/44a3cc19a003)

* [Encountered unexpected token: "if" "IF" "=" · Issue #1648 · JSQLParser/JSqlParser (github.com)](https://github.com/JSQLParser/JSqlParser/issues/1648)