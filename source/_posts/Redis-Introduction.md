---
title: Redis Introduction
date: 2019-04-11 11:11:11
categories:
    - NoSQL
tags:
    - Redis
description: "【redis学习系列——1】本文拉开了NoSQL的第一个篇章：redis入门介绍，缓存型 key-value 数据库"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## Redis概述
> Redis 是完全开源免费的，遵守 BSD 协议，是一个高性能的 key - value 数据库
> Redis 的所有操作都是原子性的

## 应用场景
1. 高速缓冲
2. 任务队列
3. 应用排行榜
4. 网站访问统计
5. 数据过期处理
6. 分布式集群架构中的session分离

## Redis 支持的五类键值数据类型：
1. 字符串类型String
2. 列表类型List
3. 集合类型Set
4. 有序集合类型Zset
5. 散列类型Hash

## Redis安装
1. 首先下载最新版本的redis [下载](https://redis.io/download)
> wget http://download.redis.io/releases/redis-4.0.6.tar.gz
2. 在路径下解压
> tar xzf redis-4.0.6.tar.gz
3. 进入文件夹进行编译（此文件夹即为安装路径）
> cd redis-4.0.6<br>
> make
4. 执行安装
> make install
5. 查看安装版本
> redis-server -v
6. 修改端口和密码
> 安装目录下 vi redis.conf<br>
> daemonize yes<br>
> bind 0.0.0.0
> protected-mode no<br>
> port 6060<br>
> requiredpass Kirito
7. 启动服务
> redis-server redis.conf
8. 关闭服务
> ps aux|grep redis<br>
> kill -9 进程号<br>
> 或者进入终端执行命令 shutdown<br>
> 或者 redis-cli shutdown
9. 新开终端
> redis-cli -p 6060 -a Kirito
10. 阿里云安全配置打开端口

## Redis基本操作
* 验证在工作
ping：返回pang
* 存放数据
set [key] [value]
* 获取数据
get [key] [value]
* 删除数据
del [key] [value]
* 查看所有key
keys *
* 退出
exit

