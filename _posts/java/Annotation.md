---
title: Annotation
date: 2018-06-13 21:35:32
categories: java
tags: [java]
---
[toc]
# Annotation 
注解是 JDK5 引入的新特性，它与普通的修饰符如 public、static、void 等的使用上没有什么区别。它们的出现代表某种 **配置语义**，使得源代码中不但可以包含功能性的实现代码，还可以添加元数据。注解已经在很多框架中得到了广泛的使用，用来简化程序中的配置。

## 使用
实例解释：
```java 注解的简单使用
public class AnnotationDemo {
    // @Test注 解修饰方法 A
    @Test
    public static void A(){
        System.out.println("Test.....");
    }
    // 一个方法上可以拥有多个不同的注解
    @Deprecated
    @SuppressWarnings({"uncheck", "unused"})
    public static void B(){
    }
}
```
通过在方法上使用 @Test 注解，在运行该方法时，测试框架会自动识别该方法并单独调用。@Test 实际上是一种标记注解，起到标记的作用，运行时告诉测试框架该方法为测试方法。对于 @Deprecated 和 @SuppressWarnings，则是 java 内置的注解，前者表明该方法/类已经过期或不建议再使用，后者表示忽略指定警告，括号里表示该注解可供配置的值，用花括号表示数组，配置参数值必须是编译时常量。


# 架构
{% asset_img Annotation架构.jpg Annotation 架构 %}
可见，
* 1 个 Annotation 和 1 个 RetentionPolicy 关联。i.e. 每个 Annotation 对象，都会有唯一的 RetentionPolicy 属性；
* 1 个 Annotation 和 1~n 个 ElementType 关联。i.e. 对于每个 Annotation 对象，可以有若干个 ElementType 属性；
* Annotation 有许多实现类，包括：Deprecated, Documented, Inherited, Override 等。每个实现类，都“和 1 个 RetentionPolicy 关联”并且“和 1~n 个 ElementType 关联”。


# 组成
Annotation 组成中有 3 个主干类：
## Annotation.java
它是一个接口：
```java
public interface Annotation {
    boolean equals(Object obj);
    int hashCode();
    String toString();
    Class<? extends Annotation> annotationType();
}

```



http://www.infoq.com/cn/articles/cf-java-annotation
https://blog.csdn.net/javazejian/article/details/71860633
https://www.cnblogs.com/skywang12345/p/3344137.html