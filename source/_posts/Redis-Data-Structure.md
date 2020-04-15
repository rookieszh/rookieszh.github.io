---
title: Redis Data Structure
date: 2019-04-12 09:21:47
categories:
    - NoSQL
tags:
    - Redis
description: "【redis学习系列——3】本文重点介绍redis的数据结构与常用命令语句"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---


## Redis数据结构
### 字符串 String
> 二进制安全的，存入和获取的数据相同。也就是说 redis 的 string 可以包含任何数据。比如 jpg 图片或者序列化的对象。
> Value最多可以容纳的数据长度是512MB
> 相当于java中的map

#### 常用语句
```
1. 赋值
set [key] [value]
2. 取值
get [key] [value]
3. 取赋值
getset [key] [value]
先把key中的值显示出来，再把新的value值赋进去
4. 删值
del [key]
5. 自增值
incr [key]
如果key不存在，则创建且赋值1；如果key存在，数值型则+1，否则报错
6. 自减值
decr [key]
如果key不存在，则创建且赋值-1；如果key存在，数值型则-1，否则报错
7. 指定数值加减
incrby/decrby [key] [具体数字]  同上
8. 字符串拼接
append [key] [具体字符，可以数字]
数字拼接数字还是数字
```


### 哈希 Hash
> String Key 和 String Value 的map容器。可以将其想成一个 hash 对应着多个 string。
> 每一个hash可以存储4294967295个键值对

#### 与 String 的区别
string 是 一个 key - value 键值对，而 hash 是多个 key - value 键值对。
![](Redis2.png)

#### 常用语句
```
1. 单次赋值
hset [hashkey] [key] [value]
hmset [hashkey] [key1] [value1] [key2] [value2]....
2. 取值
hget [hashkey] [key]
hmget [hashkey] [key1] [key2]....
hgetall [hashkey]
3. 删值
hdel [hashkey] [key1] [key2]...
del [hashkey]
4. 指定数值加减
hincrby [hashkey] [key] [具体数字]
同上述命令
5. 判断键值对是否存在
hexists myhash username
6. 一个哈希中有几个属性
hlen [hashkey]
7. 一个hash中所有的key
hkeys [hashkey]
8. 一个hash中所有的value值
hvals [hashkey]
```

### 字符串列表 List
> ArrayList使用数组方式【Java】
> LinkedList使用双向链接方式【Redis】。即按照插入顺序排序。我们可以往列表的左边或者右边添加元素。
> list 内的元素是可重复的。

#### 常用语句
```
1. 从左侧依次插入value值，若链表不存在则自动创建
lpush [listkey] [value1] [value2]...
仅当链表存在时才会在头部插入
lpushx [listkey] [value]
2. 从右侧依次插入value值，若链表不存在则自动创建
rpush [listkey] [value1] [value2]...
仅当链表存在时才会在尾部插入
rpushx [listkey] [value]
3. 指定起始终点位置（0开始）列出表中元素，-1代表倒数第一个，-2倒数第二个
lrange [listkey] [start] [end]
4. 头部弹出一个value
lpop [listkey]
5. 尾部弹出一个value
rpop [listkey]
6. 查看链表长度
llen mylist
7. 从头到尾删除掉count个value，count为负时则从后往前删除，count为0时则删除所有的value
lrem [listkey] [count] [value]
8. 在第location个角标处设置value值
lset [listkey] [location] [value]
9. 在指定的第一个value之前/后插入值
linsert [listkey] before/after [exist value] [new value]
10. 将第一个链表的最后一个值弹出并压入第二个链表的头部
rpoplpush [listkey1] [listkey2]

lpush的程序称为**生产者**；
lpop的程序称为**消费者**；
rpoplpush是为了备份，避免消费者程序崩溃导致的数据丢失。
```

### 字符串集合 set
> 无序。
> 不允许出现重复的元素。

> 集合是通过哈希表实现的，因此添加、删除、查找的复杂度都是 O(1)。也可以确保id的唯一，跟踪一些唯一性数据，同时可以得出两个集合中的同一id（比如哪个顾客同时购买了这两个商品），用于维护数据对象之间的关联关系。
> 能完成多个服务器端的聚合操作。

> 可包含的最大元素数量是4294967295。

>redis 的 set 是一个 key 对应着 多个字符串类型的 value，也是一个字符串类型的集合

#### 常用命令
```
1. 添加元素，不能添加重复项
sadd [setkey] [value1] [value2]....
2. 删除元素
srem [setkey] [value1] [value2]....
3. 列出set中所有元素
smembers [setkey]
4. 判断set中是否有value，若有返回1，无返回0
sismember [setkey] [value]
5. 差集运算，返回两个set集合的差集，即返回1中有但2中没有的元素
sdiff [setkey1] [setkey2]
6. 交集运算
sinter [setkey1] [setkey2]
7. 并集运算
sunion [setkey1] [setkey2]
8. 得到set集合中元素个数
scard [setkey]
9. 随机返回一个set集合中的元素
srandmenber [setkey]
10. 将两个集合相差的成员存储到一个其他的集合上
sdiffstore [new setkey] [setkey1] [setkey2]
11. 将两个集合相交的成员存储到一个其他的集合上
sinterstore [new setkey] [setkey1] [setkey2]
12. 将两个集合的并集存储到一个其他的集合上
sunionstore [new setkey] [setkey1] [setkey2]
```

### 有序字符串集合 Zset
> zset 每个元素都会关联一个 double 类型的分数。redis 通过分数来为集合中的成员进行从小到大的排序。
zset 的元素是唯一的，但是分数（score）却可以重复。
PS : 游戏的排名，微博的热点话题，构建索引数据等等。

![](Redis3.png)
#### 常用命令
```
1. 添加元素，若已存在元素，则会用新的分数去替换原有的分数
zadd [sortkey] [分数1] [value1] [分数2] [value2]...
2. 获得元素的分数
zscore [sortkey] [value]
3. 获取元素数量
zcard [sortkey]
4. 删除sortset集合中的元素
zrem [sortkey] [value1] [value2]
5. 指定位置查询元素
zrange [sortkey] [start] [end]
zrange [sortkey] [start] [end] withscores 由大到小
zrevrange [sortkey] [start] [end] withscores 由小到大
6. 按照范围删除元素
zremrangebyrank [sortkey] [start] [end]
7. 按照分数删除元素
zremrangebyscore [sortkey] [start score] [end score]
zrangebyscore [sortkey] [start] [end] withscores limit [start location] [count]
8. 对元素value加count
zincrby [sortkey] [count] [value]
9. 两个分数之间的计数
zcout [sortkey] [start score] [end score]
```
