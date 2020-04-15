---
title: Redis Persistence
date: 2019-04-20 10:31:22
categories:
    - NoSQL
tags:
    - Redis
description: "【redis学习系列——6】本文为redis的进阶篇，持久化机制"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---


## 持久化
> 持久化：将内存中的数据存储到硬盘中

### RDB方式
> 在指定的时间间隔内，将内存中的数据集写入到磁盘上。

#### 优势
整个文件库只包含一个文件，对于文件备份很好；对性能损耗小。

#### 劣势
有可能数据没来得及备份系统就宕机了，当数据集非常大时，很有可能导致系统停止工作几百毫秒甚至1秒

#### 配置
> 在redis的配置文件redis.conf中的 **SNAPSHOTTING**

常用配置项：

1. **save [seconds] [key nums]：** 每seconds秒若有keynums次key value的变化，则完成一次持久化
2. **stop-writes-on-bgsave-error：** 默认值为yes。当启用了RDB且最后一次后台保存数据失败，Redis是否停止接收数据。这会让用户意识到数据没有正确持久化到磁盘上，否则没有人会注意到灾难（disaster）发生了。如果Redis重启了，那么又可以重新开始接收数据了。
3. **rdbcompression：** 默认值是yes。对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。
4. **rdbchecksum：** 默认值是yes。在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。
5. **dbfilename [filename]** 备份名，默认dump.rdb
6. **dir ./** 保存路径

### AOF方式
> 用日志的形式记录服务器所处理的每一个操作。

#### 优势
可以带来更高的数据安全性，日志写入过程中如果出现宕机情况也不会破坏掉日志文件本身。

#### 劣势
效率最低。

#### 配置
> 在redis的配置文件redis.conf中的 **APPEND ONLY MODE**

常用配置项：
1. appendonly no→yes
2. appendfilename "appendonly.aof"
3. appendfsync always（每修改一次就同步）/everysec（每秒同步一次）/no（不同步）

eg.比如不小心flushall了，你只需要找到appendonly.aof日志文件，把最后一行flushall删掉即可，数据就回来了。

## 无持久化
就是把redis当成一种缓存的机制了

## 同时使用RDB和AOF
