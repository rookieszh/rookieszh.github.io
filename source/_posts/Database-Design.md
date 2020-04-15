---
title: Database Design
date: 2019-03-15 07:30:09
categories:
    - SQL
tags:
    - Mysql
    - DataBase Design
description: "本文将继续进阶Mysql——数据库设计"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

#### 第一范式（1NF）
原子性
即表的列的信息具有原子性，不可再分解。
只要数据库是关系型数据库，就自动满足1NF。
#### 第二范式（2NF）
满足第二范式必须先满足第一范式。
第二范式要求数据库表中的每个实例或行必须可以被唯一的区分。
通常我们通过设计主键来实现区分。
#### 第三范式（3NF）
满足第三范式必须先满足第二范式。
要求一个数据库表中不包含已在其他表中已包含的非主键字段。
为了满足第三范式往往会把一张表分成多张表。
范式不是绝对，可以违背，比如中间连接表，含有被连接表的主键uuid之类的。