---
title: Java-lang
date: 2018-06-03 15:15:38
categories:
tags:
---
[toc]
# 关键字
一些关键字概况。

## transient



# 技术实现
一些语言技术实现。

## COW 优化策略
COW（Copy-On-Write，写时复制）是一种用于程序设计中的优化策略：一开始大家都在共享同一个内容，当某个人（线程）想要修改这个内容的时候，才会真正把内容拷贝并形成一个新的内容然后再修改。这是一种延时懒惰策略，主要应用于多并发场景。
JDK1.5 引入了两个该机制的实现类：CopyOnWriteArrayList、CopyOnWriteArraySet。