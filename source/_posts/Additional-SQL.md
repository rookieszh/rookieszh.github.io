---
title: Additional SQL
date: 2019-03-09 11:01:03
categories:
    - SQL
tags:
    - Mysql
    - index
description: "本文将介绍Mysql的其他特性：存储过程、视图view、触发器、索引"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---


#### 存储过程
> 存储过程是用户定义的一系列sql语句的集合，涉及特定表或其它对象的任务，用户可以调用存储过程，而函数通常是数据库已定义的方法，它接收参数并返回某种类型的值并且不涉及特定用户表。
##### 存储过程和函数的区别
1. 一般来说，存储过程实现的功能要复杂一点，而函数的实现的功能针对性比较强。存储过程，功能强大，可以执行包括修改表等一系列数据库操作；用户定义函数不能用于执行一组修改全局数据库状态的操作。
2. 对于存储过程来说可以返回参数，如记录集，而函数只能返回值或者表对象。函数只能返回一个变量；而存储过程可以返回多个。存储过程的参数可以有IN,OUT,INOUT三种类型，而函数只能有IN类~~存储过程声明时不需要返回类型，而函数声明时需要描述返回类型，且函数体中必须包含一个有效的RETURN语句。
3. 存储过程，可以使用非确定函数，不允许在用户定义函数主体中内置非确定函数。
4. 存储过程一般是作为一个独立的部分来执行（ EXECUTE 语句执行），而函数可以作为查询语句的一个部分来调用（SELECT调用），由于函数可以返回一个表对象，因此它可以在查询语句中位于FROM关键字的后面。 SQL语句中不可用存储过程，而可以使用函数。
##### 编写存储过程
```sql
-- 存储过程函数
delimiter // -- 修改结束符，避免冲突

create procedure he(a int)
begin
select * from student where id=a;
end//

delimiter ;
call he(1)
```
#### 视图VIEW
```sql
-- 视图其实就是当作一个查询表来使用
CREATE view 视图名(字段名1，字段名2，....) as select 语句;
ALTER view 视图名(字段名1，字段名2，....) as select 语句;
SELECT * FROM 视图名;
drop view (if exists) 视图名;
```
#### 触发器
> 在对某个表进行操作之前/之后，进行另一个操作
```sql
delimiter //
create trigger 触发器名 after/before insert/update/delete on 表名 for each row
begin
sql语句;
end//
delimiter ;
```
#### 索引
> 索引在提供查找速度的同时，会降低增删改的速度
```sql
-- 唯一索引：不可以出现相同的值，可以有NULL值
UNIQUE
-- 普通索引：允许出现相同的索引内容；
INDEX
-- 主键索引：不允许出现相同的值
PROMARY KEY
-- 全文索引：可以针对值中的某个单词，但效率很低
fulltext
-- 组合索引：实质上是将多个字段建到一个索引里，列值的组合必须唯一

-- 添加索引
alter table 表名 add index/unique/fulltext/primary key [索引名] (字段名);
-- 删除索引
alter table 表名 drop index 索引名;
查看表的所有索引
show index from 表名;
-- Table: 表的名称
-- Non_unique: 如果索引不能包括重复词，则为0。如果可以，则为1
-- Key_name: 索引的名称
-- Seq_in_index: 索引中的列序列号，从1开始。
-- Column_name: 字段名称
-- Collation: 列以什么方式存储在索引中。在MySQL中，有值‘A’（升序）或NULL（无分类）。
-- Cardinality: 索引中唯一值的数目的估计值。通过运行ANALYZE TABLE或myisamchk -a可以更新。基数根据被存储为整数的统计数据来计数，所以即使对于小型表，该值也没有必要是精确的。基数越大，当进行联合时，MySQL使用该索引的机 会就越大。
-- Sub_part: 如果列只是被部分地编入索引，则为被编入索引的字符的数目。如果整列被编入索引，则为NULL。
-- Packed: 指示关键字如何被压缩。如果没有被压缩，则为NULL。
-- Null: 如果列含有NULL，则含有YES。如果没有，则该列含有NO。
-- Index_type: 用过的索引方法（BTREE, FULLTEXT, HASH, RTREE）。
-- Comment

-- 全文索引下模糊查询
select * from 表名 where match(字段名) against(查询值);
-- 组合索引
index(字段名1,字段名2,....);
-- 外键约束
foreign key(字段名) references 表名(字段名);
```
