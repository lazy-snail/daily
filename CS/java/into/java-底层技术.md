---
title: java-底层技术
date: 2018-06-06 08:35:12
categories: java
tags: java
---
[toc]
一些语言技术实现。

# COW 优化策略
COW（Copy-On-Write，写时复制）是一种用于程序设计中的优化策略：一开始大家都在共享同一个内容，当某个人（线程）想要修改这个内容的时候，才会真正把内容拷贝并形成一个新的内容然后再修改。这是一种延时懒惰策略，主要应用于多并发场景。

## COW 容器
i.e. 写时复制容器。JDK1.5 引入了两个 COW 机制的实现类容器：CopyOnWriteArrayList、CopyOnWriteArraySet。简单理解就是：当需要往容器添加元素时，不直接往当前容器添加，而是将当前容器进行拷贝，复制出一个新的容器，然后将待添加元素加入到新的容器中，添加完成后，再将原容器的引用指向新容器。这样做的好处是，可以 **对容器进行并发读而不需要加锁**，因为当前容器不会添加容器，也就不会改变。整体上是一种读写分离的思想，读和写在不同的容器进行。
{% asset_img COW添加元素.png 添加元素 %}
整个 add 操作在锁保护下进行，避免多线程并发造成副本混乱：
```java
public boolean add(E e) {
    synchronized (lock) {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    }
```
由于所有的写操作都是在新数组进行，如果有并发写，则通过锁来控制；如果并发读，则：
* 如果写操作未完成，那么直接读取原数组的数据；
* 如果写操作已完成，但引用还未指向新数组，那么也是直接读取原数组的数据；
* 如果写操作已完成，并且引用已指向新数组，那么直接读取新数组的数据；
可见，读操作可以不加锁。

## 应用场景
适用于 **读多写少的并发场景**。比如白名单，黑名单，商品类目的访问和更新场景等。

## 问题和解决
* 内存占用：根据实现原理，当可变操作频繁发生时，会导致效率低下，也会造成内存占用过多的问题。实用场景，根据实际需要，初始化 COW 容器大小，减少扩容开销；使用批量添加，减少容器复制次数。
* 数据一致性问题：COW 只能保证数据的最终一致性，不能保证数据的实时一致性。如果希望写入的数据马上能读取到，此时不适合适用 COW 容器。


一些泛型数据结构。

# 跳表（SkipList）
## 从红黑树到跳表
目前常用的 key-value 数据结构有三种：Hash 表、红黑树、SkipList。各自优缺点（不考虑删除操作）：
* Hash表：插入、查找最快，为 O(1)；如使用链表实现则可实现无锁；数据有序化需要显式的排序操作。
* 红黑树：插入、查找为 O(logn)，但常数项较小；无锁实现的复杂性很高，一般需要加锁；数据天然有序。
* SkipList：插入、查找为 O(logn)，但常数项比红黑树要大；底层结构为链表，可无锁实现（CAS）；数据天然有序。

如果要实现一个 key-value 结构，需求的功能有插入、查找、迭代、修改，那么首先 Hash 表就不是很适合了：因为迭代的时间复杂度比较高；而红黑树的插入很可能会涉及多个结点的旋转、变色操作，因此需要在外层加锁，这无形中降低了它可能的并发度；而 SkipList 底层是用链表实现的，可以实现为 lock free，同时它还有着不错的性能（单线程下只比红黑树略慢），非常适合用来实现 key-value 结构。

JDK 中提供的基于红黑树的 TreeMap 保证内部元素有序，但非线程安全的，而且红黑树的实现中维持二叉树的平衡是个非常复杂的实现，并且（如果实现并发安全的话）并发环境下保持平衡的操作会使性能受到一定影响。
跳表（SkipList）是一种随机化的数据结构，是一种 **空间换取时间** 的算法：建立多级索引，以（近似）二分查找的方式遍历一个有序链表。

跳表所实现的功能和红黑树类似。基于排序的索引结构，其效率和红黑树相近，但实现难度和编程难度更简单，且在并发环境下表现良好（这也是选择它作为一些容器类的线程安全版本底层数据结构的原因）。相应地，以空间换时间本就意味着空间效率的降低，这种折中在数据结构实现中经常遇到。性质：
* 由很多层结构组成
* 每一层都是一个有序的链表
* 底层(Level 1)的链表包含所有元素
* 如果一个元素出现在 Level i 的链表中，则它在 Level i 之下的链表也都会出现。
* 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素。

## 数据结构
首先，从命名可见它是基于链表的数据结构，其次它又是一种多层次的链表结构。使用概率均衡技术（而不是强制性均衡），使得增/删操作比传统平衡树更为简洁高效。
{% asset_img 一层跳表简图.jpg 一层跳表 %}

如图，类比传统链表的操作时间复杂度，跳表（一层）可以将查询降低到 O(n/2)——查询种会跳过部分节点，而 **通过随机函数生成链表内每个节点包含多少个指向后续元素的指针** 的方式（含有概率事件的意味），构造出多层次链表结构，从而理论上降低至二叉查找级别：
{% asset_img 多层跳表简图.jpg 多层跳表 %}

## 操作
### 查找
以下为查找 19 的过程。
{% asset_img 跳表查找操作.jpg 跳表查找操作 %}

### 插入
执行以下步骤：
* 通过查找定位到需要插入的位置；
* 申请新的节点；
* 调整指针。
以插入 17 为例：
{% asset_img 跳表插入操作.png 跳表插入操作 %}

### 删除
类似于插入：
* 通过查找定位到需要删除的节点；
* 删除节点；
* 调整指针。
{% asset_img 跳表删除操作.png 跳表删除操作 %}

[实现原理](https://blog.csdn.net/ict2014/article/details/17394259 "跳表实现原理")
[实现原理](https://blog.csdn.net/bigtree_3721/article/details/51291974 "跳表实现原理")


# CAS
几乎所有现代处理器都包含了某种形式的原子读-改-写的特殊指令，如 CAS、关联加载/条件存储（Load-Linked/Store-Conditional），可以自动更新共享数据，而且能够检测到其他线程的干扰。
Compare and Swap：比较并交换，就是借助了 CPU 的这些特殊指令实现的，是 java.util.concurrent（JUC）包中实现的区别于 synchronized 同步锁的一种乐观锁，非阻塞算法（一个线程的失败或挂起不应该影响其他线程的失败或挂起）。
CAS 有 3 个操作数，内存值 V，旧的预期值 A，要修改的新值 B。当且仅当预期值 A 和内存值 V 相同时，将内存值 V 修改为 B，否则什么都不做：首先读取内存位置 V 的值 A，计算是否符合修改条件，是，则以原子方式将 V 中的值由 A 变成 B，期间会检测是否有其他线程干扰（修改 V 值），有干扰则进入失败重试阶段；否，则不进行任何操作，并返回 V 中的值。
程序内部执行 CAS 并不需要执行 JVM 代码、系统调用或线程调度操作。而 CAS 的缺点主要是：它将使调用者处理竞争问题（重试、回退、放弃等），而在锁中竞争问题是自动处理的（不需要应用开发者再处理）：线程在获得锁之前一直阻塞。
java 的 CAS 同时具有 volatile 的读和写的内存语义。

## JNI 调用 和 CPU 锁
java 中，CAS 通过调用 JNI（Java Native Interface，java 本地调用，允许 java 调用其他语言）代码实现。一般借助 C 来调用 CPU 底层指令实现。这属于硬件实现细节，涉及到 CPU 硬件设计中的锁概念。
CPU 有 3 种类型的锁：
* 自动保证基本内存操作的原子性
首先处理器会自动保证基本的内存操作的原子性。处理器保证从系统内存当中读取或者写入一个字节是原子的，意思是当一个处理器读取一个字节时，其他处理器不能访问这个字节的内存地址。处理器能自动保证单处理器对同一个缓存行里进行 16/32/64 位的操作是原子的，但是复杂的内存操作处理器不能自动保证其原子性，比如跨总线宽度，跨多个缓存行，跨页表的访问。但是处理器提供总线锁定和缓存锁定两个机制来保证复杂内存操作的原子性：
* 总线锁保证原子性
第一个机制是通过总线锁保证原子性。如果多个处理器同时对共享变量进行读改写（i++就是经典的读改写操作）操作，那么共享变量就会被多个处理器同时进行操作，这样读改写操作就不是原子的，操作完之后共享变量的值会和期望的不一致，举个例子：如果i=1,我们进行两次i++操作，我们期望的结果是3，但是有可能结果是2。如下图：
{% asset_img CPU总线锁示例.png CPU总线锁示例 %}
原因是有可能多个处理器同时从各自的缓存中读取变量 i，分别进行加一操作，然后分别写入系统内存当中。那么想要保证读改写共享变量的操作是原子的，就必须保证 CPU1 读改写共享变量的时候，CPU2 不能操作缓存了该共享变量内存地址的缓存。
处理器使用总线锁就是来解决这个问题的。所谓总线锁就是使用处理器提供的一个 LOCK＃ 信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住，那么该处理器可以独占使用共享内存。
* 缓存锁保证原子性
第二个机制是通过缓存锁定保证原子性。在同一时刻我们只需保证对某个内存地址的操作是原子性即可，但总线锁定把 CPU 和内存之间通信锁住了，这使得锁定期间，其他处理器不能操作其他内存地址的数据，所以总线锁定的开销比较大，最近的处理器在某些场合下使用缓存锁定代替总线锁定来进行优化。
频繁使用的内存会缓存在处理器的 L1、L2 和 L3 高速缓存里，那么原子操作就可以直接在处理器内部缓存中进行，并不需要声明总线锁，在奔腾6和最近的处理器中可以使用“缓存锁定”的方式来实现复杂的原子性。所谓“缓存锁定”就是如果缓存在处理器缓存行中内存区域在LOCK操作期间被锁定，当它执行锁操作回写内存时，处理器不在总线上声言 LOCK＃ 信号，而是修改内部的内存地址，并允许它的缓存一致性机制来保证操作的原子性，因为缓存一致性机制会阻止同时修改被两个以上处理器缓存的内存区域数据，当其他处理器回写已被锁定的缓存行的数据时会起缓存行无效，在例1中，当 CPU1 修改缓存行中的i时使用缓存锁定，那么 CPU2 就不能同时缓存了 i 的缓存行。
但是有两种情况下处理器不会使用缓存锁定:
> * 当操作的数据不能被缓存在处理器内部，或操作的数据跨多个缓存行（cache line），此时处理器会调用总线锁定；
> * 有些处理器不支持缓存锁定。对于 Intel486 和奔腾处理器，就算锁定的内存区域在处理器的缓存行中也会调用总线锁定。

## CAS 缺陷
CAS 虽然很高效地实现了原子操作，但是 CAS 仍然存在 3 个问题：
* ABA问题。
因为 CAS 需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是 A，变成了 B，又变成了 A，那么使用 CAS 进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA 问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号 +1，那么 A-B-A 就会变成 1A-2B-3A。从 Java1.5 开始 JDK 的 atomic 包里提供了一个类 AtomicStampedReference 来解决 ABA 问题。这个类的 compareAndSet 方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值；
* 循环时间长、开销大。
自旋 CAS 如果长时间不成功，会给 CPU 带来非常大的执行开销。如果 JVM 能支持处理器提供的 pause 指令那么效率会有一定的提升，pause 指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使 CPU 不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起 CPU 流水线被清空（CPU pipeline flush），从而提高 CPU 的执行效率；
* 只能保证一个共享变量的原子操作。
当对一个共享变量执行操作时，我们可以使用循环 CAS 的方式来保证原子操作，但是对多个共享变量操作时，循环 CAS 就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量 i＝2,j=a，合并一下 ij=2a，然后用 CAS 来操作 ij。从 Java1.5 开始 JDK 提供了 AtomicReference 类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行 CAS 操作。

[CAS 分析](http://zl198751.iteye.com/blog/1848575)

# AQS
AQS：AbstractQueuedSynchronizer，抽象队列同步器，定义了一套多线程访问共享资源的同步器框架。
{% asset_img AQS框架.png 框架 %}
它维护了一个 volatile int state（代表共享资源，用来维护同步状态）和一个 FIFO 线程等待队列（多线程争用资源被阻塞时会进入此队列）。state 的访问方式有三种:
* getState()
* setState()
* compareAndSetState()

AQS 定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如 ReentrantLock）和 Share（共享，多个线程可同时执行，如 Semaphore/CountDownLatch）。通过 state 字段描述有多少线程持有锁和锁的种类，用标志位标示独占锁/共享锁以及（如果共享）锁被持有的数量、锁的可重入性以及（如果可重入）重入次数等。
不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS 已经在顶层实现。
基于 AQS 实现的同步器有：ReentrantLock、Semaphore、ReentrantReadWriteLock、CountDownLatch、FutureTask 等。

[AQS](https://blog.csdn.net/bigtree_3721/article/details/79317298)

# CLH MCS
## CLH 锁
Craig, Landin, and Hagersten  locks: 一个自旋锁，能确保无饥饿性，提供先来先服务的公平性。基于链表的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，它不断轮询前驱的状态，如果发现前驱释放了锁就结束自旋。

## MCS 锁
MCS Spinlock 是一种基于链表的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，直接前驱负责通知其结束自旋，从而极大地减少了不必要的处理器缓存同步的次数，降低了总线和内存的开销。

## vs
* 从代码实现来看，CLH比MCS要简单得多。
* 从自旋的条件来看，CLH是在前驱节点的属性上自旋，而MCS是在本地属性变量上自旋
* 从链表队列来看，CLH的队列是隐式的，CLHNode并不实际持有下一个节点；MCS的队列是物理存在的。
* CLH锁释放时只需要改变自己的属性，MCS锁释放则需要改变后继节点的属性。
注意：这里实现的锁都是独占的，且不能重入的。