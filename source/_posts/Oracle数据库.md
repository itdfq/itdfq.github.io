---
title: Oracle数据库笔记
top: false
cover: false
toc: true
mathjax: true
date: 2020-07-30 17:36:32
password:
summary: 本文主要介绍Oracle的一些基本知识。
tags:
- 原创
- 数据库
categories:
- Oracle数据库
---

# <font color='red'>Oracle数据库笔记</font>
* ### 创建表空间
```create tablespace itheima
datafile 'c:\itheima.dbf'
size 100m
autoextend on
next 10m;
```


* ### 删除表空间
```
drop tablespace itheima;
```
* ### 创建用户 
```
create user itheima
identified by itheima
default tablespace itheima;
                   --给用户授权
                   --oracle数据库中常用角色
connect            --连接角色，基本角色
resource           --开发者角色
dba                --超级管理员角色
```
* ### 给itheima授予dba角色
```
grant dba to itheima;
```

* ## 切换到itheima用户下
* ### 创建一个person表
```
create table person(
       pid number(20),
       name varchar2(10)
);
```
* ### 查询列表
```
select * from person;
grant dba to itheima;
```
* ### 添加一列
```
alter table person add(gender number(1));
```
* ### 删除一列
```
alter table person drop column gender;
```
* ### 修改列名称
```
alter table person rename column gender to sex;
```
* ### 修改列类型
```
alter table person modify gender char(1);
```
* ### 添加一条记录(CRUD都得提交事务)
```
insert into person (pid,name) values(5,'小明');
commit;
```
* ### 修改数据
```
update person set name = '小马' where pid =5;
```
 * ### 删除表中所有记录
 ```
delete from person;
```
* ### 删除表结构
```
drop table person;
```

 先删除表，再次创建表，效果等用于删除表中的全部记录。在数据量的大的情况下，尤其在表中带有索引的情况下，该操作的小路很高。
索引可以提高查询效率，但是会影响增删改查。
```
truncate table person;
```
序列不真的属于任何一张表，但是可以逻辑和表绑定

序列:默认从1开始一次递增,主要给主键复制使用
```
nextval下一个值，currval当前值

dual:虚表，知识为了不全语法，没有意义
create sequence s_person;
select s_person.nextval from dual;
insert into person(pid,name)values(s_person.nextval,'小敏');
commit;
```
scott用户，密码默认是：tiger（刚安装数据库是被锁定的，得进行解锁）

* ## 初学者必学
* ### 解锁
```
alter  user scott account unlock;
--解锁密码(重置密码)
alter user scott identified by tiger;
--切换到scoot用户
--切换到scott用户
--单行函数：作用于一行，返回一个值
---字符函数
select upper('yes') from dual;--YES
select lower('YES') from dual;
--数值函数
select round(26.18,1) from dual;--四舍五入26.2
select round(26.18,-1) from dual;--四舍五入30
select mod(10,3) from dual; --求余数1
--日期函数
--查询出emp表中所有员工距离现在几天
select  sysdate-e.hiredate from emp e;
--算出明天此刻
select sysdate+1 from dual;
---转换函数
--日期转换字符串
select to_char(sysdate,'fm yyyy-mm-dd hh24:mi:ss') from dual;
--字符串转换日期
select to_date('2018-6-7 16:39:50','fm yyyy-mm-dd hh24:mi:ss') from dual;
--通用函数
--算出emp表中所有员工的年薪
--奖金里面有null
select e.sal*12+nvl(e.comm,0 )from emp e;
---条件表达式
给emp表中员工起中文名称
select e.ename,
       case e.ename
             when 'SMTH' then '曹贼' 
                  when 'ALLEN'then '大耳贼'
                       when 'WARD' then '诸葛小二'
                                           else'无名'
                                                  end
from emp e;
--oracle除了起别名都用单引号，
--oracle专用条件表达式
select e.ename,
   decode(e.ename,
       'SMTH',  '曹贼', 
           'ALLEN', '大耳贼',
               'WARD', '诸葛小二',
                   '无名') 中文名                    
from emp e;
```



* ## 多行函数（聚合函数）：作用于多行，返回一个值
```
select count(1) from emp;--查询总数量 count(主键这一列)
select sum(sal) from emp;--工资总和
select min(sal) from emp;--最大工资
select avg(sal) from emp;--平均工资
--分组查询
--查询出每个部门的平均工资
--分组查询中，出现在group by 后面的原始列，才能出现在select后面
--没有出现在group by 后面的列，想在select后面，必须要加上聚合函数。
--聚合函数可以吧多行记录变为一个值
select e.deptno,avg(e.sal)
from emp e 
group by e.deptno;
---查询平均工资高于2000
select e.deptno,avg(e.sal) asal
from emp e 
group by e.deptno
having avg(e.sal)>2000;
--所有条件都不能使用别名来判断
select ename,sal s from emp where sal>1500;
--查询出每个部门工资高于800的员工的平均工资
select e.deptno,avg(e.sal) asal
from emp e
where e.sal>800 
group by e.deptno;
--where是过滤分组签的数据，having是过滤分组后的数据。
--表现形式：where必须在group by 之前，having是在group by 之后。
--查询出每个部门工资高于800的员工的平均工资，然后在查询出平均工资高于2000的员工
select e.deptno,avg(e.sal) asal
from emp e
where e.sal>800 
group by e.deptno
```

 