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

## HashTable 
线程安全的 HashMap。Hashtable 继承自 Dictionary 虚类，而非 AbstractMap：
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
截止到 JDK10，HashTable（也包括 Vector、Satck）并没有被标注为 Deprecated，也就是它们仍然可以正常使用。问题在于，它对每一个操作都加了 synchronized 锁，这往往并非我们所希望的：通常，我们更希望同步整个序列，而非每个操作。并且同步每个操作也并不能保证安全性，仍然需要获取一个锁来避免并发操作，但性能和效率的降低却是显而易见的。
而后续补充的相应线程同步容器类则从侧面验证了，它们不是好的选择。


## ConcurrentHashMap 
线程安全的 HashMap。随着 JDK 的变迁，ConcurrentHashMap 从一开始的分段锁（JDK1.7 以及之前）技术转换到基于 CAS 实现（JDK1.8+）。以下以 CAS 实现为例：
线程安全、支持高效并发版本的 HashMap。其源码具体实现依赖于 java 内存模型，包括重排序、内存可见性（volatile 关键字）、happen-before（偏序关系）等。[ConcurrentHashMap演进](http://www.jasongj.com/java/concurrenthashmap)
```java
package java.util.concurrent;
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {}

// ConcurrentMap 接口定义如下
public interface ConcurrentMap<K,V> extends Map<K,V> {}
```

### 数据结构
JDK1.7之前通过分段锁技术使得理论并发数量等于其 ConcurrentHashMap 类对象实例的 Segment（相当于一个小型哈希表） 个数。从 JDK1.8 开始为进一步提高并发性，摒弃了分段锁的解决方案，直接使用一个大的数组，同样考虑哈希碰撞将长度超过阈值的桶链表转换为红黑树：
{% asset_img concurrenthashmap_java8.png ConcurrentHashMap数据结构 %}

### 寻址方式
同样（类比 HashMap、HashTable、旧版本的该类实现）是通过 Key 的哈希值与数组长度取模确定该 Key 在数组中的索引。同样为了避免不太好的 Key 的 hashCode 设计，它通过以下方法计算得到 Key 的最终哈希值。不同的是，JDK1.8+ 的 ConcurrentHashMap 作者认为引入红黑树转化策略后，即使哈希冲突比较严重，寻址效率也足够高，所以并未在哈希值计算上做过多设计，而只将 Key 的 hashCode 与其高 16 位作异或（XOR）并保证最高位为 0（从而保证最终结果为正整数）：
```java
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS; }

```

### 同步方式
put 操作，如果 Key 对应的数组元素为 null，则通过 CAS 操作将其设置为当前值；否则，对该元素使用 synchronized 关键字申请锁，成功后再进行操作。操作完成后判断是否需要将该处的链表转换为红黑树。
get 操作，由于数组是被 volatile 关键字修饰的（见上），因此无需担心数组的可见性问题。同时每个元素是一个 Node 实例（JDK1.7 每个元素是一个 hashEntry），它的 Key 值和 hash 值都由 final 修饰，不可修改，故也无需担心它们被修改的可见性问题。而 Value 及对下一个元素的引用由 volatile 修饰，也能保证可见性：
```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
    ... }
```

Key 对应的数组元素的可见性，由 Unsafe 的 getObjectVolatile 方法保证:
```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectAcquire(tab, ((long)i << ASHIFT) + ABASE); }
```

### size() 操作
put 和 remove 方法都会通过 addCount 方法维护 Map 的 size。size 方法通过 sumCount 获取由 addCount 方法维护的 Map 的 size。

### 分段锁技术（JDK1.7）
采用分段锁技术实现的 ConcurrentHashMap 结构：
{% asset_img ConcurrentHashMap.jpg ConcurrentHashMap结构 %}


## ConcurrentSkipListMap 
线程安全的 TreeMap。线程安全的有序的 Map。底层数据结构使用跳表——在并发场景下，它的性能优于红黑树，实现上也简单得多。
```java
package java.util.concurrent;
public class ConcurrentSkipListMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentNavigableMap<K,V>, Cloneable, Serializable {}
```

## CAS 构建并发安全性
CAS 属于底层硬件（CPU）层面的技术实现。ConcurrentSkipListMap 使用 CAS 技术构建并发安全性。
[ConcurrentSkipListMap 源码分析](https://www.jianshu.com/p/edc2fd149255)