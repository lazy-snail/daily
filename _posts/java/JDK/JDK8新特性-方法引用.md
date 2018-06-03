---
title: Java-JDK8新特性-方法引用
date: 2018-04-28 22:49:33
categories: java
tags: [java, 新特性]
---
### 方法引用(Method Reference)
#### 简介
用来直接访问类或实例的已经存在的方法或构造函数，提供了一种引用而不执行方法的方式，它需要由兼容的函数式接口构成的目标类型上下文，计算时，方法引用会创建函数式接口的一个实例。当 Lambda 表达式只是执行一个方法调用时，不用 Lambda 表达式而直接通过方法引用的形式能提高可读性。**方法引用是一种更简洁易懂的 Lambda 表达式**。
可以认为：
>  方法引用的唯一用途是支持 Lambda 的简写：提高可读性，使得逻辑更加清晰。

#### 基本语法
对象名/类名 :: 方法名
eg:

|           Lambda表达式          |          对应的方法引用        |
| -----------------------------------  |  --------------------------------- |
| x -> String.valueOf(x)       | String::valueOf                |
| x -> x.toString()                | Object::toString               |
| () -> x.toString()                | x::toString                       |
| () -> new ArrayList<> ()   | () -> new ArrayList<>()   |

#### 分类
1. 静态方法引用

格式：
className::staticMethodName

```java
interface StringFunc {
    String func(String str);
}

class StringOps {
    // 静态方法，反转字符串
    public static String strReverse(String str) {
        String res = "";
        for (int i = str.length() - 1; i >= 0; i--) {
            res += str.charAt(i);
        }
        return res;
    }
}

public class MethodRef {
    public static String stringOps(StringFunc sf, String s) {
        return sf.func(s);
    }

    public static void main(String[] args) {
        String str = "Static Method Reference.";
        // StringOps::strReverse 相当于实现了接口方法 func(),
        // 并在接口方法中使用了 StringOps.strReverse() 操作
        String res = stringOps(StringOps::strReverse, str);
        System.out.println("Origin str: " + str);
        System.out.println("str reserved: " + res);
    }
}
```
2. 实例方法引用

分为实例上的实例方法引用、超类上的实例方法引用、类型上的实例方法引用。
上述代码，如果反转字符串方法不是静态方法，而是普通方法，则就需要 StringOps 实例化一个对象(strOps)，并通过该对象调用：
    String res = StringOps(strOps::strReverse, str);

3. 构造方法引用

分为 构造方法引用（构造器引用）、数组构造方法引用。
