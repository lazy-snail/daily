---
title: java-ReentrantLock
date: 2018-06-07 16:19:54
categories: java
tags: [java, 并发]
---
[toc]
线程安全的实现方式之一。

# 显式锁
加锁和解锁都是显式实现的，乐观锁机制。

## Lock 接口
Lock 接口：提供了一种无条件的、可轮询的、定时的、可中断的锁获取操作，所有加锁和解锁的方法都是显式的。其实现提供与内部锁相同的行为和内存可见性语义，但在加锁语义、调度算法、顺序保证、性能特性等方面可以有所不同。接口方法：
{% asset_img lock接口方法摘要.png 方法摘要 %}
ReentrantLoc 实现了 Lock 接口，并提供了与 synchronized 相同的 **互斥性、内存可见性、可重入的加锁语义**。

## 公平锁 非公平锁



## ReentrantLock 局限性
如果没有使用 finally 来释放 try-catch 块中的 Lock，将很难追踪到最初发生错误的位置，因为没有记录应该释放锁的位置和时间。这就是 ReentrantLock 不能完全替代 synchronized 的原因：它更加危险，在程序的执行控制离开被保护的代码块时，不会自动清除锁——finally 是解决办法，但可能被忘记。
_在使用某种形式的加锁时，都要考虑出现异常时的情况。_

# synchronized vs lock
java 的两种锁机制。在竞争条件下 ReentrantLock 的实现要比 synchronized 的实现更具有伸缩性，这意味着当许多线程都竞争相同锁定时，使用 ReentrantLock 的吞吐量通常要比 synchronized 好。 

## 用法上
synchronized：使用在需要同步的对象中，可以用在方法级别上，也可以加在特定代码块中，括号中表示需要锁的对象；
lock：需要显示指定起始和终止位置。一般就是使用 ReentrantLock 类作为锁，多个线程中必须要使用一个 ReentrantLoc 类作为对象才能保证锁的生效。而且在加锁和解锁处需要通过 lock() 和 unlock() 显式指出。所以一般要在 finally 块中写 unlock() 以防死锁。此外，ReentrantLoc 类提供了 2 种 竞争锁机制：公平锁和非公平锁。一般而言，非公平锁效率更高：公平锁需要维护线程队列顺序。

## 性能上
synchronized 是托管给 JVM 执行的，而 lock 是 java 写的控制锁的代码。synchronized 采用的是 CPU 悲观锁机制，即线程获得的是独占锁；而 lock 采用的是乐观锁机制，其实现就是 CAS。研究 ReentrantLock 源码可以发现，其中一个比较重要的获得锁的方法是 compareAndsetState()，这里就是调用的 CPU 提供的特殊指令支持。

## 用途上
一般情况下，两者没有显著区别。而在特殊的复杂并发同步问题中，如以下情况，则更适合 ReentrantLock：
* 某个线程在等待一个锁的控制权期间，需要中断；
* 需要分开处理一些 wait-notify，ReentrantLoc 里的 Condition 应用，能够控制 notify 特定线程；
* 具有公平锁功能，每个到来的线程都将排队等待。
{% asset_img synchronized-vs-lock.PNG synchronized vs lock %}

## 如何选择
* 最好两个都不用，而是使用 JUC（java.util.concurrent）包提供的机制，能够帮助用户处理所有与锁相关的代码；
* 如果 synchronized 关键字能满足用户的需求，就用 synchronized，因为它能简化代码；
* 如果需要更高级的功能，就用 ReentrantLock 类，此时要注意及时释放锁，否则会出现死锁，通常在 finally 代码释放锁。

