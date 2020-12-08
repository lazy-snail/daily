---
title: Java-JDK8新特性-Optional
date: 2018-04-28 22:49:33
categories: java
tags: [java, 新特性]
---
[toc]
## Optional
### 简述
字面意思是“可选的”，这里的语义是“指某个值可能有也可能没有（null）”。是为了解决 NullPointerException 空指针异常，通过使用检查空值的方式来防止代码污染。本身是一个容器，其值可能是 null 或者不是 null，java 1.8 之前一般某个函数应该返回空对象但偶尔也可能返回 null，而 java 1.8 开始，推荐返回 Optional 而非 null。

**较之 JavaDocs、@NotNull，本质上，Optional是把处理返回值为空的情况这一责任强加给调用者，即调用者必须考虑这一情况（以避免NPE）。**

#### 优点
* 显式提醒开发者需要关注 null 的情况，是一种字面上的约束；
* 将平时的一些显式的防御性检测标准化，并提供一些可串联操作；
* 解决 null 导致疑惑的概念，e.g. Map 里的 key==null 的情况，value==null 的情况。

### 使用
为了防止抛出 java.lang.NullPointerException 异常，一般写法：
```java
Student student = person.find("Neil");  
        if (student != null) {  
            student.doSomething();  
        }  
```
使用Optional的代码写法：
```java
Student student = person.find("Neil");  
        if (student.isPresent()) {  
            student.get().doSomething();  
        }  
```
如果 isPresent() 返回false，说明这是个空对象；否则，就可以把其中的内容取出来做相应的操作。单从代码量上来说，Optional的代码量并未减少，甚至比原来的代码还多。优点在于不会忘记判空，因为这里得到的不是Student类的对象，而是Optional 类对象。

### 实现的方法
#### 3 种构造方法
* Optional.of(obj)：要求传入的 obj 不能是 null 值，通过工厂方法创建一个 Optional 类；
　　使用：
　　当明确将要传给 of() 的 obj 参数不可能为 null 时，e.g. 传入刚 new 出来的对象、非 null 常量时；
　　当需要断言 obj 为null 则立即报告 NPE 而不是隐藏空指针异常时；
　　i.e. 避免不可预计的 null 传入 Optional。
* Optional.ofNullable(obj)：of方法相似，唯一的区别是可以接受参数为null的情况，此时返回 Optional.empty()，否则返回 Optional.of()；
* Optional.empty()：返回一个空Optional实例。

[用法举例](https://blog.csdn.net/sun_promise/article/details/51362838)
[用法举例2](https://blog.csdn.net/sun_promise/article/details/51362838)

 使用 Optional 时尽量不直接调用 Optional.get() 方法, Optional.isPresent() 更应该被视为一个私有方法, 应依赖于其他像 Optional.orElse(), Optional.orElseGet(), Optional.map() 等这样的方法。

[How to use Optionals In Java](https://dzone.com/articles/optional-in-java)