---
title: Java-原子操作
date: 2018-04-20 22:39:41
tags: Java
---
在 java.util.concurrent 包的许多类中，如 Semaphore、ConcurrentLinkedQueue，都提供了比 synchronized 机制更高的性能和可伸缩性。这种性能的提升主要来源于 **原子变量** 和 **非阻塞的同步机制**。




java.util.concurrent.atomic 包中提供了一些原子类：AtomicXxx，它们是线程安全的，但不是通过同步或锁机制来实现的，比锁的粒度更细，量级更轻。原子变量的操作会变为平台提供的用于并发访问的硬件原语。由于是非阻塞算法设计，不存在死锁和其他活跃性问题。
即使原子变量没有用于非阻塞算法的开发，它们也可以作为一种“更好的 vloatile 类型变量”。原子变量提供了与 volatile 类型变量相同的内存语义，并且支持原子的更新操作，从而使它们更加适用于实现计数器、序列生成器和统计数据收集等，同时还能比基于锁的方法提供更高的可伸缩性。
共有 12 个原子变量类，细分如下：
* 标量类（Scalar）：AtomicInteger、AtomicLong、AtomicBoolean、AtomicReference。最常用的一类原子变量；
* 更新器类：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater
* 数组类：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray；
* 复合变量类：AtomicStampedReference、AtomicMarkableReference；

所有这些类都支持 CAS。AtomicInteger、AtomicLong 还支持算术运算。如果想模拟其他基本类型的原子变量，可以将 short、byte 等类型呢过与 int 类型进行转换，以及使用 floatToIntBits 或 doubleToLongBits 来转换浮点数。


**锁的缺点**
* 因线程持有锁而导致其他线程被挂起并在稍后恢复的过程中，切换上下文或线程调度的开销可能比较大；
* 持有锁的线程被延迟执行（缺页、调度延迟等）情况发生时，所有等待该锁的其他线程也将无法执行：
* 对于细粒度的操作（如递增计数器）来说，锁是一种高开销的机制。

Java 语言的锁定语法本身很简洁，但 JVM 和操作在管理锁时需要完成的工作并不简单。实现锁定时需要遍历 JVM 中一条非常复杂的代码路径，并可能导致 OS 级的锁定、线程挂起以及上下文切换等操作。

_独占锁是一种悲观技术——它假设最坏情况（如果你不锁门，那么就会有破坏者进入），并且只有在确保其他线程不会造成干扰的情况下才能执行。_
对于细粒度的操作，还有一种更高效的乐观方法，可以在不发生干扰的情况下完成更新操作。这种方法需要借助冲突检查机制来判断在更新过程中是否存在来自于其他线程的干扰，如果存在，这个操作将失败，并且可以选择是否重试。

**硬件对并发的支持**
几乎所有现代处理器都包含了某种形式的原子读-改-写的特殊指令，如比较并交换（Compare-and-Swap）、关联加载/条件存储（Load-Linked/Store-Conditional）。OS 和 JVM 可以使用这些指令来实现锁和并发的数据结构。
如“比较并交换（CAS）”指令：首先读取内存位置 V 的值 A，计算是否符合修改条件，是，则以原子方式将 V 中的值由 A 变成 B，期间会检测是否有其他线程干扰（修改 V 值），有干扰则进入失败重试阶段；否，则不进行任何操作，并返回 V 中的值。程序内部执行 CAS 并不需要执行 JVM 代码、系统调用或线程调度操作。而 CAS 的缺点主要是：它将使调用者处理竞争问题（重试、回退、放弃等），而在锁中竞争问题是自动处理的（不需要应用开发者再处理）：线程在获得锁之前一直阻塞。
