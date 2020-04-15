---
title: Redis Operation
date: 2019-04-11 20:01:01
categories:
    - NoSQL
tags:
    - Redis
description: "【redis学习系列——2】紧接着上篇的入门，本篇介绍了redis键的通用操作"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## Key的通用操作
```
1. 查看所有的key
> keys *
2. 模糊查询key：
> ?某些已知字符?
3. 删除key：
> del [key1] [key2]....
4. 判断是否存在：
> exists [key]
5. 重命名key：
> rename [existed key] [new key]
6. 设置过期的时间，单位秒：
> expire [key] [time]
7. 查看key所剩时间：
> ttl [key]，没有则返回-1
8. 获得指定key的类型：
> type [key]
9. flushall：
> 清空数据库
```