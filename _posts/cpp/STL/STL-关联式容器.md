---
title: STL-关联式容器
date: 2018-05-18 16:19:38
categories: cpp
tags: [cpp, 容器]
---
[toc]
**容器，置物之所也**
常用的数据结构不外乎 array（数组）、list（链表）、tree（树）、stack（栈）、queue（队列）、hash table（散列表）、set（集合）、map（映射表）等。根据数据在容器中的排列特性，这些数据结构分为序列式（sequence）和关联式（associative）两种。

所谓关联式容器，每个元素都由键值对组成。当插入元素时，容器内部结构（如红黑树/散列表）会依照其键值大小，以给定规则将该元素插入到适当位置。没有头尾的概念，也就没有诸如 push_back()、pop_back()、begin()、end() 等操作。

**标准关联式容器**
标准的关联式容器分为 set（集合）和 map（映射表）两大类及其衍生体如，multiset（多键集合）、multimap（多键映射表）。**这些容器的底层机制均以 RB-tree（红黑树）实现**，而 RB-tree 本身也是一个独立容器，但并不开放使用。
**C++11引入的关联式容器**
C++11 标准引入的 STL 容器中，除了 forward_list 是序列式容器外，其余的4种 unordered_set、sunordered_multiset、unordered_map、unordered_multimap 是 **基于 hash table 实现的关联式容器**。
**SGI STL 提供的关联式容器**
此外，SGI STL 还提供了一个不在标准内的关联式容器：hash table（散列表），以及由此为底层机制实现的shash_set（散列集合）、hash_multiset（散列多键集合）、hash_multimap（散列多键映射表）。
但随着 C++11 标准的引入，显然，在选择上有了倾向性。

## 红黑树实现的关联式容器
提供稳定的动态操作时间：**查询、插入、删除时间复杂度都是 O(logn)。**

### set
所有元素根据键值自动排序，set 元素的键值就是实值，实值就是键值，不允许相同键值的元素。set 不可以通过迭代器修改元素值，即，set iterator 是一种 constant iterator。
set 所提供的各种操作接口，几乎都是调用底层的 RB-tree 的接口实现的。
STL 特别提供了一组 set/multiset 相关算法，包括交集（set_intersection）、联集（set_union）、差集（set_difference）、对称差集（set_symmetric_difference）。

### multiset
特性和用法和 set 完全相同。唯一的差别是它允许键值重复，因此它的插入操作采用的是底层 RB-tree 的 insert_equal() 而非 insert_unique()。

### map
所有元素根据键值自动排序，所有元素都是 pair，即键值对，不允许两个元素拥有相同的键值。map 同样不可以通过迭代器修改元素的键值——这会破坏 map 组织（排列规则），但可以修改实值，因为 map 元素的实值并不影响其元素的排列规则。因此，map iterator 既不是一种 constant iterator，也不是一种 mutable iterator。
同样地，map 的各种接口也基本借助底层的 RB-tree 来实现。
**map 和 list 相同的一些性质：插入/删除都不影响除被操作元素之外的其他元素的迭代器**。

### multimap
特性和用法和 map 完全相同。唯一的差别是它允许键值重复，因此它的插入操作采用的是底层 RB-tree 的 insert_equal() 而非 insert_unique()，和 set/multiset 一样。

## 散列表实现的关联式容器
结合 C++11 引入的4个和 SGI STL 中提供的非标准关联式容器：

### unordered_set & hash_set
非有序的 set，理论上查询时间是 O(1)，但这并不意味着一定比上面的红黑树实现更快：实际使用中要考虑数据量等因素，而且 unordered_set 的 hash 函数的构造速度也不一定很快。

### unordered_multiset & hash_multiset
特性和用法同上，区别是允许重复键值，所以还是插入函数的底层调用不同（multixxx 调用的是 insert_equal()，而非上面的 insert_unique()。

### unordered_map & hash_map
非有序的 map，理论上查询时间是 O(1)。

### unordered_multimap & hash_multimap
特性和用法同上，区别是允许重复键值，所以还是插入函数的底层调用不同。