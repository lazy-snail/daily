---
title: Arrays
date: 2018-06-01 15:18:57
categories: java
tags: [java, 容器]
---
[toc]
## Arrays
数组的操作类，定义在 java.util 中。不提供实例化接口（构造函数为 private 修饰的），所有的方法都是静态方法，主要实现对数组的排序、查找、填充等操作。
主要方法：
* boolean equals(int[] a, int[] a2)：判断两个数组是否相等，此方法被重载多次，可以判断各种数组类型的数组；
* void fill(int[] a, int val)：将指定的内容填充到数组之中，此方法被重载多次，可以填充各种数据类型的数组；
* void sort(int[] a)：数组排序，此方法被重载多次，可以对各种类型的数组进行排序。要求对象所在的类必须实现 Comparable 接口；
* int binarySearch(int[] a, int key)：对排序后的数组进行二分法检索，此方法被重载多次，可以对各种数据类型的数组进行搜索；
* String toString(int[] a)：输出数组信息，此方法被重载多次，可以输出各种数据类型的数组。

### sort() 方法
首先，Arrays 针对不同数据类型和数据量，重载了多个版本的 sort() 方法，以获得更优的效率。如针对 byte 类型的数组采用 计数排序；根据数据规模从快排/归并排序转换为插入排序等。
从 JDK1.7 开始，Arrays 的 sort() 方法采用 DualPivotQuicksort（双轴快排/双主元）排序实现。从命名可见，它是一种双 pivot 的快排策略，比传统单 pivot 快排效率更高。具体流程：
{% asset_img DualPivotQuicksort流程.png 流程示意图 %}
1. 需要排序的数组为 a，判断数组的长度是否大于 286（实现类中定义的 QUICKSORT_THRESHOLD），大于使用归并排序（merge sort），否则执行2；
2. 判断数组长度是否小于 47（INSERTION_SORT_THRESHOLD），小于则采用插入排序，否则执行 3；
3. 采用近似算法计算数组长度的 1/7：
int seventh = (length >> 3) + (length >> 6) + 1;
4. 取出5个点：
int e3 = (left + right) >>> 1;  // 中位数
int e2 = e3 - seventh;
int e1 = e2 - seventh;
int e4 = e3 + seventh;
int e5 = e4 + seventh;
5. 将这5个元素进行插入排序；
6. 选取 a[e2]、a[e4] 分别作为 pivot1、pivot2。由于步骤 5 进行了排序，所以必有 pivot1 < pivot2；
7. 定义 3 个指针：分别是 less、k、great。ess 和 great 将数组分为 3 个部分，分别是小于 less 的、大于 less 小于 great 的元素和大于 great 的元素。 
8. 将 a[k] 分别与 pivot1、pivot2 比较。如果小于 pivot1，则将 a[k] 与a [less] 对调，同时 k++。如果大于 pivot2，则执行 9；否则执行 10；
9. 将 a[great] 分别与 pivot1、pivot2 比较。如果 a[great] 大于 pivot2，则 great--，直到大于 pivot2 的条件不满足或者 k==great。如果 a[great] 小于 pivot1，则将 a[great] 换到小于 less 的区域。如果 a[great] 大于 pivot1，则说明位于中间区域，将 a[great] 与 a[k]对 调。great--；
10. k++，如果 k > great，说明处理完成，执行 11，否则继续执行 8；
11. 由于前面的操作，还未将 pivot1、pivot2 这 2 个元素放对位置，所以还需要将 a[less - 1] 移动到队头，pivot1 移动到（less - 1） 的位置，将 a[great +1] 移动到队尾，pivot2 移动到 （great +1） 的位置；
12. 至此，已经达到步骤7描述的最终结果，将数组分为了 3 个区域。对较小的区域和较大的区域递归执行步骤 2。判断中间的区域是否过大，如果是，则执行 13，否则递归执行步骤 2；
13. 将等于 pivot1 或者 pivot2 的元素移动到两边，然后递归执行步骤 2。

### parallelSort() 方法
JDK1.8 新增的排序方法，这是一种并行排序，可以多线程进行排序。使用了 JDK1.7 的 Fork/Join 框架使排序任务可以在线程池中的多个线程中进行。Fork/Join 实现了一种任务窃取算法，一个闲置的线程可以窃取其他线程的闲置任务进行处理。

### binarySearch() 方法
对排序后的数组进行二分查找。

### fill()
填充数组不是加在数组后面，而是将数组中的所有元素都重新赋值。

### asList()
可以将数组转为 List 但是，这个数组类型必须是 **引用类型** 的。所以，该方法对基本数据类型的支持并不是很好：比如当要转换的数组为基本数据类型时：
```java
int[] a_int = new int[10];
Integer[] a_integer = new Integer[10];

List a_int_list = Arrays.asList(a_int);
List a_integer_list = Arrays.asList(a_integer);

System.out.println(a_int_list.size());
System.out.println(a_integer_list.size());

--------------------------------
output: 
1
10
```

而且，其返回的 List 实例为 Arrays 类定义的内部类类型 ArrayList<E []>——而非 java.util.ArrayList 类类型，并且 E 必须为对象类型，这也就解释了上述情况的原因，并且不支持 add、remove 等操作。