---
title: java-CAS
date: 2020-04-07 16:19:54
categories: java
tags: [java, 并发]
---
[toc]

# CAS
## CAS 构建并发安全性
CAS 属于底层硬件（CPU）层面的技术实现。ConcurrentSkipListMap 使用 CAS 技术构建并发安全性。
[ConcurrentSkipListMap 源码分析](https://www.jianshu.com/p/edc2fd149255)