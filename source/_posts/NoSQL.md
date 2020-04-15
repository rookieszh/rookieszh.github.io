---
title: NoSQL
date: 2019-04-01 08:01:03
categories:
    - NoSQL
tags:
    - NoSQL
description: "本文将开始引入NoSQL的概念，并与传统SQL进行对比"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## NoSQL概述
> NoSQL：Not Only SQL
> [官方手册](http://redisdoc.com)
> [教程](https://www.runoob.com/redis/redis-tutorial.html)

#### NoSQL特点：
1. High performance - 高并发读写
2. Huge Storage - 海量数据的高效率存储和访问
3. High Scalability && High Availability - 高可扩展性（因为去掉了关系型数据库其中的关系，使得数据之间没有关系了）和高可用性
4. 灵活的数据模型，无需事先建立字段

#### NoSQL数据库的四大分类：
1. **键值对（Key-Value）存储** 能快速查询，但数据缺少结构化
2. **列存储** 能快速查询，扩展性比较强，但功能相对于局限
3. **文档数据库** mangodb，数据结构要求不高，但查询性能不高，而且缺少统一查询的语法
4. **图形数据库** 一些社交网站所用，需要对整个图做计算才能得出结果，不容易做分布式的集群方案

## SQL与NOSQL
MySQL需要定义的结构化架构。
NoSQL允许“ 文档” 中任何数据的持久性。
MySQL有一个庞大的社区支持它。
NoSQL有一个小而快速增长的社区。
NoSQL具有易于扩展的特点。
MySQL需要更多托管可伸缩性。
MySQL利用SQL，它被用于多种数据库类型。
NoSQL是一个基于设计的数据库，具有流行的实现。
MySQL使用标准查询语言（SQL）。
NoSQL不使用标准查询语言。
MySQL有许多出色的报告工具。
NoSQL很少提供难以标准化的报告工具。
MySQL可以为大数据提供性能问题。
NoSQL在大数据方面表现出色。