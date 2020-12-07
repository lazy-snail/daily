---
title: Set-线程安全
date: 2018-06-06 12:29:43
categories: java
tags: [java, 容器]
---
[toc]
这里介绍 Set 容器中实现了同步方法的 CopyOnWriteArraySet、ConcurrentSkipListSet。
# CopyOnWriteArraySet
```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
            private final CopyOnWriteArrayList<E> al;
    ... }
```
看到这个内部 CopyOnWriteArrayList，显然，它的底层数据结构使用了基于 COW 技术的 ArrayList。不再展开。

# ConcurrentSkipListSet
```java
public class ConcurrentSkipListSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {
        private final ConcurrentNavigableMap<E,Object> m;
    ... }
```
显然，底层是一个 ConcurrentSkipListMap，跳表 + CAS，提供简洁高效的线程安全实现。