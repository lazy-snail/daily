---
title: Set
date: 2018-06-06 12:29:43
categories: java
tags: [java, 容器]
---
[toc]
# Set 接口


这里介绍 Set 容器中没有同步方法的实现：HashSet、LinkedHashSet、TreeSet
# HashSet
```java
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
        private transient HashMap<E,Object> map;
        ... }
```
从源码可见，HashSet 是基于 HashMap 实现的，即其底层是一个 HashMap，HashSet 在执行构造方法时就会初始化一个 HashMap，用以保存元素。这使得 HashSet 的实现变得非常简单，很多方法的实现只需要借助 HashMap 的相应方法加以修改即可。


# LinkedHashSet
```java
public class LinkedHashSet<E> extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {}
```
继承自 HashSet，基于 HashMap 和 双向链表实现，其内部使用一个 LinkedHashMap，额外提供重要的功能就是有序遍历——按照插入顺序遍历元素。可以类比 HashMap 和 LinkedHashMap。
构造方法（该方法定义在 HashSet，供 LinkedHashSet 子类构造方法调用，并设置 dummy 为 true :
```java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor); }
```

# TreeSet
```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {
        private transient NavigableMap<E,Object> m;
    ... }
```
即使不看源码，从以上两个 Set 集合的实现，也可以推导出：TreeSet，底层应该是使用了 TreeMap 实现的。事实上也是如此，红黑树的结构在不考虑多线程的情况下表现良好，JDK1.8 中没有对该数据结构实现作修改。