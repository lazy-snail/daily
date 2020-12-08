---
title: java
date: 2018-06-03 15:15:38
categories: java
tags: [java, Q&A]
---
[toc]

# 位操作
## 移位运算符
<<：  左移运算符，num << 1，相当于 num 乘以 2；
\>>： 右移运算符，num >> 1，相当于 num 除以 2；
\>>>：无符号右移，忽略符号位，空位都以 0 补齐。

# 数组拷贝方式
## for 遍历
语言层面使用循环遍历的方式依次拷贝每个元素，如果没有编译器优化，它对应的就是遍历数组操作的字节码，执行引擎就根据这些字节码循环获取数组的每个元素再执行拷贝操作。

## System.arraycopy()
这是一个本地方法：
```java
@HotSpotIntrinsicCandidate
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos, int length);
```
如果数组比较大，那么使用 System.arraycopy() 会比较有优势，因为其使用的是 **内存复制，省去了大量的数组寻址访问等时间**。

## Arrays.copyOf()
该方法对不同数据类型都有相应的重载。观察它们的源码可见，其内部时通过调用 System.arraycopy() 实现的。所以单就效率上来说，差别不大。

## clone()
实现了 Cloneable 接口的类，如 ArrayList、HashMap、Hashtable、HashSet、LinkedList、Calendar、Date 等，支持 clone() 方法。其底层也是使用 System.arraycopy() 或 Arrays.copyOf() 实现。

