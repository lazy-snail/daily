---
title: Annotation
date: 2018-06-13 21:35:32
categories: java
tags: [java]
---
[toc]
# Annotation 
注解（元数据）是 JDK5 引入的新特性，提供用来完整地描述程序所需的信息，而这些信息是无法用 java 来表达的。使用上，除了“@”符号外，它与普通的修饰符如 public、static、void 等没有什么区别。它们的出现代表某种 **配置语义**，使得源代码中不但可以包含功能性的实现代码，还可以添加元数据。注解已经在很多框架中得到了广泛的使用，用来简化程序中的配置。注解也将编译成 class 文件。

## 内置注解
目前 JDK10 内置了以下注解。
**标准注解**：
* @Override：表示当前的方法将覆盖父类中相同签名的方法。如果不慎拼写错误或者方法签名不一致，编译器会发出错误提示。
* @SuppressWarnings：关闭不当的编译器警告。
* @Deprecated：表示不再被推荐使用，继续使用的话编译器会发出警告，未来版本中可能移除。
* @SafeVarargs：参数安全类型注解。提醒用户不要用参数做一些不安全的操作，会阻止编译器产生 unchecked 警告。JDK1.7
* @FunctionalInterface：函数式接口注解，鉴于函数式编程的兴起而添加，只起到文档作用，表明这是个函数式接口——可以很容易地转成 Lambda 表达式。比如线程开发中常用的 Runnable 接口就是一个典型的函数式接口。JDK1.8。

**元注解**：
* @Target：定义注解将应用于什么地方：如方法/域。
* @Retention：定义注解应用于什么级别，可选参数：SOURCE（源代码）：将被编译器丢弃、CLASS（类文件）：在 class 文件中可用，但会被 VM 丢弃、RUNTIME（运行时）：VM 在运行期也会保留该注解，因此可以通过反射机制读取注解的信息。
* @Documented：将此注解包含在 javadoc 中。
* @Inherited：允许子类继承父类中的注解。
* @Repeatable：可重复。JDK1.8。

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

###应用场景
注解应用场景很多，主要给编译器及工具类型软件使用，如很多框架（Spring、Junit 等）。

## 定义注解
使用 @interface 关键字定义：
```java
public @interface AnnotationName {}
```
定义注解时，需要用到一些元注解（meta-annotation），如 @Traget（定义注解将应用于什么地方：如方法/域）、@Retention（定义注解应用于什么级别，如 SOURCE 即源代码、CLASS 即类文件、RUNTIME 即运行时）等。一般还会包含一些值，就像接口的方法，并且可以提供默认值，而没有元素的注解称为 **标记注解**。

### 元注解
基本注解，能够应用到其他注解上，可以理解为注解到注解上的注解。

### 注解的属性
也称成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface AnnotationName {
    int id();
    String msg() default "No message.";
}
```
该注解有两个属性：id、msg，并且 msg 定义了默认值，如果没有默认值则需要显式给出。特别地，如果注解只有 1 个属性，那么在给出值时可以不指明属性名；没有属性时，可以直接给出注解名，不需要括号。



# 架构
{% asset_img Annotation架构.jpg Annotation 架构 %}
可见，
* 1 个 Annotation 和 1 个 RetentionPolicy 关联。i.e. 每个 Annotation 对象，都会有唯一的 RetentionPolicy 属性；
* 1 个 Annotation 和 1~n 个 ElementType 关联。i.e. 对于每个 Annotation 对象，可以有若干个 ElementType 属性；
* Annotation 有许多实现类，包括：Deprecated, Documented, Inherited, Override 等。每个实现类，都“和 1 个 RetentionPolicy 关联”并且“和 1~n 个 ElementType 关联”。


## 组成
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
https://blog.csdn.net/briblue/article/details/73824058