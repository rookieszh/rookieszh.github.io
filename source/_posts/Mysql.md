---
title: Mysql
date: 2019-03-02 09:34:57
categories:
    - SQL
tags:
    - Mysql
description: "What is Mysql? How to Use Mysql? 本篇介绍了mysql内置的数据类型、存储引擎等等"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

> 关系型数据库，开源
#### 命令行基本操作
```sql
-- 登陆mysql
$ mysql -h hostname -u username -p
```
#### 数据类型
##### 整数型
```sql
所有整数型默认是有符号数
tinyint:   1字节
smallint:  2字节
mediumint: 3字节
int:       4字节
bigint:    8字节
```
##### 小数型
```sql
float(m,d):   单精度，4字节，非精确数，通常不设定长度
double(m,d):  双精度，8字节，非精确数，通常不设定长度
decimal(m,d): 定点数，精确数，设定长度
-- m是数字总位数，d是小数点后面的位数，若省略，则根据硬件允许的限制来保存值
```
##### 字符串型
```sql
char:     0-255字节，定长字符串，存取速度快
varchar:  0-65535字节，变长字符串
tinytext: 0-255字节，短文本数据
text:     0-65535字节，文本数据
mediumtext: 0-(2^24-1)字节，中等长文本数据
longtext: 0-(2^32-1)字节，超长文本数据
set('value1','value2'...): 1、2、3、4或8个字节，取决于成员数目（最多64个成员）
enum('value1','value2'...): 1或2个字节，取决于枚举值数目（最多65535个值）
```
##### 日期时间型
```sql
year      1字节，YYYY
timestamp 4字节，YYYYMMDD HHMMSS
time      3字节，HH:MM:SS
date      3字节，YYYY-MM-DD
datetime  8字节，YYYY-MM-DD HH:MM:SS
-- 一般建议用int存储时间的 时间戳
-- 时间戳转换工具：http://tool.chinaz.com/Tools/unixtime.aspx
```
##### 二进制类型
```sql
-- 一般用来存放图片视频等数据
binary(m):    长度为0~m的定长二进制字符串
varbinary(m): 长度为0~m的变长二进制字符串
bit(m):       m位二进制数据，最多255个字节
tinyblob:     可变长二进制数据，最多255个字节
blob:         可变长二进制数据，最多(2^16-1)个字节
mediumblob:   可变长二进制数据，最多(2^24-1)个字节
longblob:     可变长二进制数据，最多(2^32-1)个字节
```
#### 存储引擎
最常用的数据库存储引擎：
1. MyISAM全文索引（full text），不支持事务，表级锁，崩溃恢复支持不好
2. InnoDB（5.6开始默认的数据库引擎），支持事务，不支持全文索引，5.6版本后开始支持，行级锁，数据量大的时候性能更好。推荐使用InnoDB，速度快。
3. BLACKHOLE
4. CSV
5. MEMORY
6. ARCHIVE
##### 数据库表引擎操作
```sql
-- 创建表时设定引擎
create table 表名(字段名 字段类型...) ENGINE=InnoDB;
-- 修改表引擎
alter table 表名 engine=目标引擎;
-- 查看数据表的引擎和所有信息
show table status\G;
```
#### 字符集编码
```sql
-- 查看数据库字符集
show create database 数据库名;
-- 创建数据库时设置字符规则
create database 数据库名 character set gbk;
-- 创建表时设置字符规则
create table 表名(字段名 字段属性...) charset=gbk;
-- 修改表字符规则
alter table 表名 charset=utf8;
-- 字符集校对
-- 校对规则是定义了比较字符串的方式，解决排序和字符分组的问题
-- 使用 utf8_general_ci 不区分大小写
create table 表名(字段名 字段属性...) collate=utf8_general_ci;
```
#### 字节字符长度
```sql
-- gbk：2字节
-- utf8：3字节
-- 英文：1字节 为 1字符
-- 汉字：2字节 为 1字符

-- int(n)中的n只有在zerofill的情况下才有用，否则存储数据不受n的位数限制
-- char(n)中的n为字符长度，上限255个字母或汉字。
-- varchar(n)同char。

-- 三种长度模式：
set sql_mode =  * ; -- 默认是宽松模式
-- ANSI 宽松模式 （数据格式不对时warning）
-- STRICT_TRANS_TABLES （数据格式不对时直接error）
-- TRADITIONAL （对所有的事务存储引擎，非事物存储引擎检查：
-- 日期类型中的月和日部分不能包含0，不能有0这样的日期0000-00-00，
-- 数据不能除0，进制grant自动创建新用户等一些校验）
```
#### 数据导入导出
```sql
-- 导出
mysqld -u用户名 -p密码 数据库名 表名 >保存文件名.sql
-- 导入
mysql -u用户名 -p密码 数据库名 < 要导入的文件名.sql
```