---
title: Queue
date: 2018-06-06 14:03:07
categories: java
tags: [java, 容器]
---
[toc]
# Queue 接口 & Deque 接口
Queue 接口，提供队列操作的典型方法：
```java
public interface Queue<E> extends Collection<E> {
    boolean add(E e);
    boolean offer(E e);
    E remove();
    E poll();
    E element();
    E peek(); }
```

Deque 接口，提供双向队列操作的典型方法（去除和 Queue 重合的部分）：
```java
public interface Deque<E> extends Queue<E> {
    void addFirst(E e);
    void addLast(E e);
    boolean offerFirst(E e);
    boolean offerLast(E e);
    E removeFirst();
    E removeLast();
    E pollFirst();
    E pollLast();
    E getFirst();
    E getLast();
    E peekFirst();
    E peekLast();
    boolean removeFirstOccurrence(Object o);
    boolean removeLastOccurrence(Object o);
    boolean contains(Object o);
    int size();
    Iterator<E> iterator();
    Iterator<E> descendingIterator(); }
```

这里介绍 Queue 容器中没有同步方法的实现：ArrayDeque、PriorityQueue、LinkedList。

# ArrayDeque
```java
public class ArrayDeque<E> extends AbstractCollection<E>
    implements Deque<E>, Cloneable, Serializable {
        transient Object[] elements;
    }
```
从命名可知，底层是使用数组实现的。ArrayDeque 和LinkedList 是 Deque 的两个通用实现，官方更推荐使用 AarryDeque 用作栈和队列。
ArrayDeque 使用循环数组实现：
{% asset_img ArrayDeque.png 循环数组实现 %}


# PriorityQueue
```java
public class PriorityQueue<E> extends AbstractQueue<E>
    implements java.io.Serializable {
        transient Object[] queue; 
    ... }
```
优先队列，不允许 null，对象必须实现了 Comparator 接口，即可比较的，排序方式为自然排序（默认）或用户指定的 Comparator 排序方法实现，可以指定初始大小（Object 数组）。

## 堆排
PriorityQueue 通过数组表示的二叉小顶堆实现排序，可以用一棵完全二叉树表示，父节点和子节点的编号是有联系的：
leftNo = parentNo*2+1
rightNo = parentNo*2+2
parentNo = (nodeNo-1)/2
通过上浮、下沉（参见 [Algs4: Priority Queues](https://algs4.cs.princeton.edu/24pq)中的堆排序实现），维持二叉堆的有序性。

## 扩容
既然底层是使用数组实现的，那么就存在容量耗尽的情况，此时的扩容策略和一般的实现无异：先申请新数组，再拷贝旧数据，完成新数组指向。

# LinkedList
除 Deque，它也实现了 List 接口，在此不再重复。