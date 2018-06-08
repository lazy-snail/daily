---
title: java-JUCL
date: 2018-06-07 16:19:54
categories: java
tags: [java, 并发]
---
[toc]
java.util.concurrent.locks，和线程安全相关的包。该包内的类/接口大多都实现了 Serializable 接口，以下图示略去。

# Lock 接口
{% asset_img Lock接口.PNG Lock接口 %}

最初的 JDK 只支持 synchronized 关键字提供的锁同步。后续到 JDK1.5 加入了 JUCL（java.utils.concurrent.locks）包，提供了可重入锁（ReentrantLock）、读写锁（ReadWriteLock）、信号量、Condition 等。都是基于一个 **基本的等待队列抽象完成的**，JDK 文档中将这个抽象队列框架称为 AQS 同步框架。
Lock 比传统线程模型中的 synchronized 方式更加 OO，锁本身也是对象。两个线程执行的代码片段要实现同步互斥的效果，它们必须用同一个 Lock 对象。

## 接口实现
接口完全用 java 写成，在 java 语言层面是无关 JVM 的。提供了一种无条件的、可轮询的、定时的、可中断的锁获取操作，所有加锁和解锁的方法都是显式的。其实现提供与内部锁相同的行为和内存可见性语义，但在加锁语义、调度算法、顺序保证、性能特性等方面可以有所不同。接口方法：
{% asset_img Lock接口方法摘要.png 方法摘要 %}

## 显式锁 ReentrantLock
加锁和解锁都是显式实现的，乐观锁机制。
实现了 Lock 接口，并提供了与 synchronized 相同的 **互斥性、内存可见性、可重入的加锁语义**。
而 ReentrantLock 的锁功能是由其成员变量 Sync 类型的 sync 实现的。Sync 就继承了 AQS。从图中可以看出，Sync 在 ReentrantLock 中有两种实现类：NonfairSync、FairSync，对应非公平锁、公平锁两大类型。

### 公平锁与非公平锁
ReentrantLock 提供了两种锁的构造方法：创建非公平锁（默认）和公平锁：
```java
public ReentrantLock() {
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```
在公平锁上，线程按照它们发出请求的顺序获取锁；而在非公平锁上，允许线程请求插队：当一个线程请求非公平锁时，如果在发出请求的同时该锁变为可用状态，那么这个线程可以马上获得这个锁（跳过队列中所有等待线程）。非公平锁不提倡插队行为，但无法（不主动）防止某个线程在某个适合的时机进行插队。i.e. 公平锁中，如果锁被某一线程占有，那么所有后续请求该锁的线程都将按到达顺序排队；非公平锁中，如果请求该锁的线程到达的同时该锁恰好被另一线程释放，那么请求线程无需等待而直接获得锁，否则，即当前该锁处于被占有状态时，请求线程同样进入等待队列。

### 非公平锁性能更好
因为 **在恢复一个被挂起的线程与该线程真正运行之间存在着严重的延迟**。显然，非公平锁没有选择（如果此时恰好有一个线程请求了该锁）在前一个线程释放锁后去恢复一个等待该锁的线程，而是将锁直接分配给请求线程，这样就避免了该延迟：
假设线程 A 有一个锁，并且线程 B 请求这个锁（被挂起），当 A 释放锁时，B 将被唤醒，因此 B 将再次尝试获取该锁。与此同时，如果 线程 C 也请求这个锁，那么 C 很可能会在 B 被完全唤醒之前获得、使用以及释放该锁。这是一种双赢的局面，B 并没有因为 C 的插队而延迟（B 在唤醒过程中，C 就已经完成了对该锁的操作），C 也在快速地获取锁并执行了自己的操作，吞吐量也随之提高。
当持有锁的时间相对较长或者请求锁的平均时间间隔较长，应该使用公平锁：这些情况下，插队带来的吞吐量提升可能不会出现。

## 局限性
如果没有使用 finally 来释放 try-catch 块中的 Lock，将很难追踪到最初发生错误的位置，因为没有记录应该释放锁的位置和时间。这就是 ReentrantLock 不能完全替代 synchronized 的原因：它更加危险，在程序的执行控制离开被保护的代码块时，不会自动清除锁——finally 是解决办法，但可能被忘记。
_在使用某种形式的加锁时，都要考虑出现异常时的情况。_


# ReadWriteLock 接口
{% asset_img ReadWriteLock接口.PNG ReadWriteLock接口 %}

## 读写锁 ReentrantReadWriteLock
读写分离思想。维护了一对锁，分为读锁和写锁，多个读锁不互斥，读锁和写锁互斥，由 JVM 控制实现，语言层面无需考虑。可见，ReentrantReadWriteLock 也是通过 Sync 同步器类来实现锁机制的，且同样有公平/非公平之分。内部有 ReadLock、WriteLock 两个类，对应读锁、写锁。

## 读锁
如果确定一段临界值代码段只有读操作，那么可以上读锁：可以并发读，但不能写。线程进入读锁的条件：
* 没有其他线程的写锁；
* 没有写请求 | 有写请求，但调用线程和持有锁的线程是同一个。

读锁可重入的情况是：读线程在获得读锁后，可以再次获得读锁，但不可以获得写锁。
读锁不可升级为写锁，因为获得读锁后不释放前，无法再获取写锁。

## 写锁
如果临界区有写操作，需要上写锁。写锁是排他锁，只能单线程访问。进入写锁的条件：
* 没有其他线程的读锁（如果有读锁正在访问，则无法进入）；
* 没有其他线程的写锁。

写锁也是可重入锁：写线程在获得写锁或，可以再次获得写锁和读锁。
写锁可以降级为读锁：先获取写锁，再获取读锁，随后释放写锁，此时为读锁。

## vs
* 重入性方面见上；
* ReadLock 可以被多个线程持有并且在作用时排斥任何的 WriteLock，而 WriteLock 则是完全的互斥。这一特性最为重要（也是实现它的缘由）：对于高读取频率而相对较低写入的数据结构，使用此类锁同步机制则可以提高并发量；
* 不管是 ReadLock 还是 WriteLock 都支持 Interrupt，语义与 ReentrantLock 一致。 
* WriteLock 支持 Condition 并且与 ReentrantLock 语义一致，而 ReadLock 则不能使用 Condition，否则抛出 UnsupportedOperationException。


# Condition 接口
{% asset_img Condition接口.PNG Condition接口 %}
Lock 可以很好地解决线程同步问题，但线程间不仅仅有互斥，还有通信问题，而 JUCL 包中的 Condition 接口，就是用来处理线程间通信问题的。很大程度上，Condition 能够解决 Object 监视器方法（wait/notify/notifyAll）难以使用的问题。即，Condition 是多线程间协调通信的工具类。
条件（也称为条件队列或条件变量）为线程提供了一个含义，以便在某个状态条件现在可能为 true 的另一个线程通知它之前，一直挂起该线程（即让其“等待”）。因为访问此共享状态信息发生在不同的线程中，所以它必须受保护，因此要将某种形式的锁与该条件相关联。等待提供一个条件的主要属性是：以原子方式释放相关的锁，并挂起当前线程，就像 Object.wait 做的那样。
{% asset_img ConditionUML.PNG Condition接口 %}
这些接口方法中，
* await* 对应于 Object.wait()；
* signal() 对应于 Object.notify()；
* signalAll() 对应于 Object.notifyAll()。

每个 Lock 可以有任意数量的 Condition 对象，Condition 是与 Lock 绑定的，所以就有 Lock 的公平性特性：如果是公平锁，线程为按照 FIFO 的顺序从 Condition.await 中释放，如果是非公平锁，那么后续的锁竞争就不保证 FIFO 顺序。