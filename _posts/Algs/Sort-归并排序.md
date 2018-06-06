---
title: Sort-归并排序
date: 2018-05-01 01:45:10
categories: Algs
tags: [Sort, Algs]
---
## 简述
将已有序的子序列合并，得到完全有序的序列的过程，i.e. 先使子序列有序，再使序列段间有序。
* 时间复杂度： O(NlogN) ;
* 空间复杂度：辅助空间：O(N);
* 稳定排序，常使用递归实现。

```java
void merge(Comparable[] a, int lo, int mid, int hi) {
    int i = lo, j = mid + 1;

    for (int k = lo; k <= hi; k++)
        aux[k] = a[k];

    for (int k = lo; k <= hi; k++) {
        if (i > mid)
            a[k] = aux[j++];
        else if (j > hi)
            a[k] = aux[i++];
        else if (less(aux[j], aux[i]))
            a[k] = aux[j++];
        else
            a[k] = aux[i++];
    }
}

// 自顶向下递归：
void sort(Comparable[] a, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    sort(a, lo, mid);
    sort(a, mid + 1, hi);
    merge(a, lo, mid, hi);
}

// 自底向上循环：
void sort(Comparable[] a) {
    int N = a.length;
    aux = new Comparable[N];
    for (int sz = 1; sz < N; sz = sz + sz) {
        for (int lo = 0; lo < N - sz; lo += sz + sz)
            merge(a, lo, lo + sz - 1, Math.min(lo + +sz + sz - 1, N - 1));
    }
}
```

## 优化
* 小规模子序列（7 ~ 15）改用插入排序/选择排序；
* 测试子序列是否已经有序：a[mid] <= a[mid]，则这两个子序列无需调用接下来的 merge() ，直接拷贝即可；
* 不将元素复制到辅助空间：将辅助空间也带入 sort()、merge() 方法，每次递归变换二者的位置，从而无需反复拷贝子序列到辅助空间，而是临时将辅助空间用于排序和归并。
