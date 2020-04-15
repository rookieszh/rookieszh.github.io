---
title: Avoid Accident of Redis
date: 2019-04-25 19:41:23
categories:
    - NoSQL
tags:
    - Redis
    - BigData
description: "【redis学习系列——7】如何避免在海量数据访问中产生事故"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---


#### 事故产生
用户token缓存采用了【user_token:userid】格式的key，保存用户的token的值，运维为了帮助开发小伙伴们查一下线上现在有多少登录用户，直接用了 keys user_token* 方式进行查询，事故就此发生了。导致redis不可用，假死。

#### 分析原因
我们线上的登录用户有几百万，数据量比较多；keys算法是遍历算法，复杂度是O(n)，也就是数据越多，时间复杂度越高。
数据量达到几百万，keys这个指令就会导致 Redis 服务卡顿，因为Redis 是单线程程序，顺序执行所有指令，其它指令必须等到当前的 keys 指令执行完了才可以继续。

#### 解决方案
我们可以采用redis的另一个命令scan。我们看一下scan的特点

> 1、复杂度虽然也是 O(n)，但是它是通过游标分步进行的，不会阻塞线程
2、提供 count 参数，不是结果数量，是redis单次遍历字典槽位数量(约等于)
3、同 keys 一样，它也提供模式匹配功能;
4、服务器不需要为游标保存状态，游标的唯一状态就是 scan 返回给客户端的游标整数;
5、返回的结果可能会有重复，需要客户端去重复，这点非常重要;
6、单次返回的结果是空的并不意味着遍历结束，而要看返回的游标值是否为零。

##### scan命令格式与解释
```
SCAN cursor [MATCH pattern] [COUNT count]

scan 游标 MATCH <返回和给定模式相匹配的元素> count 每次迭代所返回的元素数量
SCAN命令是增量的循环，每次调用只会返回一小部分的元素。所以不会让redis假死 SCAN命令返回的是一个游标，从0开始遍历，到0结束遍历
```

##### 举例
```
redis > scan 0 match user_token* count 5
 1) "6"
 2) "user_token:1000"
 2) "user_token:1001"
 3) "user_token:1010"
 4) "user_token:2300"
 5) "user_token:1389"
```
从0开始遍历，返回了游标6，又返回了数据，继续scan遍历，就要从6开始
```
redis > scan 6 match user_token* count 5
 1) "10"
 2) 1) "user_token:3100"
 2) "user_token:1201"
 3) "user_token:1410"
 4) "user_token:5300"
 5) "user_token:3389"
```
