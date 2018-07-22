---
title: java-String
date: 2018-04-26 22:55:01
categories: java
tags: [java]
---
### == vs equals() 比较问题
java 中，equals()方法被认为是对对象的值进行深层次的比较，而“==”是进行浅层次的比较：
**==**
* 当比较对象是值类型（如基本数据类型）时，对比的是数据的值，此时效果等同于 equals()；
* 当比较对象是对象引用时，只要左右两边引用指向同一个地址，才为真，否则，即使两个对象的内容完全一样，也为假。

**equals()**
equals() 比较两个对象的内容，相同返回真。
在 String 中，按照如下顺序工作（其他对象也类似）：
1. 比较引用，如果相同，返回 true；
2. 比较类型，如果类型不同，返回 false；
3. 比较长度，不等时，返回 false；
4. 逐字符比较两个字符串，遇到不同，返回 false；
5. 不满足 2-4，返回 true。

要注意的是，equals()是在 Object 类中的，而 java 中“一切皆是对象”，所以，如果被比较的对象没有覆盖 Object 类中的 equals()的话，必然返回真，这与事实可能不符。而 String 等类型都已经重写了 equals() 方法。
另外，jls 规定了 equals()遵循的几个规则：
1. 自反性：对于任何非空引用值 x，x.equals(x) 都应返回 true。
2. 对称性：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。
3. 传递性：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。
4. 一致性：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。
5. 对于任何非空引用值 x，x.equals(null) 都应返回 false。

### 字符串常量池
字符串的分配，和其他的对象分配一样，耗费高昂的时间与空间代价。JVM 为了提高性能和减少内存开销，在实例化字符串常量的时候进行了一些优化。为了减少在 JVM 中创建的字符串的数量，字符串类维护了一个字符串池，每当代码创建字符串常量时，JVM 会首先检查字符串常量池。如果字符串已经存在池中，就返回池中的实例引用。如果字符串不在池中，就会实例化一个字符串并放到池中。Java 能够进行这样的优化是因为字符串是不可变的，可以不用担心数据冲突进行共享。这是 java 节约资源的一种方式。从 JVM 角度讲，它属于方法区的运行时常量池的一部分。
字符串常量总是指向字符串池中的一个对象，而通过 new 操作符创建的字符串对象不指向字符串池中的任何对象，但是可以通过使用字符串的 intern() 方法来指向其中的某一个。
有两种方式进入 String 常量池：
* 编译期：通过双引号声明的常量（显示声明、静态编译优化后的常量等），JIT 优化也可能产生一些；
* 运行期：调用 String 的 intern()方法，可能会将该 String 对象动态地写入。

方式1 的行为是明确的，从 class 文件结构、类加载、编译期及运行期优化等过程都可以很明确地得知。而方式2 在不同 JDK 甚至有不同行为。

### 两种创建方式
注意到以下代码结果：
{% codeblock Main.java lang:java  %}
public class Main {
    public static void main(String[] args) {
        String s1 = new String("123");
        String s2 = new String("123");

        String s3 = "123";
        String s4 = "123";

        System.out.println(s1 == s2);
        System.out.println(s3 == s4);
    }
}
------- output: -------
false
true
{% endcodeblock %}
s1、s2 是 java 标准的对象创建方式，每调用一次就会在堆上创建一个新的对象，并指向它，同时创建一个相同内容的 String 对象存储在常量池中，如果常量池已存在则不重复创建（事实上 String 常量池只保存一份相同内容的String）；而 s3、s4 则是在栈中创建对象引用变量 str，然后查看 String 常量池中是否存在含有该内容的 String 对象，如果存在，则令 str 直接指向它，如果不存在，则先在池中创建该字符串，然后令 str 指向它。这样充分利用了栈的数据共享的优点。
故，s1 和 s2 分别指向不同的两个对象，s3 和 s4 指向的则是同一个地址。考虑到前述事实，如果用 equals() 比较 s1 和 s2 的话，将返回 true。

注意到以下代码：
```java
public class Main {
    public static void main(String[] args) {
        String s1 = "123";
        String s2 = "12";
        final String s3 = "12";
        String s4 = "3";

        String s5 = "12" + "3";
        String s6 = s2 + s4;
        String s7 = s3 + s4;
        String s8 = s3 + "3";
        String s9 = new String("12") + "3";

        System.out.println(s1 == s5);
        System.out.println(s1 == s6);
        System.out.println(s1 == s7);
        System.out.println(s1 == s8);
        System.out.println(s1 == s9);
    }
}
------- output: -------
true
false
false
true
false
```
这里引出以下问题：

### 字符串拼接方法
**append()方法（StringBuilder/StringBuffer）**
源码（手动添加注释）：
```java
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    // 追加后的字符数组长度是否超过当前值
    ensureCapacityInternal(count + len);
    // 复制到目标数组
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        // 加长并拷贝
        value = Arrays.copyOf(value, newCapacity(minimumCapacity));
    }
}
```
可以看出，整个 append()方法是通过字符数组的加长、拷贝等处理完成的，没有生成中间对象，只有最后用 toString()返回一个 String 对象。速度最快。

**concat()方法**
继续看源码：
```java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    // 获取原字符串数组长度
    int len = value.length;
    // 将原字符串的字符数组拷贝到 buf 数组
    char buf[] = Arrays.copyOf(value, len + otherLen);
    // 追加的字符串转化为字符数组，添加到 buf 中
    str.getChars(buf, len);
    // 产生一个新的对象
    return new String(buf, true);
}
```
整体是一个数组的拷贝，在内存中是原子操作，速度也很快，但最后返回语句会新建一个对象，这对速度有一定影响。

** “+” **
虽然编译器对“+”做了优化，会使用 StringBuilder 的 append()方法进行追加，但它最终会通过 toString()方法转换成 String 字符串，速度比较慢。其过程为：
1. 创建 StringBuilder 对象；
2. 执行完毕调用 toString() 方法。

具体情况分为：
如果加号两边都是字面量（或等效于字面量），形如
String str = "abc" + "123";
此时的处理方式是直接将 str 作为一个对象处理，相当于先处理右边的加号，再进行创建，参考上面的“=”创建规则：
String str = "abc123";
否则，形如：
String str = "abc";
str += "123";
此时执行情况则明显不同，相当于将 str 内容取出，再与第二个字符串进拼接，返回新生成的字符串，注意这里有新建动作，所以创建规则相当于使用“new”：
String str = new StringBuilder(str).append("123").toString();
即，执行完毕会有 3 个对象存在。
**而 final 相当于宏，编译器在编译期会把所有用 final 定义的变量直接用定义值替换掉（宏替换）**。
_这就解释了以上代码的结果。_

**使用场景：**
* 大多数情况，使用“+”，它符合编码习惯以及可读性较好；
* 当频繁进行字符串的运算（拼接、替换、删除等）时，或在系统性能临界的时候，应考虑使用 append() 和 concat()。

### String、StringBuffer、StringBuilder
简单讲，
** String：字符串常量（String 为 final 修饰类）；
StringBuffer：字符串变量，线程安全。
StringBuilder：字符串变量，非线程安全；**

**StringBuffer（Since JDK1.0）**
线程安全的可变字符序列，在任意时间点上它都包含某种特定的字符序列，但通过某些方法调用（如 append()、insert()等）可以改变序列的内容和长度。可将该字符串缓冲区安全地用于多个线程。观察源码可见，为提供线程安全，其内部实现中大部分方法都是 synchronized 修饰的。

**StringBuilder（Since JDK5.0）**
可变字符序列。提供与 StringBuffer 兼容的 API，不过不保证同步，被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的场景（很普遍），大多数情况下，它提供更快的执行效率。

**在频繁对字符串内容进行修改的场景，根据对线程安全性的需求，优先考虑 StringBuilder、StringBuffer 中的一种。**

### 关于 String 的 intern()方法
上面提到，inter()方法可能导致将 String 对象写入 String 常量池。从 JDK 7 开始，常量池 从 PermGen 区移到了 java 堆（jvms 角度讲，应该是从 方法区 移到了 java 堆，PermGen 是 HotSpot 里的概念），语义上的变化为：
* 调用 intern()方法时，如果堆中有该字符串而常量池中没有，则直接在常量池中保存堆中对象的引用，而不是在常量池中新建对象。
