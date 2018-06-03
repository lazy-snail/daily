---
title: cpp-STL组件-配置器
date: 2018-05-19 22:52:28
categories: cpp
tags: cpp
---
[toc]
## 空间配置器
配置器（allocators），负责空间配置与管理，从实现的角度看，是一个实现了动态空间配置、空间管理、空间释放的 class template。
从 STL 应用角度看，它总是隐藏在一切组件（指容器）的背后；从 STL 实现角度看，整个 STL 操作对象都存放在容器里，而容器的空间配置都是由空间配置器完成的。
allocator 被称为空间配置器而非内存配置器，是因为空间也有可能是磁盘或其他辅助存储介质，即，可以实现直接向磁盘读取空间的 allocator。

C++ 内存配置基本操作是 ::operator new()，内存释放基本操作是 ::operator delete()。这两个全局函数相当于 C 的 malloc()和 free()函数。

## SGI 双层配置器
SGI 的空间配置与释放设计哲学：
* 向 system heap 请求空间；
* 考虑多线程（multi-threads）状态；
* 考虑内存不足时的应对措施；
* 考虑过多“小型区块”可能造成的内存碎片（fragment）问题。

考虑到小型区块可能造成的内存破碎问题，SGI 设计了双层级配置器，第一级配置器直接使用 malloc()和free()，第二级配置器则视情况采用不同的策略：当配置区块超过 128 bytes 时，视之为“足够大”，调用第一级配置器；当配置区块小于 128 bytes 时，视之为“过小”，为了降低额外负担（overhead），采用复杂的 memory pool 整理方式，不再借助第一级配置器。

### 第一级配置器 __malloc_alloc_template
以 malloc()、free()、realloc()等 C 函数执行实际的内存配置、释放、重配置等操作，并实现类似 C++ new-handler 机制（即，它不能直接使用 C++ new-handler 机制，因为它并非使用 ::operator new 来配置内存）。

**C++ new Handler**
可以要求系统在内存配置请求无法被满足时，调用一个指定的函数，即，一旦 ::operator new()无法完成任务，在抛出 std::bad_alloc 异常之前，先调用指定的处理函数，该处理函数通常被称为 new-handler

SGI 第一级配置器的空间配置函数 allocate()和 realloc()都是在调用 malloc()和realloc()不成功后，改调用 oom_malloc()和 oom_realloc()。后两者通过内循环不断调用“内存不足处理例程”，期望在某次调用之后，获得足够的内存而完成人物。但如果该例程没有被用户设置，oom_malloc()和 oom_realloc()只能调用 __THROW_BAD_ALLOC，抛出 bad_alloc 异常，或利用 exit(1) 终止程序。空间释放函数 deallocate()直接调用 free()执行。

**设计“内存不足处理例程”是用户（开发者）的责任。**

### 第二级配置器 __default_alloc_template
通过一些机制避免太多小型区域造成的内存碎片。具体做法是，当配置区块超过 128 bytes 时，视之为“足够大”，移交并调用第一级配置器；当配置区块小于 128 bytes 时，采用内存池管理，不再借助第一级配置器。

#### 第二级配置器的空间配置与释放
通过 __default_alloc_template 的标准接口函数 allocate()和 deallocate()进行空间配置和释放。显然，二者会判断空间大小（是否大于 128 bytes），从而决定是否调用第一级配置器的配置/释放函数。

#### 内存池
tbd