---
title: STL
date: 2018-05-18 09:30:39
categories: cpp
tags: cpp
---
[toc]
## 关于 STL
STL（标准模板库）的中心思想是：将数据容器（containers）和算法（algorithms）分开，彼此独立设计，最后再以胶着剂（迭代器）将它们撮合在一起。

## 提供的六大组件：
1. 容器（containers），各种数据结构如，vector、list、deque、set、map...;
2. 算法（algorithms），各种常用算法如，sort、search、copy、erase...；
3. 迭代器（iterators），容器与算法之间的胶着剂，泛型指针，共有5种类型。所有 STL 容器
4. 仿函数（functors）/函数对象（function objects），行为类似函数，可作为算法的某种策略（policy）。从实现的角度看，是一种重载了 operator() 的 class 或 class template，一般函数指针可以视为狭义的仿函数；
5. 适配器（adapters），修饰仿函数、容器、迭代器等接口（相应地称为：仿函数接口适配器、容器接口适配器、迭代器接口适配器等）。如 queue、stack 等，它们看似容器，但只能算是一种容器适配器，因为其底层完全借助 deque，所有操作都由底层的 deque 提供；
6. 配置器（allocators），负责空间配置与管理，从实现的角度看，是一个实现了动态空间配置、空间管理、空间释放的 class template。

