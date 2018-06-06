---
title: Java-线程
date: 2018-04-14 15:39:53
categories: java
tags: [java, 并发]
---
[toc]
线程安全的实现方式。

## 隐式锁 synchronized
线程同步 synchronized 有两种方式，一种是放在同步方法上，一种是放在同步代码块里。 

## 显示锁 Lock 和 ReentrantLock 


### volatile
见 _java 内存模型_

### synchronized vs volatile
* volatile 是线程同步的轻量级实现，所以 volatile 性能比 synchronized 要好，并且 volatile 只能用于修饰变量，而 synchronized 可以修饰方法、代码块。目前 synchronized 在执行效率上提升明显，使用频率在增大；
* 多线程访问 volatile 不会发生阻塞，而 synchronized 会发生阻塞；
* volatile 能保证数据的可见性，但不保证原子性；synchronized 可以保证原子性，也可以间接保证可见性，因为它会将私有内存和公共内存中的数据做同步；
* volatile 解决的是变量在多个线程之间的可见性，synchronized 解决的是多个线程之间访问资源的同步性。

### 显式锁 ReentrantLock
Lock 接口：提供了一种无条件的、可轮询的、定时的、可中断的锁获取操作，所有加锁和解锁的方法都是显式的。其实现提供与内部锁相同的内存可见性语义，但在加锁语义、调度算法、顺序保证、性能特性等方面可以有所不同。
ReentrantLoc 实现了 Lock 接口，并提供了与 synchronized 相同的互斥性、内存可见性、可重入的加锁语义。

**局限性**
如果没有使用 finally 来释放 try-catch 块中的 Lock，将很难追踪到最初发生错误的位置，因为没有记录应该释放锁的位置和时间。这就是 ReentrantLock 不能完全替代 synchronized 的原因：它更加危险，在程序的执行控制离开被保护的代码块时，不会自动清除锁——finally 是解决办法，但可能被忘记。

**在使用某种形式的加锁时，都要考虑出现异常时的情况。**

### synchronized vs lock
java 的两种锁机制。
**用法上**
synchronized：使用在需要同步的对象中，可以用在方法级别上，也可以加在特定代码块中，括号中表示需要锁的对象；
lock：需要显示指定起始和终止位置。一般就是使用 ReentrantLock 类作为锁，多个线程中必须要使用一个 ReentrantLoc 类作为对象才能保证锁的生效。而且在加锁和解锁处需要通过 lock() 和 unlock() 显式指出。所以一般要在 finally 块中写 unlock() 以防死锁。此外，ReentrantLoc 类提供了 2 种 竞争锁机制：公平锁和非公平锁。一般而言，非公平锁效率更高：公平锁需要维护线程队列顺序。

**性能上**
synchronized 是托管给 JVM 执行的，而 lock 是 java 写的控制锁的代码。从 java 1.6 开始，synchronized 可以有很多优化：有适应自旋、锁消除、锁粗化、轻量级锁、偏向锁等，性能上也有很大提升。synchronized 采用的是 CPU 悲观锁机制，即线程获得的是独占锁；而 lock 采用的是乐观锁机制，其实现就是 CAS。研究 ReentrantLock 源码可以发现，其中一个比较重要的获得锁的方法是 compareAndsetState()，这里就是调用的 CPU 提供的特殊指令支持。

**用途上**
一般情况下，两者没有显著区别。而在特殊的复杂并发同步问题中，如以下情况，则更适合 ReentrantLoc：
* 某个线程在等待一个锁的控制权期间，需要中断；
* 需要分开处理一些 wait-notify，ReentrantLoc 里的 Condition 应用，能够控制 notify 特定线程；
* 具有公平锁功能，每个到来的线程都将排队等待。

{% asset_img synchronized-vs-lock.PNG synchronized vs lock %}


### 锁与无锁

#### 基于锁的并发
优点：
1、编程模型简单，如果小心控制上锁顺序，一般来说不会有死锁的问题；
2、可以通过调节锁的粒度来调节性能。
缺点：
1、所有基于锁的算法都有死锁的可能；
2、上锁和解锁时进程要从用户态切换到内核态，并可能伴随有线程的调度、上下文切换等，开销比较重；
3、对共享数据的读与写之间会有互斥。

#### 无锁编程（lock free）
常见的lock free编程一般是基于CAS(Compare And Swap)操作：CAS(void *ptr, Any oldValue, Any newValue);
即查看内存地址ptr处的值，如果为oldValue则将其改为newValue，并返回true，否则返回false。X86平台上的CAS操作一般是通过CPU的CMPXCHG指令来完成的。CPU在执行此指令时会首先锁住CPU总线，禁止其它核心对内存的访问，然后再查看或修改*ptr的值。简单的说CAS利用了CPU的硬件锁来实现对共享资源的串行使用。
优点：
1、开销较小：不需要进入内核，不需要切换线程；
2、没有死锁：总线锁最长持续为一次read+write的时间；
3、只有写操作需要使用CAS，读操作与串行代码完全相同，可实现读写不互斥。
缺点：
1、编程非常复杂，两行代码之间可能发生任何事，很多常识性的假设都不成立。
2、CAS模型覆盖的情况非常少，无法用CAS实现原子的复数操作。

#### 对比
在性能层面上，CAS与mutex/readwrite lock各有千秋，简述如下：
1、单线程下CAS的开销大约为10次加法操作，mutex的上锁+解锁大约为20次加法操作，而readwrite lock的开销则更大一些。
2、CAS的性能为固定值，而mutex则可以通过改变临界区的大小来调节性能；
3、如果临界区中真正的修改操作只占一小部分，那么用CAS可以获得更大的并发度。
4、多核CPU中线程调度成本较高，此时更适合用CAS。
跳表和红黑树的性能相当，最主要的优势就是当调整(插入或删除)时，红黑树需要使用旋转来维护平衡性，这个操作需要动多个节点，在并发时候很难控制。而跳表插入或删除时只需定位后插入，插入时只需添加插入的那个节点及其多个层的复制，以及定位和插入的原子性维护。所以它更加可以利用CAS操作来进行无锁编程。