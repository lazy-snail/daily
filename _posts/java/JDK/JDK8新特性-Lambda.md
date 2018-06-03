---
title: JDK8新特性-Lambda
date: 2018-04-28 22:49:33
categories: java
tags: [java, 新特性]
---
### Lambda 表达式
#### 简介
允许把函数作为一个方法的参数（传递函数），或者说把代码当作数据。本质上是语法糖方式实现的匿名方法，但这个方法不能独立执行，而要用于实现函数式接口的方法，即，使用 Lambda 表达式实例化函数式接口。因此，Lambda 表达式会导致产生一个匿名类。
具体过程是，Lambda 表达式构成了一个函数式接口中（唯一）抽象方法的实现，该函数式接口定义了 Lambda 表达式的目标类型（抽象方法的具体定义）。所以，只有在定义了 Lambda 表达式的目标类型的上下文中，即，该上下文中可以使用相应的接口，才能使用该 Lambda 表达式。当目标类型上下文中出现 Lambda 表达式时，就会自动创建实现了函数式接口的一个类的实例（类似于匿名类），而函数式接口的抽象方法的行为则有 Lambda 表达式定义。通过目标调用该方法时，就会执行 Lambda 表达式。
为了在目标类型上下文中使用 Lambda 表达式，抽象方法的类型和 Lambda 表达式的类型必须兼容。eg：如果抽象方法指定了两个 int 型参数，相应的 lambda 表达式的形式就应该是
* 上下文能够推断参数类型时： (x, y) -> {};
* 上下文无法推断参数类型时： (int x, int y) -> {};
    _如果显式声明一个参数的类型，那么同时必须提供其他剩余参数的类型。_

即，**Lambda 表达式的参数的类型和数量、返回类型必须和相应的函数式接口的抽象方法兼容，并且 Lambda 表达式可能抛出的异常必须也能被该抽象方法接受**。

_**闭包**_
函数式语言提供的一种强大的功能：是一个可调用的对象，它记录了一些信息，这些信息来自于创建它的作用域。

**基本语法**
    (parameters) -> expression
或
    (parameters) -> {statements;}

#### 使用
和匿名内部类还是有些不同的：匿名内部类只能引用作用域外的 final 修饰的变量（见语法糖部分），而 lambda 表达式则削弱了这一限制：只要是“等效于 final”的变量即可，而不一定非要 final 修饰，也可以认为是隐含转换成了 final 修饰：
```java
String str = “Hello.”;
Runnable runnable = () -> System.out.println(str);
```
[Lambda 表达式使用示例](https://blog.csdn.net/sun_promise/article/details/51121205)
[Lambda 表达式作用域](https://blog.csdn.net/sun_promise/article/details/51132916)
