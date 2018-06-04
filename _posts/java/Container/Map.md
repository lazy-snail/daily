---
title: Map
date: 2018-06-03 19:27:12
categories: java
tags: [java, 容器]
---
[toc]
**实现了 Map 接口的常用容器类**
Map 可以看作是一种符号表，使用键值对的数据结构存储数据。
包括非线程安全的 HashMap、LinkedHashMap、TreeMap，线程安全的 ConcurrentHashMap、ConcurrentSkipListMap，线程安全但目前已不推荐使用的 HashTable。

## HashMap

### 实现
```java
package java.util;
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {}
```
其内部的 Node 结构：
```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        ...
}
```

### 特点
* 非线程安全的；
* 根据键的 hashCode 值存储数据。大多数情况下可以直接定位键值对，因而具有很快的访问速度，理论上时间复杂度为 O(1)；
* 遍历顺序是不确定的，也无法支持随机访问；
* 最多只允许一条记录的键为 null，允许多条记录的值为 null。

### hashCode 方法和加载因子
HashMap 基于哈希函数来存储键值对，即，由一个哈希函数决定每个键值对的存放位置，存放位置所使用的数据结构为一个数组，其初始容量为（DEFAULT_INITIAL_CAPACITY= 1 << 4），即 16，并且要求该值必须为 2 的幂，这与其底层数据结构有关。在不出现哈希碰撞（两个键值对的哈希结果相同）的情况下，即完美哈希时，访问时间复杂度即为 O(1)。HashMap 使用 **开散列** 方法来解决哈希冲突，即，**对应一个特定 hash 值的位置存储的是一个链表头，指向 hash 到同一个位置的多个键值对组成的链表**。这种实现的访问时间复杂度显然不是常数级，且随着键值对的增多，发生碰撞的情况会加剧。所以 HashMap 有一个加载因子（DEFAULT_LOAD_FACTOR = 0.75f），**用来控制当 HashMap 到达一定“装载程度”时执行 rehash（重哈希）操作，使得容量变为原来的约 2 倍**。注意，重哈希是个非常耗时的操作，所以有必要采取一些措施（比如预估并在初始化时调用指定初始数组容量的构造方法）来避免/减少。

### 重写 hashCode() 和 equal()
HashMap 的很多方法都是基于 hashCode() 和 equal() 的，前者用来定位，后者用来判断是否相等。很多情况下，equal 方法可能并不符合我们程序的逻辑——Object 的 equel 方法只是简单地判断是不是同一个实例，因此当我们认为判定 equals 应该是逻辑上的相等而不是仅仅判断是不是内存中中同一个东西的时候，就需要重写 equal()，此时，就必须重写 hashCode()。
重写 equal() 要使得其依然满足：
* 自反性
* 对称性
* 传递性
* 一致性

### 常用方法
put()、get()：根据数据结构特点可知，先利用 hashCode 定位到具体的链表，再在链表中遍历是否已存在，然后操作。

### 问题
#### 容量为什么必须是 2 的幂？
HashMap 中的数据结构是数组 + 单链表的组合。我们希望元素存放的更均匀，理想情况是，Entry 数组中每个位置都只有一个元素，这样，查询的时候效率最高，不需要遍历单链表，也不需要通过 equals() 去比较 K，而且空间利用率最大，时间复杂度最优。
而使得计算分布得更均匀，就是使用 % 运算，即取模：
哈希值 % 容量 = bucketIndex
当容量为 2 的幂时，可以（也是源码方法）写成：
h & (length - 1) ，
这与前式等价但不等效：后者位运算的效率更高（CPU 效率最慢的也就是除法和取余了）！ 

#### 加载因子怎么选择，为什么默认 0.75？
这是一种时间和空间的折中选择：加载因子过高虽然减少了空间开销，但同时也增加了查询成本；反之亦然。

[参考博客](http://capps.cn/EbQRVj "理解 HashMap")

## LinkedHashMap
### 实现
```java
package java.util;
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V> {}
```
其内部的 Entry 扩展了 HashMap 的 Node 并添加了两个指针：
```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```
{% asset_img LinkedHashMap示意图.png LinkedHashMap示意图 %}
将所有节点链入一个双向链表的 HashMap。继承自 HashMap，所以拥有 HashMap 的所有特性。比如，LinkedHashMap 的元素存取过程基本与 HashMap 基本类似，只是在细节实现上稍有不同。这是由 LinkedHashMap 本身的特性所决定的，因为它额外维护了一个双向链表用于 **保持迭代顺序**，并重写了HashMap 的迭代器，使用其维护的双向链表进行迭代输出，可以以插入顺序遍历元素——这是和无序的 HashMap 最大的区别。

### 存取
LinkedHashMap 的存取过程与 HashMap 基本类似，只是在细节实现上稍有不同，这是由 LinkedHashMap 本身的特性所决定的，因为它要额外维护一个双向链表用于保持迭代顺序。put 操作上，虽然 LinkedHashMap 完全继承了 HashMap 的 put 操作，但是在细节上还是做了一定的调整，比如，在 LinkedHashMap 中向哈希表中插入新 Entry 的同时，还会通过 Entry 的 addBefore 方法将其链入到双向链表中。在扩容操作上，虽然 LinkedHashMap 完全继承了 HashMap的resize 操作，但是鉴于性能和 LinkedHashMap 自身特点的考量，LinkedHashMap 对其中的重哈希过程(transfer方法)进行了重写。get 操作上，LinkedHashMap 重写了 HashMap 的 get 方法，通过 HashMap 的 getEntry 方法获取 Entry 对象，在此基础上，进一步获取指定键对应的值。

### LinkedHashMap 与 LRU
当使用 LinkedHashMap 实现 LRU 算法时，需要调用其第 5 个构造方法，将 accessOrder 置为 true，即，元素按访问顺序排序（默认 false 按插入顺序排序）。LinkedHashMap 重写了 HashMap 的 recordAccess 方法（HashMap中该方法为空），当调用父类的put 方法时，在发现 key 已经存在时，会调用该方法；当调用自己的 get 方法时，也会调用到该方法。该方法提供了 LRU 算法的实现，它将最近使用的 Entry 放到双向循环链表的尾部。即当 accessOrder 为 true 时，get 方法和 put 方法都会调用 recordAccess 方法使得最近使用的 Entry 移到双向链表的末尾；当 accessOrder 为默认值 false 时，从源码中可以看出 recordAccess 方法什么也不会做。
可以用以上方法快速构建 LRU 算法实现。

[参考博客](https://blog.csdn.net/justloveyou_/article/details/71713781 "理解 LinkedHashMap")
