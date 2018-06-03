---
title: Algs-KMP算法
date: 2018-05-05 08:42:23
categories:
tags:
---
[toc]
## The Knuth-Morris-Pratt Algorithm
经典的字符串匹配算法。但实现起来并不复杂。
首先一个概念是：
## 部分匹配表 The Partial Match Table
参考这篇博文：[The Knuth-Morris-Pratt Algorithm](http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/)
当弄清楚了什么是部分匹配表之后，接下来就是怎么使用它，在匹配失败的时候进行适当的跳跃。
使用参考这部分内容[从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827)

最后给出一个[实现](https://blog.csdn.net/biaobiaoqi/article/details/8975536)

## 其他两个字符串匹配算法
BM算法和Sunday算法，参考第二篇博客。
