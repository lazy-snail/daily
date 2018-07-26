---
title: GC实现
date: 2018-04-25 10:28:55
categories: java
tags: [java, JVM]
---
[toc]
**收集算法是内存回收的方法论，垃圾收集器是内存回收的具体实现**。jvms 并未对垃圾收集器的实现作任何规范。这里以基于 JDK 1.7 的 HotSpot 虚拟机为例：
{% asset_img GC收集器.jpg "HotSpot 所包含的收集器" %}
上图中存在连线的收集器表明两者可以搭配使用，也可以看出不同收集器所处的区域（新生代收集器和老年代收集器）。
_显然，HotSpot 实现了这么多种类的收集器，就证明并没有一个“万能型收集器”存在以适用于各种场景。不同收集器就是为不同的使用场景而存在的。_

# Serial
{% asset_img Serial-SerialOld收集器.png Serial-SerialOld收集器运行示意图 %}
最基本、发展历史最悠久的新生代收集器。从名字可以看出，是一个单线程的收集器——并不是仅仅说明它只用一个 CPU 或一条收集线程来完成收集工作，而且它在进行 GC 时，必须暂停其他所有的工作线程（Stop the World），直到它收集结束。“Stop the World”是由 JVM 在后台自动发起和完成的，但这种在用户不可见的情况下把用户正常工作的线程全部停掉，甚至有明显的停顿，这对很多应用来说是难以接受的。
随着 JVM 的发展，新的收集器被不断开发实现，它们造成用户线程停顿的时间越来越短，但实现复杂度随之不断增长。但 Serial 收集器依然是 Client 模式下的默认新生代收集器：它简单而高效（与其他收集器的单线程相比），在用户的 **桌面应用场景** 中，收集几十上百兆新生代的停顿时间可以控制在几十毫秒内，这在 GC 不频繁的情况下是能够接受的。

# ParNew
{% asset_img ParNew-SerialOld收集器.png ParNew-SerialOld收集器运行示意图 %}
Serial 收集器的多线程版本。除使用多线程这一点不同外，其他行为包括所有控制参数、收集算法、Stop the World、对象分配规则、回收策略等都与 Serial 收集器一样。
ParNew 收集器是许多 **运行在 Server 模式下 JVM 的首选新生代收集器**。其中一个与性能无关但很重要的原因是，除了 Serial 收集器之外，目前只有它能与 CMS 收集器配合工作。它默认开启的收集线程数与 CPU(核) 数量相同，也可参数指定。

# Parallel Scavenge
新生代收集器，使用复制算法，并行多线程执行。
其特别之处在于，它的关注点与其他收集器不同：CMS 等收集器关注的是尽可能缩短 GC 时用户线程的停顿时间，而 Parallel Scavenge 收集器的目标是达到一个可控制的吞吐量（Throughput）。这里的吞吐量概念是：吞吐量 = 用户代码运行时间 / (用户代码运行时间 + GC 时间)。因而被称为 _“吞吐量优先”收集器_。
停顿时间越短就越适合交互式程序，良好的响应速度能够提升用户体验。而高吞吐量则可以高效率利用 CPU 时间，尽快完成程序的运算任务，主要 **适合后台执行而少交互的程序**。

**一个值得关注的参数：-XX: UseAdaptiveSizePolicy**
当打开这个参数，就不需要再手动指定新生代的大小（-Xmn）、Eden 与 Survivor 空间区域的比例（-XX: SurvivorRatio）、晋升老年代对象年龄（-XX: PretenureSizeThreshold）等细节参数了。jVM 将根据当前系统运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或最大吞吐量。这种调节方式成为 GC 自适应的调节策略（GC Ergonomics）。

# Serial Old
是 Serial 收集器的老年代版本，使用“标记-整理”算法。主要用于 Client 模式下。如果在 Server 模式下，主要有两大用途：
* 在 JDK 1.5 及之前的版本中与 Parallel Scavenge 收集器搭配使用；
* 作为 CMS 收集器的后备预案：在发生 Concurrent Mode Failure 时使用。

# Parallel Old
{% asset_img ParallelScavenge-ParallelOld收集器.png ParallelScavenge-ParallelOld收集器运行示意图 %}
是 Parallel Scavenge 收集器的老年代版本，使用“标记-整理“算法。在 JDK 1.6 开始提供，此前，Parallel Scavenge 收集器一直处于比较尴尬的状态：老年代只能用 Serial Old（PS MarkSweep）收集器，性能并不理想。直到该收集器的出现，“吞吐量优先”收集器才有了名副其实的应用组合：在 **注重吞吐量及 CPU 资源敏感的场景**，都可以优先考虑 Parallel Scavenge + Parallel Old 收集器。

# CMS
Concurrent Mark Sweep。
{% asset_img ConcurrentMarkSweep收集器.png ConcurrentMarkSweep收集器运行示意图 %}
以获取最短 GC 停顿时间为目标。目前很大一部分 java 应用集中在 **互联网站或 B/S 系统服务器上**，这类应用非常注重服务的响应速度，希望系统停顿时间最短以带给用户良好的体验。
从名字可以看出，使用的是“标记-清除”算法。但执行过程要更复杂一些：
* 初始标记（CMS initial mark）
* 并发标记（CMS concurrent mark）
* 重新标记（CMS remark）
* 并发清除（CMS concurrent sweep）

其中，初始标记、重新标记这两个步骤仍需要“Stop the World”。初始标记仅仅是标记一下 GC Roots 能直接关联到的对象，速度很快，并发标记阶段就是进行 GC RootsTracing 的可达性分析过程，而重新标记阶段则是为了修正并发标记期间因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿会比初始标记阶段稍长一些，但远比并发标记的时间短。
而由于整个过程中耗时最长的并发标记和并发清除过程的收集器线程都可以与用户线程一起工作，从总体上来说，CMS 收集器的 GC 过程是与用户线程一起并发执行的。

## 缺点
CMS 优点是并发收集、低停顿，因此也被称为“Concurrent Low Pause Collector，并发低停顿收集器”。但它也有 3 个明显的缺点：
* 和面向并发设计的程序一样，CMS 同样对 CPU 资源非常敏感：并发阶段会启动线程，占用一部分 CPU 资源；
* 无法处理浮动垃圾（Floating Garbage），可能出现“Concurrent Mode Failure”失败而导致另一次 Full GC。由于 CMS 并发清理阶段用户程序还在执行，因此这期间产生的新垃圾就没有被标记，即浮动垃圾，只能留到下一次 GC 时处理；
* 同样有基于“标记-清除”算法的缺点：内存碎片化。为此，CMS 提供了一个参数 -XX: UseCMSCompactAtFullCollection（默认开启），用于在 CMS 要进行 Full GC 时开启内存碎片的整理过程，过程中无法并发，即，会导致停顿时间变长；还提供了另一个参数 -XX: CMSFullGCsBeforeCompaction，用于设置执行多少次不压缩的 Full GC 后，跟着执行一次带压缩的 GC（默认为 0，表示每次进入 Full GC 时都会进行碎片整理）。

# G1
Garbage First。
{% asset_img G1收集器.png G1收集器运行示意图 %}
当前收集器技术发展的最前沿成果之一，**面向服务端应用**。特点有：
* 并行与并发：能充分利用多 CPU、多核环境下的硬件优势，使用多个 CPU 来缩短“Stop the World”停顿的时间，部分其他收集器原本需要停顿的 GC 动作，G1 仍然可以通过并发的方式让 java 程序继续执行；
* 分代收集：与其他收集器一样，G1 保留了分代概念。虽然可以不需要其他收集器配合就能独立管理整个 GC 堆，但它也能够采用不同方式处理不同存活时间的对象，以获取更好的收集效果；
* 空间整合：与 CMS 的“标记-清理” 算法不同，G1 从整体来看是基于“标记-整理”算法的，从局部（两个 Region 之间）来看却是基于“复制”算法实现的——总之，都不会产生空间碎片。有利于程序的长时间运行；
* 可预测的停顿：这是 G1 相比于 CMS 的另一大优势。能够建立可预测的停顿时间模型，让使用者明确指定在 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒——这几乎就是 实时 Java（RTSJ）的垃圾收集器的特征了。

不同于 CMS 等进行收集的范围是整个新生代或老年代，使用 G1 收集器时，java 堆的内存布局有很大变化：将整个 java 堆划分为多个大小相等的独立区域（Region），同时保留的新生代/老年代的概念在这里也不再是物理隔离的，而是都作为一部分 Region（不需要物理上连续）的集合。其特点中之所以能够建立可预测的停顿时间模型，就是因为它可以有计划地避免在整个 java 堆中进行全区域的垃圾回收。G1 跟踪各个 Region 里 GC 堆的价值大小（回收所获的的空间大小以及回收所需的时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region（这也就是 Garbage-First名称的由来）。
_Region 不可能是孤立的，对象间的引用关系复杂，还是会导致可达性分析时扫描整个 java 堆？这个问题不光在 G1 中存在，只是 G1 中更加明显。_
G1 中每个 Region 都有一个与之对应的 Remembered Set，jVM 用它来避免全堆扫描。JVM 发现程序在对 reference 类型的数据进行写操作时，会产生一个 Write Barrier 暂时中断写操作，检查 reference 引用的对象是否处于不同 Region 中（在分代的例子中就是检查是否老年代中的对象引用了新生代中的对象），如果是，就通过 CardTable 把相关引用信息记录到被引用对象所属 Region 的 Remembered Set 中。当进行内存回收时，在 GC 根节点的枚举范围中加入 Remembered Set 即可保证不对全堆扫描，也不会有遗漏。
如果不计算维护 Remembered Set 的操作，G1 收集器的执行大致可划分为：
* 初始标记（Initial Marking）
* 并发标记（Concurrent Marking）
* 最终标记（Final Marking）
* 筛选回收（Live Data Counting & Evacuation）

可以看出，前两个阶段和 CMS 的类似，而“最终标记”其实也类似于 CMS 的“重新标记”，只不过是把并发标记阶段期间新产生的垃圾记录到 Remembered Set Logs 中（CMS 无法处理之），随后需要停顿线程，将该日志记录合并到 Remembered Set中，最后在筛选回收阶段首先对各个 Region 的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划，这个阶段可以并发执行（但由于停顿时间是用户可控制的，并且停顿用户线程能大幅提高收集效率，也可停顿执行）。
在追求低停顿的场景，无疑 G1 是更好的选择，但如果追求吞吐量，那么 G1 也许并非首选。
