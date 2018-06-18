---
title: List
date: 2018-05-28 16:51:40
categories: java
tags: [java, 容器]
---
[toc]
# List 接口
List 接口中的元素是有序的、可重复的、可以为 null 的集合（序列）。
提供以下功能方法：
* 位置相关：List 和 数组一样，都是从 0 开始，可以根据元素在 list 中的位置进行操作，比如说 get, set, add, addAll, remove;
* 搜索：从 list 中查找某个对象的位置，比如 indexOf, lastIndexOf;
* 迭代：使用 Iterator 的拓展版迭代器 ListIterator 进行迭代操作;
* 范围性操作：使用 subList 方法对 list 进行任意范围的操作。

**List 相关类**
包括非线程安全的 ArrayList、LinkedList，线程安全的 CopyOnWriteArrayList，线程安全但目前已不推荐使用的 Vector 和其子类 Stack。

# ArrayList 非线程安全的动态数组
使用连续内存空间，容量动态增长。行为类似于 Arrays，但只能存放对象类型，而不能像 Arrays 可以支持基本数据类型。其类定义如下：
```java
java.lang.Object
   ↳     java.util.AbstractCollection<E>
         ↳     java.util.AbstractList<E>
               ↳     java.util.ArrayList<E>
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```
可见，其继承了 AbstractList 虚类，扩展了 List、RandomAccess、Cloneable、Serializable 4 个接口。显然，首先它是一个队列，并且支持 **随机访问元素** 和 **序列化**。i.e. **擅长随机读取元素，但在中间插入/移除元素比较慢**。

## 内存分配及扩容策略
默认的初始容量（DEFAULT_CAPACITY）为 10，默认增长方式为 1.5 倍率扩容：
1. 申请新的连续内存空间；
2. 将数据拷贝到新内存区域。

## 三种遍历元素的方式
### 迭代器
对于 List、String 等类型，由于实现了 Iterator 接口（事实上是实现了 Iterable 接口），需要保证要遍历的对象非空，判断是否还有下一个元素：
```java
Iterator<Integer> it = arrayList.iterator();
while(it.hasNext()) 
    System.out.print(it.next() + " ");
```

### 索引（for 循环）
由于扩展了 RandomAccess 接口，有实例方法 arrayList.size()，可以获取元素数量，并且，list 接口的子类都按插入顺序排列元素，i.e. 可以以插入顺序取得所有元素，支持类似下标的索引操作：
```java
for(int i = 0; i < arrayList.size(); i++)
   System.out.print(arrayList.get(i) + " ");
```

### foreach 
是的，它是一种语法糖实现：对于 List 类型，它们能使用 foreach 的基础是因为实现了 Iterator 接口。即，javac 会将这种遍历展开成迭代器遍历：
```java
for(Integer number : arrayList)
   System.out.print(number + " ");
```

注意，**效率方面：索引（for 循环）遍历是最高的（也要根据元素的数据类型具体分析，并非绝对）**，而另外两种本质上是一样的，并不好作判断。至于 for 循环遍历的效率高，主要原因是 RandomAccess 接口的功劳，其描述文档中有：
{% asset_img 遍历效率对比.JPG 遍历效率对比 %}

可以看出，RandomAccess 接口带来的优势：
* 可以快速随机访问集合；
* 使用快速随机访问（for 循环）效率可以高于 Iterator。


# LinkedList 双向链表
```java
public class LinkedList<E> extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable {}
```
LinkedList 与 ArrayList 一样实现了 List 接口。不同在于 ArrayList 是 List 接口的大小可变数组的实现，LinkedList 是 List 接口链表的实现。基于链表实现的方式使得 LinkedList 在插入和删除时更优于 ArrayList，而随机访问则比 ArrayList 逊色些。同时实现了 Deque 接口，所以可以提供双向队列的典型方法：pop、peek、push 等。

## 方法
**get、set**
既然实现了 List 接口，也就能够进行类似下标访问的 get、set 等方法。
实际原理非常简单，它就是通过一个计数索引值来实现的。如，当调用 get(int location) 时，首先会比较“location”和“双向链表长度的1/2”；若前者大，则从链表头开始往后查找，直到 location 位置；否则，从链表末尾开始先前查找，直到 location 位置。

**Deque 接口提供的方法**
包括 push()、peek*()、poll*()、pop() 等。

# CopyOnWriteArrayList 线程安全的 ArrayList
JDK1.5 新增数据结构，使用 COW 实现的线程安全数组。doc 简介：
A thread-safe variant of ArrayList in which all mutative operations (add, set, and so on) are implemented by making a fresh copy of the underlying array.
i.e. 所有可变操作都是对底层数组进行一次新的复制实现的。
其类定义如下：
```java
package java.util.concurrent;
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```

具体线程安全保证见 COW 技术实现。

# Vector 线程安全的动态数组
其类定义如下：
```java
java.lang.Object
   ↳     java.util.AbstractCollection<E>
         ↳     java.util.AbstractList<E>
               ↳     java.util.Vector<E>
public class Vector<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```

整体上，Vector 可以看作线程安全版本的 ArrayList，但两者有些区别：
* Vector 是同步访问的；
* Vector 包含了许多传统的方法，这些方法不属于集合框架。

## 线程安全保证
大量方法使用 synchronized 关键字修饰，以加锁的方式确保线程安全。

## 内存分配及扩容策略
默认的初始容量（DEFAULT_CAPACITY）为 10， 默认增长方式为 2 倍率扩容：
1. 申请新的连续内存空间；
2. 将数据拷贝到新内存区域。

## 四种遍历元素的方式
相比于 ArrayList，多出了一种 Enumeration 遍历方式：_这就是上述的“传统方法”之一，也是目前不再推荐使用 Vector 等一些老旧的类的原因。_
```java
Integer value = null;
Enumeration enu = vec.elements();
while (enu.hasMoreElements()) {
    value = (Integer)enu.nextElement();
}
```
四种遍历方式的效率也和 ArrayList 类似，for 循环最快。

## 为什么不再推荐使用？
根据以下的解释，通常我们是想同步整个序列，而提供的方法是同步单个操作，这并不安全，仍然需要获得一个锁来避免并发修改，并且其方法的效率较低。
https://stackoverflow.com/questions/1386275/why-is-java-vector-and-stack-class-considered-obsolete-or-deprecated


# Stack 继承自 Vector 的栈实现
其源码实现比较简单，栈的几个典型方法通过对 Vector 的一些方法修改得到。也是线程安全的。