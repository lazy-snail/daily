---
title: Java-集合类
date: 2018-05-31 10:37:54
categories: Java
tags: [java, 集合类]
---
[toc]
## 集合类
java 集合类图谱：
{% asset_img Java集合.png Java集合类族谱 %}
可以看出，java 的集合类主要由两个接口派生而出：Collection 和 Map，这两个接口又包含了一些接口或实现类。

## Collection 接口
一个 Collection 代表一组 Object，即 Collection 的元素（Elements）。一些 Collection 允许相同的元素而另一些不行，一些能排序而另一些不行。JDK 不提供 Collection 的直接实现，而是继承自 Collection “子接口”如 Set、List、Queue 的类。
**Collections 是一个集合框架的帮助类，里面包含一些对集合的排序，搜索以及序列化的操作**。

## Iterable 接口：
Collection 接口扩展了 Iterable 接口，后者的 Iterator() 方法返回一个迭代器 Iterator（一个接口），为集合框架提供 **遍历所有元素** 的方法。Iterator 模式把访问逻辑从不同的集合类中抽象出来，从而避免向客户端暴露集合的内部结构。每种集合类返回的 Iterator 具体类型可能不同，但它们都实现了 Iterator 接口，因此，不必关心到底是哪种 Iterator，它只需要获得这个 Iterator 接口即可。
迭代器取代了 Java 集合框架中的 Enumeration（历史遗留类），Iterator 更加安全，因为当一个集合正在被遍历的时候，它会阻止其它线程去修改集合（以抛异常的方式）。要确保遍历过程顺利完成，必须保证遍历过程中不更改集合的内容（Iterator 的 remove() 方法除外）。所以，确保遍历可靠的原则是：**只在一个线程中使用这个集合，或者在多线程中对遍历代码进行同步**。
### Iterator
显然，一个实现了 Iterable 接口的类必然实现了 Iterator 接口——作为 Iterable 的一个方法。反之则不成立。
Iterator接口提供的三种方法： 
* boolean hasNext(): 返回集合里的下一个元素；
* Object next(): 返回集合里下一个元素；
* void remove(): 删除集合里上一次next方法返回的元素。 

## Set 接口
Set 是一种不包含重复元素的 Collection，i.e. 任意两个元素都有 e1.equals(e2) == false（也最多只有一个 null 元素）。这里就需要注意，必须要小心操作可变对象（Mutable Object）：如果一个 Set 中的可变元素改变了自身状态而导致 Object1.equals(Object2) == true，将引发一些问题。

实现 Set 接口的常用类有：
### HashSet
采用散列存储方式，**为快速查找设计的 Set，无序**——不保证元素插入的顺序和输出顺序一致。**非线程安全**

### LinkedHashSet：
具有 HashSet 的查询速度优势，内部使用链表维护元素的顺序（插入次序），所以使用迭代器时会按照元素的插入顺序遍历。**非线程安全**

### TreeSet
使用红黑树结构实现，维持集合中的元素有序，相应地时间复杂度损失：添加、删除、包含的算法复杂度为O(logn)。**非线程安全**


## List 接口
List 是有序的 Collection，可以含有相同的元素，每个元素的插入位置是可控制的，能够使用检索（类似于数组的下标）访问元素。
除了具有 Collection 接口必备的 Iterator() 方法外，List 还提供一个 listIterator() 方法，返回一个ListIterator 接口，和标准的 Iterator 接口相比，ListIterator 多了一些 add()、set() 之类的方法，允许添加、删除、设定元素，还能向前或向后遍历 previous()。

实现 List 接口的常用类有：
### ArrayList
实现了可变大小数组，**非线程安全**，支持随机访问。不同于 Array 的是，Array 可以容纳基本类型和对象，而 ArrayList 只能容纳对象。
每个 ArrayList 实例都有一个容量（capacity)，即用于存储元素的数组的大小，该容量可以随着元素的不断添加而自动增长，但是增长算法并没有定义。当需要插入大量元素时，在插入前可以调用 arrayList.ensureCapacity(minCapacity) 来增加容量以提高插入效率。

### LinkedList
擅长增删元素，**非线程安全**。同时扩展了 Queue 接口（准确说是 扩展了 Deque 接口，而 Deque 接口扩展了 Queue 接口），具有队列的性质和优势。

### Vector _遗留类_：
类似于 ArrayList，支持同步，**线程安全**。

### Stack _遗留类_：
继承自 Vector，也支持同步，提供了更多的的方法：
* push、pop；
* peek：得到栈顶元素；
* empty：判断是否为空；
* search 检测一个元素的位置。

## Queue 接口
支持队列的一般性质：先进先出，头部出队，尾部入队，etc. 

实现 Queue 接口的常用类有：
### LinkedList
见上述。

###　PriorityQueue
保存队列元素的顺序并不是按照加入队列的顺序，而是 **按照队列元素的大小进行重新排序**。所以调用 peek() 或 poll() 的方法取出队列中的元素通常都是最小的元素。其扩展的是 AbstractQueue 接口，而 AbstractQueue 接口 扩展了 Queue 接口。
一些方法：
* peek：取出队列头元素，但不删除之；
* pull：取出并删除队列头元素。

## Map 接口
用于存储键值对结构对象。

实现 Map 接口的常用类有：
### HashMap
非同步的，非线程安全。

### HashTable
同步的，保证线程安全，其他入操作等几乎等同于 HashMap。由于同步的原因，单线程环境下效率不高，此时应使用 HashMap。
从 JDK 1.5 开始，新增 ConcurrentHashMap 类，提供更好的扩展性，用以替代 HashTable。

### LinkedHashMap


### TreeMap

