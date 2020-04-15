---
title: SQL
date: 2019-03-06 14:27:12
categories:
    - SQL
tags:
    - Mysql
description: "紧接着上篇Mysql，本文介绍了SQL的书写规则以及sql语句五大分类"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

> SQL——Structured Query Language 结构化查询语言
#### 书写规则
```
1. 分号结尾
2. 关键词不区分大小写
3. % 任意字符
4. _ 单个字符

```
#### 注释方法
##### 单行注释
```sql
# 注释内容
-- 注释内容，注意有空格
```
##### 多行注释
```sql
/*注释*/
```
#### 字段类型/修饰
```sql
unsigned：无符号
auto_increment：自增
default：默认值
comment：对字段解释说明
null：空
not null：非空
unique：唯一索引
index：普通索引
primary key：主键
auto_increment必须指定为primary key
zerofill：位数不足的地方补零
```
#### 语句运算符
```sql
= ：赋值和判断都是，没有 ==
!=、<>、<、>、<=、>=
OR：或者
AND：且
BETWEEN...AND：在...和...之间（范围）
IN：eg.in(1,2,3,4)给定具体范围
NOT IN
```
#### SQL语句的分类
```sql
1、DDL（data defination language）数据定义语言，作用：创建 删除 修改 库表结构
2、DML（data manipulation language）数据操作语言，作用：增 删 改表的记录
3、DCL（data control language）数据控制语言，作用：用户的创建以及授权
4、DQL（data query language）数据查询语言，作用：查询数据
```
##### DDL
```sql
-- 查看数据库
SHOW DATABASES;
-- 切换数据库
USE 数据库名;
-- 创建数据库
CREATE DATABASE 数据库名;
-- 删除数据库
DROP DATABASE 数据库名;
-- 查看数据库版本，字符编码等信息
\s
-- 查看数据库所用什么字符型式
show variables like '%char%';
-- 解决显示乱码问题
修改 my.ini 配置文件中 default-character-set=gbk;
-- 数据库导出数据
mysqldump -uroot -p密码 数据库名称 表名称 >表名称.sql
-- 修改结束符
delimiter 自定义的结束符;

-- 创建数据表
CREATE TABLE 表名(
    字段名 字段类型 其他关键词,
    ...
)
-- 查看当前数据库的表
SHOW TABLES;
-- 查看表结构
DESC 表名;
-- 查看创建表的SQL语句
SHOW CREATE TABEL 表名;
-- 删除表，但有外键关联的时候不能直接删除主表
DROP TABLE 表名;
-- 添加表字段
ALTER TABLE 表名 ADD (字段名 字段类型, 字段名 字段类型, ...); -- 默认添加到所有字段之后
ALTER TABLE 表名 ADD 字段名1 字段类型 AFTER/FIRST 字段名2; -- 将字段名1添加到字段名2之后/放在表头
-- 删除表字段
ALTER TABLE 表名 DROP 字段名;
-- 修改表字段类型
ALTER TABLE 表名 MODIFY 字段名 字段类型; -- 别忘了not null
-- 修改表字段名
ALTER TABLE 表名 CHANGE 原字段名 新字段名 字段类型;
-- 修改表名
ALTER TABLE 表名 RENAME (TO) 新表名;
-- 外键关联主键，fk为约束名
CREATE TABLE tbl_book(
  id INT PRIMARY KEY AUTO_INCREMENT,
  bookTypeId INT,
  CONSTRAINT `fk` FOREIGN KEY (`bookTypeId`) REFERENCES `tbl_bookType`(`id`)
);
```
##### DML
```sql
-- 查看表内容
select * from 表名;
-- 插入字段值
INSERT INTO 表名(列1, 列2, 列3...) VALUES(列值1, 列值2.....); -- 列和值是一一对应的
INSERT INTO 表名 values(值1, 值2,...,值n); -- 这样做需要把每个字段的值都一一对应起来
-- 更新字段值
UPDATE 表名 SET 列1=列值1, 列2=列值2, ... WHERE 条件; -- 不加where条件会修改所有的记录
-- 删除字段值，同样不加where条件会删除所有的记录
DELETE FROM 表名 WHERE 条件;
-- 删除整个表，这个是DDL的指令，好处是再次插入id从1开始。
-- 用delete则是从后面继续插入，前面的id不会再次使用
TRUNCATE 表名;
```
##### DCL
```sql
-- 修改登录权限，数据库mysql中的表user
-- 表中字段Host代表可登陆ip
-- 登录默认 -h 是localhost/127.0.0.1。
-- 刷新权限
flush privilges;
-- 普通用户的创建
create user '用户名'@'指定ip' identified by '用户名密码'; -- 指定ip处可输入%代表匹配所有ip（不安全）
-- 删除用户
drop user '用户名'@'指定ip';
-- 用户的授权，权限一般有 insert update delete select
grant 权限1,权限2,...on 数据库名.* to '用户名'@'ip地址或者%';
grant all privileges on *.* to '用户名'@'ip地址或者%';
-- 创建用户并授权
grant 权限1,权限2,...on 数据库名.* to '用户名（新建）'@'ip地址或者%' identified by '新密码';
-- 撤销权限
revoke 权限1,权限2,.. on 数据库名.* from '用户名'@'ip地址或者%';
-- 查看权限
show grants for '用户名'@'IP地址'(\G)
-- 删除用户
drop user'用户名'@'IP地址'
```
##### DQL
```sql
-- ###########################################################
-- ##################### 0. 常用关键字 ########################
-- ###########################################################
WHERE 条件;
AS 新的字段名;
\G; -- 更格式化的输出
EXISTS --假如子查询查询到记录，则进行外层查询，否则不执行外层查询
ANY --表示满足其中任一条件
ALL --表示满足所有条件

-- ###########################################################
-- ###################### 1. 普通查询 #########################
-- ###########################################################
-- SELECT 普通查询
SELECT 字段名1,字段名2,..... FROM 表名;
-- DISTINCT 过滤掉重复的值
SELECT DISTINCT 字段名1,字段名2,... FROM 表名;
-- CONCAT 连接数个字段（无空格连接）
SELECT CONCAT(字段名1,字段名2,...) FROM 表名;
-- CONCAT_WS 连接数个字段并重新命名并加入分隔符
SELECT CONCAT_WS('你定义的分隔符',字段名1,字段名2,.....) FROM 表名;
-- 模糊查询（LIKE只适用于数据量很小的情况下，数据量大的话使用sphinx）
SELECT 字段名1,字段名2,..... FROM 表名 字段名 LIKE '%模糊条件%';
-- ORDER BY 排序
SELECT * FROM 表名 ORDER BY 字段名 ASC/DESC; -- 升序或降序
-- GROUP BY，分组查询，单独使用毫无意义
-- 与 GROUP_CONCAT 一起使用，GROUP_CONCAT里包含的字段，以默认逗号连接，以 GROUP BY 的字段为分类标准输出
SELECT mathpixtype,GROUP_CONCAT(nickname) FROM weixinuserinfo GROUP BY mathpixtype;
+--------------+-----------------------------------+
| mathpixtype  | group_concat(nickname)            |
+--------------+-----------------------------------+
| asciimath    | presume                           |
| latex_styled | 王雅萱                            |
| mathml       | 痒肉多,悦,城北徐公,Jasper,Suntion3 |
| text         | 乐多,Law.D.Ares,李蕴斐Amber,Rookie |
+--------------+-----------------------------------+
-- 与聚合函数 COUNT 一起使用
SELECT mathpixtype,COUNT(nickname) FROM weixinuserinfo GROUP BY mathpixtype;
+--------------+-----------------+
| mathpixtype  | count(nickname) |
+--------------+-----------------+
| asciimath    |               1 |
| latex_styled |               1 |
| mathml       |              17 |
| text         |               4 |
+--------------+-----------------+
-- 与 HAVING 一起使用，选取条件限制输出结果
SELECT mathpixtype,COUNT(nickname) FROM weixinuserinfo GROUP BY mathpixtype HAVING COUNT(nickname)>3;
+-------------+-----------------+
| mathpixtype | count(nickname) |
+-------------+-----------------+
| mathml      |              17 |
| text        |               4 |
+-------------+-----------------+
-- 与 WITH ROLLUP 一起使用，最后加入一个总和行
SELECT mathpixtype,COUNT(nickname) FROM weixinuserinfo GROUP BY mathpixtype WITH ROLLUP;
+--------------+-----------------+
| mathpixtype  | count(nickname) |
+--------------+-----------------+
| asciimath    |               1 |
| latex_styled |               1 |
| mathml       |              17 |
| text         |               4 |
| NULL         |              23 |
+--------------+-----------------+

-- ###########################################################
-- ###################### 2. 聚合函数 #########################
-- ###########################################################
-- 查询表的记录数，不适合数据量大的数据库，解决办法可以直接看id最后一条
SELECT COUNT(*) FROM 表名;
-- 查询此列的和
SELECT SUM(字段名) FROM 表名;
-- 查询此列的平均值
SELECT AVG(字段名) FROM 表名;
-- 查询此列的最大值
SELECT MAX(字段名) FROM 表名;
-- 查询此列的最小值
SELECT MIN(字段名) FROM 表名;

-- ###########################################################
-- ###################### 3. 连接查询 #########################
-- ###########################################################
-- 内连接查询
SELECT s.name,m.mark from student as s ,          mark as m where    s.id=m.stu_id;
SELECT s.name,m.mark from student as s inner join mark as m where/on s.id=m.stu_id;
-- 左连接查询(先全列出左边的表，再以左边的表为基础，右边的表对应过去)
SELECT s.name,m.mark from student as s left join mark as m on s.id=m.stu_id;
-- 右连接查询
SELECT s.name,m.mark from student as s right join mark as m on s.id=m.stu_id;
-- 联合查询（字段数量一定要一致）
SELECT name from student union all select mark from mark;
-- 以下两句等价
SELECT * from student where id in(2,5);
SELECT * from student where id=2 union all SELECT * from student where id=5;
-- 子查询
SELECT * from student where id in(select stu_id from mark);
-- 分页查询 LIMIT
SELECT * FROM student limit 3; -- 限制查询条数为3
SELECT id FROM student order by id desc limit 1; -- 取最大id
SELECT * FROM student limit 3,2; -- 限制查询条数为从第三个开始取两个


-- 动态SQL中连接AND条件
where 1=1 是为了避免where 关键字后面的第一个词直接就是 “and”而导致语法错误。
```
##### DTL 事务控制语言
```sql
-- 关闭自动提交
set autocommit=0;

-- 开启事务
START TRANSACTION;
-- 如果开启事务后所有语句执行都没有问题
commit;
-- 如果有执行失败的语句
rollback; -- 记得set autocommit=0关闭自动提交
```