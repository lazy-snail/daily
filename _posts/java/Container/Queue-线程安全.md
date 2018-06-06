---
title: Queue
date: 2018-06-06 14:03:07
categories: java
tags: [java, 容器]
---
[toc]
这里介绍 Queue 容器中实现了同步方法的 LinkedBlockingQueue、ConcurrentLinkedQueue、LinkedBlockingDeque、ConcurrentLinkedDeque

# LinkedBlockingQueue 阻塞队列
```java
public class LinkedBlockingDeque<E>
    extends AbstractQueue<E>
    implements BlockingDeque<E>, java.io.Serializable {}
```
优先队列，不允许 null，对象必须实现了 Comparator 接口，即可比较的，排序方式为自然排序（默认）或用户指定的 Comparator 排序方法实现，可以指定初始大小（Object 数组）。

## 








# ConcurrentLinkedQueue 并发队列
```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {}
```