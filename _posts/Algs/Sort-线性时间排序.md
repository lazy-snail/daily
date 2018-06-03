---
title: Sort-线性时间排序算法
date: 2018-03-28 00:34:10
tags: [Sort, Algs]
---
比较排序的时间复杂度下限 O(n*logn) 是确定的。在这篇博客里有[各种比较排序的对比](http://www.cnblogs.com/gaochundong/p/comparison_sorting_algorithms.html)	 。
还有一类非比较排序算法，适用于一些特定情况。这种特定情况一般是对集合的范围界定：当集合满足一定条件，可以不使用比较的方式实现排序，从而获得优于比较排序下限的时间复杂度：线性时间复杂度内完成排序。
常见的线性时间复杂度排序算法有：
1. 计数排序（Counting Sort）
2. 基数排序（Radix Sort）
3. 桶排序（Bucket Sort）

#### 计数排序
限制条件：取值范围在 [m, n] 之间的整数，wiki解释集合分布在 [0, 100] 时最适合使用计数排序。
原理：对每一个输入元素x，确定出小于x的元素个数，有了这一信息，就可以把x直接放在它在最终输出数组的位置上，例如，如果有17个元素小于x，则x就是属于第18个输出位置。当几个元素相同是，方案要略作修改。
时间复杂度：O(n)。
空间复杂度：O(n)。
这是一种稳定排序。
伪代码：
```java
COUNTING-SORT(A;B; k)
let C[0..k] be a new array
for i = 0 to k
	C[i] = 0;
for j = 1 to A.length
	C[A[j]] = C[A[j]] + 1
// C[i] now contains the number of elements equal to i .
for i = 1 to k
	C[i] = C[i] + C[i-1]
// C[i] now contains the number of elements less than or equal to i .
 for j = A.length downto 1
 	B[C[A[j]]] = A[j]
 	C[A[j]] = C[A[j]] - 1
```

#### 基数排序
原理：以十进制数组n为例，k=10，最大数字的位数是d。把元素从个位排好序，然后再从十位排好序，，，，一直到元素中最大数的最高位排好序，那么整个元素就排好序了。
时间复杂度：O(d(n+r))。
空间复杂度：O(n+r)。
这是一种稳定排序。
伪代码：
```java
RADIX-SORT(A, d)
for  i = 1 to d
	use a stable sort  to sort array A on digit i.
```

#### 桶排序
假定数据服从均匀分布，均匀独立分布在[0， 1)区间。假设有m个桶，即将区间划分为m个大小相同的子区间。将n个元素分别存放到相应的桶中，再对各个桶进行排序，如插入排序。最后遍历每个桶，按照次序列出所有元素即可。
时间复杂度：平均为O(n)。
空间复杂度：O(n)。
伪代码：
```java
BUCKET-SORT(A)
n = A.length
let B[0.. n-1] be a new array
for i = 0 to n-1
	make B[i] an empty list
for i = 1 to n
	insert A[i] into B[⌊A[i]⌋]
for i = 0 to n-1
	sort list B[i] with insertion sort
concatenate the lists B[0], B[i],...B[n-1] together in order.
```
