---
title: cpp-STL组件-迭代器
date: 2018-05-18 10:16:55
categories: C++
tags: C++
---
[toc]
## 关于迭代器
是一种抽象的设计概念，《设计模式》对迭代器模式的定义：**提供一种方法，使之能够顺序访问一个聚合对象中各个元素，而又不需暴露该对象内部的表示**。
容器和算法的泛型化设计可以通过 class remplates 和 function templates 实现，而如何设计出良好的胶着剂，是 STL 的难题。

## 详解
迭代器是一种行为类似指针（smart pointer）的对象。根据移动特性与实现的操作，STL 提供了5种类型的迭代器：
1. Input Iterator（输入迭代器），不允许改变，只读（read only）；
2. Output Iterator（输出迭代器），只写（write， only）；
3. Forward Iterator（前向迭代器），允许写入型算法如，replace()，在迭代器所制定的区间进行读写操作；
4. Bidirectional Iterator（双向迭代器），某些算法需要逆向访问某个迭代器区间（如逆向拷贝某范围的元素）；
5. Random Access Iterator（随即访问迭代器），前四种都只提供一部分指针运算能力：前三种支持 operator++，第四种加上了 operator--，第5种则涵盖所有指针运算：p±n、p[n]、p1-p2、p1 < p2。

{% asset_img 5种迭代器.png 5种迭代器 %}
每一个左边的迭代器都实现了右边迭代器的方法，是进一步的强化。

## traits 技术
STL 使用 iterator_traits 这个结构体专门“萃取”迭代器的特性如，迭代器所指向的对象的类型。有以下5种：
```c++
tempalte<typename I>  
struct iterator_traits  
{  
    typedef typename I::value_type value_type;  
    typedef typeanme I:difference_type difference_type;  
    typedef typename I::pointer pointer;  
    typedef typename I::reference reference;  
    typedef typename I::iterator_category iterator_category;  
};  
```

1. value_type：指迭代器所指对象的类型，如，原生指针也是一种迭代器，对于原生指针 int*，int 即为指针所指对象的类型，即所谓的 value_type；
2. difference_type：用来表示两个迭代器之间的距离，如：
```c++
int array[5] = {1, 2, 3, 4, 5};  
int *ptr1 = array + 1;  //指向2  
int *ptr2 = array + 3;  //指向4  
ptrdiff_t distance = ptr2 - ptr1;  //结果即为difference_type
```
3. reference：指迭代器所指对象的类型的引用。一般用在迭代器的 * 运算符重载上，如果 value_type 是 T，那么对应的 reference_type 就是 T&，如果value_type 是 const T，那么对应的 reference_type 就是 const T&。从“迭代器所指对象的内容是否允许改变的角度”，迭代器分为两种：
    > 不允许改变“所指对象的内容”，即 const iterators；如 const int *pic;
    > 允许改变“所指对象的内容”，即 mutable iterators，如 int *pi;
4. pointer：指迭代器所指的对象，也就是相应的指针。对指针而言，最常用最重要的功能就是 operator* 和 operator-> 这两个运算符。因此迭代器需要对这两个运算符进行重载。pointer 和 reference 在 C++ 中关联密切：如果“返回一个左值，令它代表 p 所指对象”是可行的，那么，“返回一个左值，令它代表 p 所指对象的地址”也一定可行。即：我们能够返回一个 pointer，指向迭代器所指对象。

5. iterator_category：分类迭代器，以上提到的5类迭代器（Input/Output Iterator等），由于有递进增强的关系，在使用时如，某个算法可以接受 Forward Iterator，此时使用 Random Access Iterator 自然同样可用，但可用不代表最佳。任何一个迭代器，其类型永远应该落在“该迭代器所隶属的各种类型中，最强化（最佳效率）的那个”。