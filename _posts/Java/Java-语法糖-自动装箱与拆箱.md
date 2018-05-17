---
title: Java-语法糖-自动装箱与拆箱
date: 2018-04-26 19:19:26
tags: [Java]
---
### 装箱与拆箱
#### 概念
Java 为每种基本数据类型提供了对应的包装器类型，形如
Integer i = 10;
这个过程中会自动根据数值创建对应的 Integer 类对象，即，装箱。
对应地，自动将包装器类型转换为基本数据类型的过程，就是拆箱：
int n = i;

即，装箱就是自动将基本数据类型转换为包装器类型，拆箱就是将包装器类型转换为基本数据类型。

#### 实现
以 Integer 为例，自动装箱调用的是 Integer 的 valueOf(int)方法，拆箱时自动调用的是 Integer 的 intValue()方法。其他的类似。

几个容易出错的例子：
1. **新建对象还是引用已存在对象？**

```java
public class Main {
    public static void main(String[] args) {

        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;

        System.out.println(i1==i2);
        System.out.println(i3==i4);
    }
}
------- output: -------
true
false
```
输出结果表明i1和i2指向的是同一个对象，而i3和i4指向的是不同的对象。通过查看源码 ↓↓↓
这段是 valueOf()方法具体实现：
```java
public static Integer valueOf(int i) {
        if(i >= -128 && i <= IntegerCache.high)
            return IntegerCache.cache[i + 128];
        else
            return new Integer(i);
}
```
而其中IntegerCache类的实现为：
```java
private static class IntegerCache {
        static final int high;
        static final Integer cache[];

        static {
            final int low = -128;

            // high value may be configured by property
            int h = 127;
            if (integerCacheHighPropValue != null) {
                // Use Long.decode here to avoid invoking methods that
                // require Integer's autoboxing cache to be initialized
                int i = Long.decode(integerCacheHighPropValue).intValue();
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - -low);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
    }
```
可以看出，在通过 valueOf()方法创建 Integer 对象的时候，如果数值在 [-128, 127] 之间，则返回指向 IntegerCache.cache 中已经存在的对象的引用，否则创建一个新的 Ingeter 对象。故，i1 和 i2 会直接从 cache 中取出已经存在的对象，所以它们指向的是同一个对象；而 i3 和 i4 则是新创建的两个不同对象。类似的还有 Long、Short、Character、Byte，但 Double、Float、Boolean 则直接返回新创建的对象。

2. **它们相等吗？**
注意以下代码：
```java
public class Main {
    public static void main(String[] args) {

        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Long g = 3L;
        Long h = 2L;

        System.out.println(c==(a+b));
        System.out.println(c.equals(a+b));
        System.out.println(g==(a+b));
        System.out.println(g.equals(a+b));
        System.out.println(g.equals(a+h));
    }
}

------- output: -------
true
true
true
false
true
```

这里需要注意以下事实：
* 当 “==”运算符的两个操作数都是包装器类型的引用，则是比较指向的是否是同一个对象；如果其中有一个操作数是表达式（即包含算术运算），则比较的是数值（即会触发自动拆箱的过程）；
* 对于包装器类型，equals方法不会进行类型转换。

其中，c.equals(a+b)会先触发自动拆箱，再触发自动装箱：即，a、b 先各自调用 intValut()方法得到值进行加运算，然后调用 Integer.valueOf()将结果装箱，再与 c 比较。
_可尝试反编译字节码查看相关内容。_

#### 自动装箱与直接新建的区别
即，以下两种创建方式的区别：
```java
Integer i = new Integer(x);
Integer i = x;
```
主要表现在：
* 第一种方式不会触发自动装箱的过程，第二种会；
* 执行效率和资源占用上：一般情况下，第二种方式优于第一种。
