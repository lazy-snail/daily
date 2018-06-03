---
title: 常见问题-py
date: 2018-04-22 23:00:42
categories: py
tags: [py, Q&A]
---
**python 中 range 和 xrange 的区别**
range
函数说明：range(start, stop[, step])，根据 start、stop 指定的范围和步长 step 生成一个序列 list。
xrage
使用方法和 range 一样，但生成的不再是 list，而是一个生成器。
_在 python3.x 中，range 就是使用 xrange 的实现方式，所以不再有 xrange 函数。_
