---
title: Map-线程安全
date: 2018-06-03 19:27:12
categories: java
tags: [java, 容器]
---
[toc]
**实现了 Map 接口的常用容器类**
Map 可以看作是一种符号表，使用键值对的数据结构存储数据。
包括非线程安全的 HashMap、LinkedHashMap、TreeMap，线程安全的 ConcurrentHashMap、ConcurrentSkipListMap，线程安全但目前已不推荐使用的 HashTable。

## HashTable 线程安全的 HashMap
Hashtable 继承自 Dictionary 虚类，而非 AbstractMap：
```java
package java.util;
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {}
```
其使用方式上，除了支持多线程外，和 HashMap并没有明显区别。
HashTable 使用“拉链法”实现哈希表。几个重要参数：
* table：一个 Entry[] 数组类型，Entry 代表了“拉链”的节点，每一个 Entry 代表了一个键值对；
* count：HashTable 的大小，包含 Entry 键值对的数量，而不是 HashTable 容器大小；
* threshold：阈值，用于判断是否需要调整容量。threshold =“容量 * 加载因子”；
* loadFactor：加载因子。
* modCount：用来实现“fail-fast”机制（即快速失败）。所谓快速失败，就是在并发集合进行迭代操作时，若有其他线程对其进行结构性的修改，这时迭代器会立马感知到，并立即抛出 ConcurrentModificationException 异常，而不是等到迭代完成之后才告诉你（此时早已经出错了）。

### 线程安全的保证
很多方法使用了 synchronized 修饰，以锁的方式保证操作的同步和安全。这意味着，如果确定程序可以在单线程表现良好，那么不适合使用它（显然应该考虑 HashMap）：锁机制带来的效率降低是很显著的。

### 为什么不再推荐使用？
截止到 JDK10，HashTable（也包括 Vector、Satck）并没有被标注为 Deprecated，也就是它们仍然可以正常使用。问题在于，它对每一个操作都加了 synchronized 锁，这往往并非我们所希望的——通常，我们更希望同步整个序列，而非每个操作。并且同步每个操作也并不能保证安全性，仍然需要获取一个锁来避免并发操作，但性能和效率的降低却是显而易见的。
而后续补充的相应线程同步容器类则从侧面验证了，它们不是好的选择。


## ConcurrentHashMap
线程安全、支持高效并发版本的 HashMap。默认的理想状态下，可支持 16 个线程执行并发写操作及任意数量线程的读操作。其源码具体实现依赖于 java 内存模型，包括重排序、内存可见性（volatile 关键字相关）、happen-before（偏序关系）等。
```java
package java.util.concurrent;
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {}

// ConcurrentMap 接口定义如下
public interface ConcurrentMap<K,V> extends Map<K,V> {}
```

### Segment 和 

### 高效并发
不同于 HashTable 和由同步包装器包装的 HashMap (Collections.synchronizedMap(Map<K,V> m))需要全局锁来同步不同线程间的并发访问，ConcurrentHashMap 的并发读写操作对性能的影响更小：读操作（几乎）不需要加锁，写操作使用 **锁分段技术** 来实现只对所操作的数据段加锁而不影响对其他段的访问。
ConcurrentHashMap 本质上是一个 Segment 数组，一个 Segment 实例又包含若干个桶，每个桶中都包含一条由若干个 HashEntry 对象链接起来的链表：
{% asset_img ConcurrentHashMap.jpg ConcurrentHashMap结构 %}
其通过以下机制保证高效并发：
* 通过锁分段技术保证并发环境下的写操作；
* 通过 HashEntry 的不变性、Volatile 变量的内存可见性和加锁重读机制保证高效、安全的读操作；
* 通过不加锁和加锁两种方案控制跨段操作的的安全性。