---
title: Python-迭代器
date: 2018-04-23 09:50:09
tags: py
---
#### 迭代（Iteration）
如，给定一个 list 或 tuple，可以通过 for 循环遍历之，称之为 迭代。
只要是可迭代对象，无论是否可以使用下标遍历，都可以使用迭代，如 dict。通过 collections 模块的 Iterable 类型判断来验证一个对象是否可迭代：
```
>>> from collections import Iterable
>>> isinstance("abc", Iterable)  # True
```
dict 默认迭代的是 key，迭代 value：for v in d.values( )；迭代 key 和 value：for k, v in d.items( )。

#### 生成器（Generator）
通过列表生成式（List Comprehensions），可以用来创建 list，如
```
>>> [x * x for x in range(1, 6)]
>>> [1, 4, 9, 16, 25]
```
但是列表生成式的容量受内存限制：声明的容量会直接占用相应的内存。所以，如果列表元素可以按照某种算法推算出来，那么就可以在循环中不断推算出后续元素，这样就不必创建完整的 list，从而节省空间。这种边循环边计算的机制，被称为生成器（Generator）。

**创建**
1. 最简单的方式，把上面的生成式“[]”改成“()”，就成了创建一个 generator：
```
>>> (x * x for x in range(1, 6))
>>> <generator object <genexpr> at 0x7f2a39f1b280>
```
2. yield 关键字
如果一个函数定义中使用了 yield 关键字，那么它就不再是一个普通函数，而是一个迭代器。

**访问**
* 使用 next(g)，每次使用，会返回一个对象，直到对象被用尽；
* for 循环。
