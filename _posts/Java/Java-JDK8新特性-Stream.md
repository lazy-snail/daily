---
title: Java-JDK8新特性-Stream
date: 2018-04-28 22:49:33
categories:
tags: Java
---
[toc]
## Stream API
### 简介
Stream API (java.util.stream)把真正的函数式编程风格引入到 java 中。这里的 Stream 和 I/O 流不同，它更像具有 Iterable 的集合，但行为和集合类又有所不同。
官方描述：A sequence of elements supporting sequential & parallel aggregate operations. 可以理解为：
* Stream 是元素的集合，这让它看起来类似 Iterator；
* 支持顺序或并行方式进行聚合操作。

可以把 Stream 当作一个高级版本的 Iterator：对于 Iterator，用户只能按顺序一个一个地遍历元素，过程中对元素执行某些操作；而对于 Stream，用户只要给出需要对元素执行的操作，如“过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等，具体这些操作如何应用到每个元素上，则是 Stream 本身需要完成的工作。
eg：获取一个 List 中元素不为 null 的个数：
```java
List<Integer> nums = Lists.newArrayList(1, null, 3, 4, null, 6, 7);
nums.stream().filter(num -> num != null).count();
```
流提供了流畅的 API，可以进行数据转换和对结果执行某些操作，这些流操作既可以是“中间的”，也可以是“末端的”：
* **中间的：**
中间的操作保持流打开状态，并允许后续的操作，e.g. filter() 和 map() 方法就是中间的操作。这些操作的返回数据类型还是流，返回当前的流以便串联更多的操作。中间操作是延迟（lazy）的；
* **末端的：**
末端的操作必须是对流的最终操作，当一个末端操作被调用，流被“消耗”并且不可再使用。eg：sum() 方法就是一个末端操作。末端操作会立即开始流中元素的处理。

### 流操作特性：
* 有状态的：有状态的操作给流增加了一些新的属性，如：元素的唯一性/元素的最大数量/保证元素以排序的方式被处理。这导致比无状态的中间操作代价更大；
* 短路：短路操作允许对流的操作尽早停止，而不去检查所有的元素（这跟 && 条件判断类似）。这是对无限流特殊设计的一个属性，如果对流的操作没有短路，那么代码可能无法终止；
* 中间的操作（API 方法）；
* 末端的操作（API 方法）。

### 处理一个流的基本步骤：
{% asset_img JDK1.8-Stream通用语法.png JDK1.8-Stream通用语法 %}
#### 1. 创建 Stream
常用途径有两种：
1. 通过 Stream 接口的静态工厂方法：
　　三种方式：
　　a. **of() 方法**
　　　　接受单一值参数：
　　 　　```java
　　 　　static <T> Stream<T>of(T t)：Stream<String> stringStream = Stream.of("lazysnail");
　　 　　```
　　 　　接受变长参数：
　　 　　```java
　　 　　static <T> Stream<T>of(T...values)：Stream<Integer> integerStream = Stream.of(1, 3, 5, 7);
　　 　　```
　　b. **generator() 方法**
　　　　生成一个无限长度的 Stream，其元素的生成是通过给定的 Supplier（该接口可以看作一个对象的工厂，每次调用返回一个给定类型的对象）：
　　　　```java
　　　　static <T> Stream<T>generate(Supplier<T> s);  e.g.
　　　　// 普通表现形式
　　　　Stream.generate(new Supplier<Double>() {
　　　　    @Override
　　　　    public Double get() {
        　　　　return Math.random();
    　　　　}
　　　　});
　　　　// lambda 表达式形式
　　　　Stream.generate(() -> Math.random());
　　　　// 方法引用表达形式
　　　　Stream.generate(Math::random);
　　　　```
　　　　以上三种方式作用相同，懒加载生成无限长度的流，一般会配合 limit() 方法触发短路。
　　c. **iterate() 方法**
　　　　同样生成无限长度的 Stream，和 generator 不同的是，其元素的生成是重复对给定的种子值（seed）调用指定函数来生成的，元素可以认为是：seed, f(seed), f(f(seed)), ...
　　　　```java
　　　　static <T> Stream<T>iterate(T seed, UnaryOperator<T> f); e.g.
　　　　// 先获取一个无限长的正整数集合 Stream，然后打印前 10 个：
　　　　Stream.iterate(1, item -> item + 1).limit(10).forEach(System.out::println);
　　　　```
　　　　同样需要配合 limit() 方法。

2. 通过 Colletcion 接口的默认方法 stream()，把一个 Collection 对象转换成 Stream。
　　Collection 接口有个 stream 方法（Since 1.8），其所有实现类都可以获取对应的 Stream 对象：
　　　　```java
　　　　Stream<T> stream = collection.stream();
　　　　```

#### 2. 转换 Stream
**执行一个/多个中间操作，每次转换都不改变原有 Stream，而是返回一个新的 Stream 对象**
* **中间的操作（API 方法）：**
　　distinct()：根据 .equals() 的行为排除所有重复元素，有状态的操作；
　　{% asset_img JDK1.8-Stream之distinct.png distinct %}
　　filter()：过滤所有与断言不匹配的元素；
　　{% asset_img JDK1.8-Stream之filter.png filter %}

　　map()：通过 Function 对元素执行一对一的转换。有 3 个变种：mapToInt()、mapToLong()、mapToDouble()，即把原始 Stream 转换成一个对应类型的新 Stream，可免除自动装箱/拆箱的额外消耗；
　　{% asset_img JDK1.8-Stream之map.png map %}

　　flatMap()：通过 FlatMapper 将每个元素转变为无或更多的元素，即，每个元素都转换为一个新 Stream 对象，再将子 Stream 的元素压缩到父集合中；
　　{% asset_img JDK1.8-Stream之flatMap.png flatMap %}

　　limit()：截断，保证后续的操作所能看到的元素的最大数量，有状态的短路操作；
　　{% asset_img JDK1.8-Stream之limit.png limit %}

　　skip()：返回一个丢弃原 Stream 前 N 个元素后剩下的元素组成的新 Stream；
　　{% asset_img JDK1.8-Stream之skip.png skip %}

　　peek()：生成一个包含原 Stream 的新 Stream，同时提供一个消费函数（Consumer 实例），新 Stream 每个元素被消费的时候都执行给定的消费函数。主要用于调试；
　　{% asset_img JDK1.8-Stream之peek.png peek %}

　　sorted()：确保流中的元素在后续的操作中，按照比较器（Comparator）决定的顺序访问，有状态的操作；
　　substream()：确保后续的操作只能看到一个范围的（根据 index）元素。有两种形式：有开始索引的、有结束索引的，二者都是有状态的操作，有结束索引的同时也是短路操作。

**性能问题**
常规操作可能是：如对 Iterable 集合(N 个元素)进行一系列操作，每次都对每个元素进行相应操作，这样的遍历需要 k 次（k 为对每个元素操作的次数），即，时间复杂度为 O(Nk)。而，Stream 提供的转换操作则不同：Stream 的中间转换操作都是 lazy 的：**多个转换融合到聚合阶段（reduce/fold）通过一次循环完成。i.e. Stream 里有个操作函数的集合，每个转换操作就是把转换函数放入到这个集合中，在聚合操作的时候，循环这个集合，对每个元素执行该集合操作。**


#### 3. 聚合/折叠（Reduce/Fold）
**执行一个末端操作，i.e. 对 Stream 进行聚合操作，获得结果**。接受一个元素序列作为输入，反复使用某个聚合操作，把序列中的元素聚合成一个总的结果。e.g. 查找一个数字列表的总和或最值、把这些数字累积成一个 List 对象等。
* Stream 接口通用的聚合操作：reduce()、collect()等；
* 特定用途的聚合操作：sum()、count()等；
Note：sum() 方法只有 IntStream、LongStream、DoubleStream 的实例才拥有。

**末端的操作（API 方法）：**
　　forEach()：对流中每个元素执行一些操作；
　　toArray()：将流中的元素倾倒入一个数组；
　　reduce()：通过一个二进制操作将流中的元素合并到一起；
　　collect()：将流中的元素倾倒入某些容器，e.g. 一个 Collection 或 Map；
　　min()：根据一个比较器找到流中元素的最小值；
　　max()：根据一个比较器找到流中元素的最大值；
　　count()：计算流中元素的数量；
　　anyMatch()：判断流中是否 **至少有一个元素** 匹配断言，短路操作；
　　allMatch()：判断流中是否 **每一个元素都** 匹配断言，短路操作；
　　noneMatch()：判断流中是否 **没有任何一个元素** 匹配断言，短路操作；
　　findFirst()：查找流中的第一个元素，短路操作；
　　findAny()：查找流中任意元素，对某些流代价可能比 findFirst 低，短路操作。

**聚合分类**
**可变聚合**：把输入的元素累积到一个可变的容器中，e.g. Collection、StringBuilder 等。
collect()方法可以把Stream中的所有元素收集到一个结果容器中（e.g. Collection）
```java
// Supplier supplier是一个工厂函数，用来生成一个新的容器；BiConsumer accumulator也是一个函数，
// 用来把Stream中的元素添加到结果容器中；BiConsumer combiner还是一个函数，
// 用来把中间状态的多个结果容器合并成为一个（并发的时候会用到）。
<R, A> Rcollect(Collector<? super T, A, R> collector)
<R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner)

// e.g. 对一个元素是Integer类型的List，先过滤掉全部的null，然后把剩下的元素收集到一个新的List中。
List<Integer> nums = Lists.newArrayList(1, 1, null, 2, 3, 4, null, 5, 6, 7, 8, 9, 10);
List<Integer> numsWithoutNull = nums.stream().filter(num -> num != null).collect(
        () -> new ArrayList<Integer>(), (list, item) -> list.add(item), (list1, list2) -> list1.addAll(list2)
    );
    // Note:
    // 第一个函数生成一个新的ArrayList实例；
    // 第二个函数接受两个参数，第一个是前面生成的ArrayList对象，
    // 第二个是stream中包含的元素，函数体就是把stream中的元素加入ArrayList对象中。
    // 第二个函数被反复调用直到原stream的元素被消费完毕；
    // 第三个函数也是接受两个参数，这两个都是ArrayList类型的，
    // 函数体就是把第二个ArrayList全部加入到第一个中。
```

**其他聚合**：
reduce()、count()、sum()等方法。

reduce()方法有三种形式：
```java
Optional<T> reduce(BinaryOperator<T> accumulator)
T  reduce(T identity, BinaryOperator<T> accumulator)
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)

// e.g. 一个参数的reduce方法：
List<Integer> ints = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
System.out.println("ints sum is: " + ints.stream().reduce((sum, item) -> sum + item).get());
// 这个函数有两个参数，第一个参数是上次函数执行的返回值（也称为中间结果），
// 第二个参数是stream中的元素，这个函数把这两个值相加，
// 得到的和会被赋值给下次执行这个函数的第一个参数。
// 第一次执行的时候第一个参数的值是Stream的第一个元素，第二个参数是Stream的第二个元素。
// 返回类型是Optional，这是Java8防止出现NPE的一种可行方法。

// e.g. 两个参数的reduce方法：
List<Integer> ints = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
System.out.println("sum is: " + ints.stream().reduce(0, (sum, item) -> sum + item));
// 它允许用户提供一个循环计算的初始值，如果Stream为空，就直接返回该值。
// 这个方法不会返回Optional，因为其不会出现null值。

// e.g. count()方法示例：
List<Integer> ints = Lists.newArrayList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
System.out.println("sum is:" + ints.stream().count());
```

### Stream 示例
生成斐波那契数列示例（利用Stream API，可以设计更加简单的数据接口。）
```java
// 生成斐波那契数列，完全可以用一个无穷流表示
// 受限long型大小，可以改为BigInteger。
class FibonacciSupplier implements Supplier<Long> {
    long a = 0;
    long b = 1;

    @Override
    public Long get() {
        long x = a + b;
        a = b;
        b = x;
        return a;
    }
}

public class FibonacciStream {
    public static void main(String[] args) {
        Stream<Long> fibonacci = Stream.generate(new FibonacciSupplier());
        fibonacci.limit(10).forEach(System.out::println);

        //如果想取得数列的前10项，用limit(10)，如果想取得数列的第20~30项,用skip()，
        //通过collect()方法把Stream变为List。该List存储的所有元素就已经是计算出的确定的元素了.
        List<Long> list = fibonacci.skip(20).limit(10).collect(Collectors.toList());
    }
}
```

### Stream 的串行和并行
一个流就像一个地带器，这些值流过（遍历）的过程中接受某些处理。流支持串行或并行，也可以从一种方式切换到另一种方式，使用 stream.sequential() 切换串行、stream.parallel() 切换并行，串行流在一个线程上连续操作，并行流可能出现在多个线程上。在并行化一个流前，需要考虑很多特性，关于流、它的操作以及数据的目标方面等。e.g. 访问顺序确实对我有影响吗？我的函数是无状态的吗？我的流有足够大，并且我的操作有足够复杂，这些能使得并行化是值得的吗？

### Stream vs Collection
* Collection 是关于静止的数据结构，而 Stream 是有关动词算法和计算的。
* Collection 是主要面向内存，存储在内存中；Stream 主要是面向 CPU，通过 CPU 实现计算。

**为什么不在集合类实现元素迭代等操作，而是定义了全新的Stream API？**
Oracle官方给出的解释：
1. 集合类持有的所有元素都是存储在内存中的，非常巨大的集合类会占用大量的内存，而Stream的元素却是在访问的时候才被计算出来，这种“延迟计算”的特性有点类似Clojure的lazy-seq，占用内存很少。
2. 集合类的迭代逻辑是调用者负责，通常是for循环，而Stream的迭代是隐含在对Stream的各种操作中，e.g. map()。

要理解“延迟计算”，不妨创建一个无穷大小的Stream：如果要表示自然数集合，显然用集合类是不可能实现的，因为自然数有无穷多个。但是Stream可以做到：
```java
/**
 * 自然数集合的规则非常简单，每个元素都是前一个元素的值+1。
 * 反复调用get()，将得到一个无穷数列，利用这个Supplier，可以创建一个无穷的Stream
 */
class NaturalSupplier implements Supplier<Long> {
    long num = 0;

    public Long get() {
        this.num = this.num + 1;
        return this.num;
    }
}

public class NaturalNum {
    public static void main(String[] args) {
        /**
         * 对这个Stream做任何map()、filter()等操作都是完全可以的，这说明Stream API对Stream进行转换并
         * 生成一个新的Stream并非实时计算，而是做了延迟计算。
         * 当然，对这个无穷的Stream不能直接调用forEach()，这样会无限打印下去。但是我们可以利用limit()变换，
         * 把这个无穷Stream变换为有限的Stream。
         */
        Stream<Long> natural = Stream.generate(new NaturalSupplier());
        natural.map((x) -> {
            return x * x;
        }).limit(10).forEach(System.out::println);
    }
}

```

[Stream 类](https://blog.csdn.net/sun_promise/article/details/51480257)
