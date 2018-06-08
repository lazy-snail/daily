---
title: java-ThreadLocal
date: 2018-06-07 20:06:13
categories: java
tags: [java, 并发]
---
[toc]
# ThreadLocal
线程本地变量/线程本地存储，是一个关于 **创建线程局部变量的类**。通常情况下创建的变量是可以被任何一个线程访问并修改的。而使用 ThreadLocal 创建的变量只能 **被当前线程访问**，其他线程则无法访问和修改。
其基本原理是，同一个 ThreadLocal 所包含的对象（如 ThreadLocal< String >就是 String 类型变量），在不同的 Thread 中有不同的副本（实际是不同的实例）。当线程结束时，它所使用的所有 ThreadLocal 相对应的实例副本都可被回收。要注意：
* 因为每个 Thread 内有自己的实例副本，且该副本只能由当前 Thread 使用。这是也是 ThreadLocal 命名的由来
* 每个 Thread 有自己的实例副本，且其它 Thread 不可访问，也就不存在多线程间共享的问题，也即不存在同步问题。
{% asset_image ThreadLocal.PNG ThreadLocal类 %}

## 原理
ThreadLocal 有一个内部类：ThreadLocalMap，用来维护线程与实例的映射。
### ThreadLocal 维护线程与实例的映射
作为一个可能的解决方案。既然每个访问 ThreadLocal 变量的线程都有自己的一个“本地”实例副本。一个可能的方案是 ThreadLocal 维护一个 Map，键是 Thread，值是它在该 Thread 内的实例。线程通过该 ThreadLocal 的 get() 方案获取实例时，只需要以线程为键，从 Map 中找出对应的实例即可：
{% asset_image ThreadLocal维护Map.png ThreadLocal维护线程与实例的映射 %}
该方案可满足上文提到的每个线程内一个独立备份的要求。每个新线程访问该 ThreadLocal 时，需要向 Map 中添加一个映射，而每个线程结束时，应该清除该映射。这里就有两个问题：
* 增加线程与减少线程均需要写 Map，故需保证该 Map 线程安全。虽然从ConcurrentHashMap的演进看Java多线程核心技术一文介绍了几种实现线程安全 Map 的方式，但它或多或少都需要锁来保证线程的安全性
* 线程结束时，需要保证它所访问的所有 ThreadLocal 中对应的映射均删除，否则可能会引起内存泄漏。（后文会介绍避免内存泄漏的方法）其中锁的问题，是 JDK 未采用该方案的一个原因。

### Thread 维护 ThreadLocal 与实例的映射
实际的解决方案。上一个方案中，出现锁的问题，原因在于多线程访问同一个 Map。转而，如果该 Map 由 Thread 维护，从而使得每个 Thread 只访问自己的 Map，那就不存在多线程写的问题，也就不需要锁：
{% asset_image Thread维护Map.png Thread维护ThreadLocal与实例的映射 %}
该方案虽然没有锁的问题，但由于每个线程访问某 ThreadLocal 变量后，都会在自己的 Map 内维护该 ThreadLocal 变量与具体实例的映射，如果不删除这些引用（映射），则这些 ThreadLocal 不能被回收，可能会造成内存泄漏。
该方案中的 Map 由 ThreadLocal 类的静态内部类 ThreadLocalMap 提供。该类的实例维护某个 ThreadLocal 与具体实例的映射。与 HashMap 不同的是，ThreadLocalMap 的每个 Entry 都是一个对键的弱引用。另外，每个 Entry 都包含了一个对值的强引用：
```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
使用弱引用的原因在于，当没有强引用指向 ThreadLocal 变量时，它可被回收，从而避免上文所述 ThreadLocal 不能被回收而造成的内存泄漏的问题。
但是，这里又可能出现另外一种内存泄漏的问题。ThreadLocalMap 维护 ThreadLocal 变量与具体实例的映射，当 ThreadLocal 变量被回收后，该映射的键变为 null，该 Entry 无法被移除。从而使得实例被该 Entry 引用而无法被回收造成内存泄漏。
_Entry 虽然是弱引用，但它是 ThreadLocal 类型的弱引用（也即上述它是对键的弱引用），而非具体实例的的弱引用，所以无法避免具体实例相关的内存泄漏_。

## 方法
### get()
读取实例时，线程首先通过 getMap(t) 方法获取自身的 ThreadLocalMap。从如下该方法的定义可见，该 ThreadLocalMap 的实例是 Thread 类的一个字段，即由 Thread 维护 ThreadLocal 对象与具体实例的映射：
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
  return t.threadLocals;
}
```
获取到 ThreadLocalMap 后，通过 map.getEntry(this) 方法获取该 ThreadLocal 在当前线程的 ThreadLocalMap 中对应的 Entry。该方法中的 this 即当前访问的 ThreadLocal 对象。
如果获取到的 Entry 不为 null，从 Entry 中取出值即为所需访问的本线程对应的实例。如果获取到的 Entry 为 null，则通过 setInitialValue() 法设置该 ThreadLocal 变量在该线程中对应的具体实例的初始值。

### setInitialValue()
该方法为 private 方法，无法被重载。
首先，通过 initialValue() 获取初始值。该方法为 public 方法，且默认返回 null。所以典型用法中常常重载该方法。上例中即在内部匿名类中将其重载。
然后拿到该线程对应的 ThreadLocalMap 对象，若该对象不为 null，则直接将该 ThreadLocal 对象与对应实例初始值的映射添加进该线程的  ThreadLocalMap 中。若为 null，则先创建该 ThreadLocalMap 对象再将映射添加其中。
这里并不需要考虑 ThreadLocalMap 的线程安全问题。因为每个线程有且只有一个 ThreadLocalMap 对象，并且只有该线程自己可以访问它，其它线程不会访问该 ThreadLocalMap，也即该对象不会在多个线程中共享，也就不存在线程安全的问题。
```java
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

### set()
该方法先获取该线程的 ThreadLocalMap 对象，然后直接将 ThreadLocal 对象（即代码中的 this）与目标实例的映射添加进 ThreadLocalMap 中。当然，如果映射已经存在，就直接覆盖。另外，如果获取到的 ThreadLocalMap 为 null，则先创建该 ThreadLocalMap 对象。
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

## 适用场景
根据以上对 ThreadLocal 的描述可见，其适用于：
* 每个线程需要有自己单独的实例；
* 实例需要在多个方法中共享，但不希望被多线程共享；
* 承载一些线程相关的数据，避免在方法中来回传递参数。

最常见的使用场景为 用来解决 数据库连接、Session管理等。

## 内存泄漏问题
ThreadLocal 并不会产生内存泄露，因为 ThreadLocalMap在 选择 key 的时候，并不是直接选择 ThreadLocal 实例，而是 ThreadLocal 实例的弱引用。

[理解ThreadLocal](http://www.jasongj.com/java/threadlocal)
[理解Java中的ThreadLocal](http://droidyue.com/blog/2016/03/13/learning-threadlocal-in-java)

