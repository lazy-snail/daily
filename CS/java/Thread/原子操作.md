---
title: java-原子操作
date: 2018-04-20 22:39:41
categories: java
tags: [java, 并发]
---
在 JUC 包的许多类中，如 Semaphore、ConcurrentLinkedQueue，都提供了比 synchronized 机制更高的性能和可伸缩性。这种性能的提升主要来源于 **原子变量** 和 **非阻塞的同步机制**。主要是利用 CPU 硬件对并发的支持，如 CAS 技术等。

## 分类
java.util.concurrent.atomic 包中提供了一些原子类：AtomicXxx，它们是线程安全的，但不是通过同步或锁机制来实现的，比锁的粒度更细，量级更轻。原子变量的操作会变为平台提供的用于并发访问的硬件原语。由于是非阻塞算法设计，不存在死锁和其他活跃性问题。
即使原子变量没有用于非阻塞算法的开发，它们也可以作为一种“更好的 vloatile 类型变量”。原子变量提供了与 volatile 类型变量相同的内存语义，并且支持原子的更新操作，从而使它们更加适用于实现计数器、序列生成器和统计数据收集等，同时还能比基于锁的方法提供更高的可伸缩性。
共有 12 个原子变量类，细分如下：
* 标量类（Scalar）：AtomicInteger、AtomicLong、AtomicBoolean、AtomicReference。最常用的一类原子变量；
* 更新器类：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater
* 数组类：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray；
* 复合变量类：AtomicStampedReference、AtomicMarkableReference；

所有这些类都支持 CAS。AtomicInteger、AtomicLong 还支持算术运算。如果想模拟其他基本类型的原子变量，可以将 short、byte 等类型通过与 int 类型进行转换，以及使用 floatToIntBits 或 doubleToLongBits 来转换浮点数。

## 为什么不直接使用锁
锁有以下问题：
* 因线程持有锁而导致其他线程被挂起并在稍后恢复的过程中，切换上下文或线程调度的开销可能比较大；
* 持有锁的线程被延迟执行（缺页、调度延迟等）情况发生时，所有等待该锁的其他线程也将无法执行：
* 对于细粒度的操作（如递增计数器）来说，锁是一种高开销的机制。

Java 语言的锁定语法本身很简洁，但 JVM 和操作在管理锁时需要完成的工作并不简单。实现锁定时需要遍历 JVM 中一条非常复杂的代码路径，并可能导致 OS 级的锁定、线程挂起以及上下文切换等操作。

_独占锁是一种悲观技术——它假设最坏情况（如果你不锁门，那么就会有破坏者进入），并且只有在确保其他线程不会造成干扰的情况下才能执行。_
对于细粒度的操作，还有一种更高效的乐观方法，可以在不发生干扰的情况下完成更新操作。这种方法需要借助冲突检查机制来判断在更新过程中是否存在来自于其他线程的干扰，如果存在，这个操作将失败，并且可以选择是否重试。

## 硬件对并发的支持 CAS
实现方面见底层技术和内存模型。
以 AtomicInteger 为例，看看在没有锁的情况下如何做到数据的正确性。
```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private volatile int value; 
    ... }
```
显然，在没有锁的情况下，要借助 volatile 解决可见性问题，以便在获取变量的值的时候可以直接读取：
```java
public final int get() { return value; }
```
++i 的实现：
```java
// jdk 1.7-
public final int incrementAndGet() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}
...
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```
可见在这里采用了 CAS（compareAndSet）操作：每次从内存中读取数据然后将此数据和 +1 后的结果进行 CAS 操作，如果成功就返回结果，否则重试直到成功为止。其中 compareAndSet 方法利用 JNI 来完成 CPU 指令的操作。
而到了 JDK1.8+，该方法有所调整，但整体解决方案还是一样的：
```java
public final int incrementAndGet() {
    return U.getAndAddInt(this, VALUE, 1) + 1;
}
...
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
```

