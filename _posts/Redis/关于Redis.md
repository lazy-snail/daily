---
title: 关于Redis
date: 2018-05-30 09:14:03
categories: Redis
tags: [Redis, DB]
---
[toc]
## 基于 Key-Value 的 NoSQL 内存数据库
也称数据结构服务器。
特点和优势：
* 支持数据的持久化，可以将内存中的数据持久化到磁盘中，重启时再次加载使用；
* 不仅支持简单的 key-value 类型数据，同时还提供 list、set、zset、hash 等数据结构的存储；
* 支持数据备份，即 master-slave 模式的数据备份；
* 性能极高：读的速度是110000次/s,写的速度是81000次/s；
* 丰富的数据类型，提供了5种数据结构：String、List、Hash、Set、Ordered Set（ZSet）；
* 原子操作：Redis 的所有操作都是原子性的，同时还支持对几个操作全并后的原子性执行；
* 丰富的特性：支持 publish/subscribe、通知 key 过期等特性。


## 数据结构
{% asset_img 数据结构类型.JPG 5种数据结构类型 %}

### string
类似于其他编程语言中字符串的概念。
redis 中的 string 类型是二进制安全的，i.e. 可以包含任何数据，比如一张 .jpg 格式的图片。一个键最大存储量为 512 MB。
{% asset_img string.JPG 字符串 %}

### list
按照插入顺序有序存储多个字符串，相同元素可重复，双向操作（LPHSH、LPOP、RPUSH、RPOP）。
每个 list 最多存储元素数量：2^32 - 1。
{% asset_img list.JPG 列表 %}

### set
集合和列表都可以存储多个字符串，不同之处在于：列表可以存储多个相同的字符串，集合通过 **散列表来保证存储的每个字符串都是不相同的**。
redis 的集合使用无序（unordered）方式存储元素，不支持像列表一样将元素从某一端 push/pop 的操作，相应地，使用 SADD/SREM 添加/移除元素。由于是通过哈希表实现的，所以添加/移除/查找的时间复杂度为 O(1)。
每个 set 最多存储元素数量：2^32 - 1。
{% asset_img set.JPG 集合 %}

### hash
可以存储多个键值对之间的映射。
每个 hash 最多存储键值对数量：2^32 - 1。
{% asset_img hash.JPG 散列 %}

### zset
有序集合（zset）和散列一样，都用于存储键值对，不支持重复元素。不同之处在于：有序集合的键被称为成员（member），每个成员都是各不相同的；值被称为分值（score），必须为浮点数（分值可重复）。zset 既可以根据成员访问元素（和散列一样），又可以根据分值以及分值的排列顺序来访问元素的结构。
每个 zset 最多存储键值对数量：2^32 - 1。
{% asset_img zset.JPG 有序集合 %}


## HyperLogLog


## 发布/订阅
redis 支持发布/订阅的消息通信模式。


## 不完全（partial）事务支持