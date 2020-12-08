---
title: Queue-线程安全
date: 2018-06-06 14:03:07
categories: java
tags: [java, 容器]
---
[toc]
这里介绍 Queue 容器中实现了同步方法的 LinkedBlockingQueue、ConcurrentLinkedQueue、LinkedBlockingDeque、ConcurrentLinkedDeque

# LinkedBlockingQueue & ConcurrentLinkedQueue
二者都是线程安全的队列实现。从命名可知，前者是使用锁（ReentrantLock）实现的阻塞队列（BlockingQueue），会遇到锁本身的所有可能的问题：死锁、阻塞等（名称由来），而后者是采用 CAS 实现的线程安全，不存在阻塞情况。

## LinkedBlockingQueue
```java
public class LinkedBlockingDeque<E>
    extends AbstractQueue<E>
    implements BlockingDeque<E>, java.io.Serializable {}
```

## ConcurrentLinkedQueue 并发队列
```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {}
```

## 对比
* 对于 LinkedBlockingQueue，take 方法虽然在内部实现了加锁 wait()，但是由于其他的开销，导致性能相比于 poll + wait() 有所下降。但是如果采用 poll 方法，那么由于大量的线程存在空转的情况，导致争用处理机，导致性能急剧下降。
* 对于 ConcurrentLinkedQueue，由于内部采用 CAS 保证并发安全，在采用 poll + wait() 时相比前者有所提升，但是不是很明显，但是对于 poll 方式，由于去除了锁的开销，同时虽然相比于其自身的 poll + wait() 方式性能下降不少，但是相对于 LinkedBlockingQueue，性能提升相当明显。


# LinkedBlockingDeque & ConcurrentLinkedDeque
线程安全的双向队列，具体情况类似上面的两个。


# 阻塞队列
BlockingQueue，使用锁机制实现（ReentrantLock），支持两个附加操作的队列：
* 队列空时，获取元素的线程会等待队列变为非空；
* 队列满时，存储元素的线程会等待队列可用。
{% asset_img 阻塞队列处理方法.JPG 阻塞队列处理方法 %}

>* 抛出异常：当阻塞队列满时，尝试继续往队列里插入元素，会抛出 IllegalStateException(Queue full)异常。当队列为空时，尝试继续从队列里获取元素时会抛出 NoSuchElementException 异常；
>* 返回特殊值：插入方法会返回是否成功，成功则返回 true。移除方法则是从队列里拿出一个元素，如果没有则返回 null；
>* 一直阻塞：当阻塞队列满时，如果生产者线程往队列里 put 元素，队列会一直阻塞生产者线程，直到拿到数据，或者响应中断退出。当队列空时，消费者线程试图从队列里 take 元素，队列也会阻塞消费者线程，直到队列可用；
>* 超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，如果超过一定的时间，生产者线程就会退出。

**阻塞队列常用于生产者和消费者的场景**。

## 阻塞队列实现
截至 JDK1.8，包括上面的两个（LinkedBlockingQueue 和 LinkedBlockingDeque）阻塞队列实现，共有以下 7 个阻塞队列实现。

## ArrayBlockingQueue
用数组实现的有界阻塞队列。此队列按照先进先出（FIFO）的原则对元素进行排序。默认情况下不保证访问者公平的访问队列，所谓公平访问队列是指阻塞的所有生产者线程或消费者线程，当队列可用时，可以按照阻塞的先后顺序访问队列，即先阻塞的生产者线程，可以先往队列里插入元素，先阻塞的消费者线程，可以先从队列里获取元素。通常情况下为了保证公平性会降低吞吐量。
可通过以下方式创建公平的阻塞队列（第二个参数 fair = true 即表明创建为公平队列）：
```java
ArrayBlockingQueue<Object> arrayBlockingQueue = new ArrayBlockingQueue<>(100, true);
```

## LinkedBlockingQueue
用链表实现的有界阻塞队列。此队列按照先进先出的原则对元素进行排序。

## LinkedBlockingDeque
由链表结构组成的双向阻塞队列。双端队列因为多了一个操作队列的入口，在多线程同时入队时，也就减少了一半的竞争。相比其他的阻塞队列，LinkedBlockingDeque 多出了双向队列的操作方法，在初始化 LinkedBlockingDeque 时可以设置容量防止其过渡膨胀。另外双向阻塞队列可以运用在“工作窃取”模式中。

## PriorityBlockingQueue
支持优先级的无界阻塞队列。默认情况下元素采取自然顺序排列，也可以通过比较器 comparator 来指定元素的排序规则。

## DelayQueue
使用优先级队列实现的、支持延时获取元素的无界阻塞队列。队列使用 PriorityQueue 来实现。队列中的元素必须实现 Delayed 接口，在创建元素时可以指定多久才能从队列中获取当前元素，只有在延迟期满时才能从队列中提取元素。必须实现 compareTo来 指定元素的顺序。
应用场景：
* 缓存系统的设计：可以用 DelayQueue 保存缓存元素的有效期，使用一个线程循环查询 DelayQueue，一旦能从 DelayQueue 中获取元素时，表示缓存有效期到了；
* 定时任务调度。使用 DelayQueue 保存当天将会执行的任务和执行时间，一旦从 DelayQueue 中获取到任务就开始执行，如TimerQueue 就是使用 DelayQueue 实现的。

## SynchronousQueue
不存储元素的阻塞队列。每一个 put 操作必须等待一个 take 操作，否则不能继续添加元素。可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合于传递性场景，比如在一个线程中使用的数据，传递给另外一个线程使用，吞吐量高于 LinkedBlockingQueue 和 ArrayBlockingQueue。

## LinkedTransferQueue
由链表结构组成的无界阻塞 TransferQueue 队列。相对于其他阻塞队列，多了 tryTransfer 和 transfer 方法：
* transfer方法：如果当前有消费者正在等待接收元素（消费者使用 take 方法或带时间限制的 poll 方法时），transfer 方法可以把生产者传入的元素立刻 transfer（传输）给消费者。如果没有消费者在等待接收元素，transfer 方法会将元素存放在队列的 tail 节点，并等到该元素被消费者消费了才返回。transfer 方法的关键代码如下：
```java
// 试图把存放当前元素的 s 节点作为 tail 节点：
Node pred = tryAppend(s, haveData);

// 让 CPU 自旋等待消费者消费元素。因为自旋会消耗 CPU，
// 所以自旋一定的次数后使用 Thread.yield() 来暂停当前正在执行的线程，并执行其他线程：
return awaitMatch(s, pred, e, (how == TIMED), nanos);
```
* tryTransfer方法：试探下生产者传入的元素是否能直接传给消费者。如果没有消费者等待接收元素，则返回 false。和 transfer方法的区别是 tryTransfer 方法无论消费者是否接收，方法都立即返回。而 transfer 方法是必须等到消费者消费了才返回。对于带有时间限制的 tryTransfer(E e, long timeout, TimeUnit unit) 方法，则是试图把生产者传入的元素直接传给消费者，但是如果没有消费者消费该元素则等待指定的时间再返回，如果超时还没消费元素，则返回 false，如果在超时时间内消费了元素，则返回 true。


[阻塞队列](http://www.infoq.com/cn/articles/java-blocking-queue)