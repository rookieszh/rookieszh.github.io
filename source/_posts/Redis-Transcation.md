---
title: Redis Transcation
date: 2019-04-13 12:08:12
categories:
    - NoSQL
tags:
    - Redis
description: "【redis学习系列——4】和mysql一样，redis也有事务，本文将进行简要介绍"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 特性与事务

### 多数据库
一个Redis实例最多可以提供16个数据库，下标分别是0~15，客户端默认的是0号数据库
1. select [db num]即可选择几号数据库
2. move [key] [db num]；将key移至另一个数据库，剪贴操作

### 事物
事物中所有命令都会被串行化顺序去执行。
事务执行期间redis不会再为其他客户端提供服务，以保证事物原子化执行（即事物期间的操作对其他客户端不会造成影响）
事物中一个命令执行失败，后面的命令还是会顺序执行。
1. multi 开启事务
2. exec 提交事务
3. discard 撤销事务（之前所有的操作）
