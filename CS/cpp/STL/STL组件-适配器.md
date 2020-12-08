---
title: STL-适配器
date: 2018-05-19 15:31:26
categories: cpp
tags: cpp
---
[toc]
## 适配器(adapters)
适配器概念上是一种设计模式：《设计模式》对它的描述为：将一个类的接口 **转换成** 客户希望的另外一个接口，使得原本由于接口不兼容而不能合作的类能够一起工作。

## 分类
STL 提供函数对象接口适配器、容器接口适配器、迭代器接口适配器，相应地，它们用于改变/转换函数对象、容器、迭代器的接口。
* 容器适配器：queue、stack，见“序列式容器”；

* 迭代器适配器：STL 提供了很多应用于迭代器的适配器：insert iterators、reverse iterators、iostream iterators 等，可以从 <iterator\> 获得，SGI STL 则定义于 <stl_iterator.h\>。
> **Insert Iterators**：可以将一般迭代器的赋值操作转变为插入操作。包括专司尾端插入操作的 back_insert_iterator、头部插入操作的 front_insert_iterator、任意位置插入操作的 insert_iterator，由于它们的使用接口不够直观，STL 提供了相应的封装函数：back_inserter()、front_inserter()、inserter()，提升使用的便捷性。
> **Reverse Iterators**：可以将一般迭代器的行进方向逆转，使原本应该前进的 operator++ 变成后退操作，& vice versa。
> **IOStream Iterators**：可以将迭代器绑定到某个 iostream 对象上。相应地，绑定到 istream 如 std::cin 上的，称为 istream_iterator，拥有输入功能；绑定到 ostream 如 std::cout 上的，称为 ostream_iterator，拥有输出功能。以此为基础，稍加修改，便可适用于任何 I/O 设备上，如绑定到 IE cache 上，或绑定到一个磁盘目录上，etc.

* 函数对象适配器：数量最多、灵活度最高，可以适配、适配、再适配。可以从 <functional\> 获得，SGI STL 则定义于 <stl_function.h\>。通过它们之间的绑定、组合、修饰，几乎可以无限制地创造出各种可能的表达式，搭配 STL 算法一起使用。而且，这为数众多的适配器也使得“一般函数”、“成员函数”得以和其他适配器或算法无缝结合。