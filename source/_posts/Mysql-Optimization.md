---
title: Mysql Optimization
date: 2019-03-13 15:22:04
categories:
    - SQL
tags:
    - Mysql
    - Optimization
description: "本文将继续进阶Mysql——优化Mysql"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

#### 开启慢查询并分析
```sql
-- 打开慢语句日志记录
show variables like '%slow%'; -- 找到 slow_query_log
set global slow_query_log=on; -- 打开
show variables like '%long%'; -- 找到 long_query_time，单位为秒，默认10秒
set global long_query_time=1; --改小此值（你目标的查询速度）
explain 查询得很慢的那条语句;

show variables like '%profiling%';
set profiling=ON;
show profiles; -- 就能看到所有之前执行语句的执行时间
show profile for query Query_ID; -- 就能看到执行那条语句所有操作以及对应的时间；然后就能有针对的优化了
```
#### 缓存
```sql
-- 找到缓存
show variables like ‘%cache%’; -- 找到 query_cache_size 和 query_cache_type
set global query_cache_size = 1024000; -- 然后给size附一定的值
```