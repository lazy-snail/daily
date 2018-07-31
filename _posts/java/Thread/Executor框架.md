---
title: Executor框架
date: 2018-06-08 19:11:50
categories: java
tags: [java, 并发, 线程]
---
[toc]
# 组成
{% asset_img Executor框架组成.png Executor框架组成 %}
框架主要由 3 大部分组成：
* 任务：被执行任务需要实现 Runnable/Callable 接口；
* 任务的执行：任务执行机制的核心接口就是 Executor，ExecutorService 接口继承了 Executor 接口，拥有两个具体实现类：ThreadPoolExecutor、ScheduledThreadPoolExecutor；
* 异步计算的结果：包括接口 Future 和 其实现类 FutureTask

## 执行流程
{% asset_img Executor框架使用示意图.PNG Executor框架使用示意图 %}

# Executor 接口
整个框架的基础，它 **将任务的提交和任务的执行解耦**。
该接口用 Runnable/Callable 来表示任务，提供了一种标准的方法将任务的提交过程与执行过程分离开，并提供对生命周期的支持和统计信息收集、性能监视等机制。
```java
public interface Executor {
    void execute(Runnable command);
}
```
Executor 接口只有一个 execute 方法，用来替代通常创建或启动线程的方法，如，使用 Thread 来创建并启动线程：
```java
Thread t = new Thread();
t.start();
```
而使用 Executor 来启动线程执行任务的代码如下：
```java
Thread t = new Thread();
executor.execute(t);
```
对于不同的 Executor 实现，execute() 方法可能是创建一个新线程并立即启动，也有可能是使用已有的工作线程来运行传入的任务，也可能是根据设置线程池的容量或者阻塞队列的容量来决定是否要将传入的线程放入阻塞队列中或者拒绝接收传入的线程。

## ExecutorService 接口
继承了 Executor 接口，提供了管理终止的方法，以及可以为跟踪一个或多个异步任务执行状况而生成 Future 的方法，增加了 shutDown()、shutDownNow()、invokeAll()、invokeAny()、submit() 等方法。如果需要支持即时关闭，即 shutDownNow() 方法，则任务需要正确处理中断。

## ScheduledExecutorService 接口
继承了 ExecutorService 接口并增加了 schedule 方法。调用 schedule 方法可以在指定的延时后执行一个 Runnable 或者 Callable 任务。ScheduledExecutorService 接口还定义了按照指定时间间隔定期执行任务的 scheduleAtFixedRate() 方法和 scheduleWithFixedDelay() 方法。

# ThreadPoolExecutor 类
继承自 AbstractExecutorService 虚类，后者实现了 ExecutorService 接口。
ThreadPoolExecutor 通常使用工厂类 Executors 来创建，主要创建 3 种类型的 ThreadPoolExecutor：SingleThreadExecutor、FixedThreadPool和CachedThreadPool。具体又包含以下 5 种：

## 5 种线程池
### newSingleThreadExecutor
创建一个单线程化的线程池，即它只会使用一个线程来执行所有任务（相当于串行操作），保证所有任务按照指定顺序（FIFO、LIFO、优先级等）顺序执行。
适用于需要顺序执行各个任务的场景，它保证在任意时间点，不会有多个活动线程。

### newCachedThreadPool
创建一个可缓存线程池，如果线程池长度超过处理需求，可灵活回收空闲线程，若没有可回收，则为任务新建线程。此种线程池（理论上）可无限扩张，然而当执行下一个任务时如果探测到前一个任务已经完成，那么就会复用（而非新建）线程。
适用于执行很多短期异步任务的小程序、负载较轻的服务器等场景。

### newFixedThreadPool
创建一个固定长度的线程池。也就是可根据系统资源情况，控制线程的最大并发数，超出的线程会在队列中等待。
适用于需要控制资源管理的场景，能够限制线程数量，如重量级服务器。

### newScheduledThreadPool
创建一个固定长度的线程池。支持定时/延迟和周期执行任务等功能。
适用于需要多个后台线程执行周期任务，同时能够提供资源管理需求、限制后台线程数量的场景。

### newSingleThreadScheduledExecutor
创建一个单线程“池”，支持定时/延迟和周期执行任务等。可以看作上面两个的结合。
适用于需要单个后台线程执行周期任务，同时需要保证顺序地执行各个任务的场景。

## 重要字段
```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int COUNT_MASK = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```
ctl 是对线程池的运行状态和线程池中有效线程数量进行控制的一个字段：包含两部分信息：
* 线程池的运行状态（runState）；
* 线程池内有效线程的数量（workerCount）。
可见，使用 int 的高 3 位保存 runState，低 29 位保存 workerCount：COUNT_BITS = 29。COUNT_MASK 用来计算最大线程数，位 2^29-1，约 5 亿。

## 构造方法
```java
public ThreadPoolExecutor(
    int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler) 
    {}
```
### 参数解释：
* corePoolSize：核心线程数量，当有新任务在 execute() 方法提交时，会执行以下判断：
> 如果运行的线程少于 corePoolSize，则创建新线程来处理任务，即使线程池中的其他线程是空闲的；
> 如果线程池中的线程数量大于等于 corePoolSize 且小于 maximumPoolSize，则只有当 workQueue 满时才创建新的线程去处理任务；
> 如果设置的 corePoolSize 和 maximumPoolSize 相同，则创建的线程池的大小是固定的，这时如果有新任务提交，若 workQueue 未满，则将请求放入 workQueue 中，等待有空闲的线程去从 workQueue 中取任务并处理；
> 如果运行的线程数量大于等于 maximumPoolSize，这时如果 workQueue 已经满了，则通过 handler 所指定的策略来处理任务；
> 所以，任务提交时，判断的顺序为 corePoolSize --> workQueue --> maximumPoolSize。
* maximumPoolSize：最大线程数量；
* workQueue：保存等待执行的任务的阻塞队列，当提交一个新的任务到线程池以后, 线程池会根据当前线程池中正在运行着的线程的数量来决定对该任务的处理方式，主要有以下几种处理方式:
> 直接切换：这种方式常用的队列是 SynchronousQueue，但现在还没有研究过该队列，这里暂时还没法介绍；
> 使用无界队列：一般使用基于链表的阻塞队列 LinkedBlockingQueue。如果使用这种方式，那么线程池中能够创建的最大线程数就是 corePoolSize，而maximumPoolSize 就不会起作用了（后面也会说到）。当线程池中所有的核心线程都是 RUNNING 状态时，这时一个新的任务提交就会放入等待队列中;
> 使用有界队列：一般使用 ArrayBlockingQueue。使用该方式可以将线程池的最大线程数量限制为 maximumPoolSize，这样能够降低资源的消耗，但同时这种方式也使得线程池对线程的调度变得更困难，因为线程池和队列的容量都是有限的值，所以要想使线程池处理任务的吞吐率达到一个相对合理的范围，又想使线程调度相对简单，并且还要尽可能的降低线程池对资源的消耗，就需要合理的设置这两个数量。

* keepAliveTime：线程池维护线程所允许的空闲时间。当线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了keepAliveTime；
* threadFactory：ThreadFactory 类型的变量，用来创建新线程。默认使用 Executors.defaultThreadFactory() 来创建线程。使用默认的 ThreadFactory 来创建线程时，会使新创建的线程具有相同的 NORM_PRIORITY 优先级并且是非守护线程，同时也设置了线程的名称;
* handler：RejectedExecutionHandler 类型的变量，表示线程池的饱和策略，及达到 maximumPoolSize。如果阻塞队列满了并且没有空闲的线程，这时如果继续提交任务，就需要采取一种策略处理该任务。线程池提供了4种策略：
> AbortPolicy：直接抛出异常，这是默认策略；
> CallerRunsPolicy：用调用者所在的线程来执行任务；
> DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
> DiscardPolicy：直接丢弃任务；

# Future 接口
Future 接口和其实现类 FutureTask 用来表示异步计算的结果：把 Runnable/Callable 接口的实现类提交（submit）给 ThreadPoolExecutor/ScheduledThreadPoolExecutor 时，ThreadPoolExecutor/ScheduledThreadPoolExecutor 会返回一个 FutureTask 对象：
```java
<T> Future<T> submit(Callable<T> task)
<T> Future<T> submit(Runnable task, T result)
Future<> submit(Runnable task)
```
FutureTask.get() 方法等待（也就是会阻塞在任务执行线程上）线程执行完任务，然后返回执行结果。

# Runnable & Callable 接口
二者的实现类都可以被 ThreadPoolExecutor/ScheduledThreadPoolExecutor 执行，区别在于：Runnable 不会返回结果，Callable 可以返回结果。
可以通过工厂类 Executors 把一个 Runnable 包装成一个 Callable：
```java
// 把一个 Runnable 包装成一个 Callable
public static Callable<Object> callable(Runnable task)
// ，把一个 Runnable 和一个待返回的结果包装成一个 Callable
public static <T> Callable<T> callable(Runnable task, T result)
```


https://blog.csdn.net/bluetjs/article/details/52935594
https://www.cnblogs.com/dolphin0520/p/3932921.html
https://juejin.im/entry/58fada5d570c350058d3aaad
https://blog.csdn.net/u011974987/article/details/51027795