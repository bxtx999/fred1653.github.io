---
title: 分治策略-快速排序.md
date: 2021-08-25 23:36:41
categories: "算法与数据结构"
tags: ["算法与数据结构", "algorithms", "data structure", "分治策略", "快速排序"]
desc: 快速排序
toc: true
---

## 基础思想

- 用首元素$x$作划分标准，将输入数组$A$划分成不超过$x$的元素构成的数组$A_L$，大于$x$的元素构成的数组$A_R$。其中，$A_L,A_R$从左到右存放数组$A$的位置。

- 递归地堆子问题$A_L$和$A_R$进行排序，直到子问题规模为$1$时停止。

<!-- more -->

## 伪代码

算法：`Quicksort(A,p,r)`

输入：数组`A[p..r]`

输出：排好序的数组`A`

```c
if p < r
  then q <- Partition(A, p, r)
        A[p]<->A[q]
        Quicksort(A, p, q-1)
        Quicksort(A, q+1, r)
```

初始置`p=1, r=n`，然后调用上述算法。

```c
x <- A[p]
i <- p
j <- r + 1
while true do
    repeat j <- j - 1
    until A[j] <= x // 不超过首元素的
    repeat i <- i + 1
    until A[i] > x // 比首元素大的
    if i < j
        then A[i] <-> A[j]
    else return j
```

划分时间复杂度

- 最坏情况：

  $$
    W(n)=W(n-1)+n-1 \\ W(1)=0 \\ W(n)=n(n-1)/2
  $$

- 最好情况：

  $$
    T(n) = 2T(n/2)+n-1 \\
    T(1) = 0 \\
    T(n) = \varTheta(n \log {n})
  $$

  利用生成递归树计算复杂度也是$O(n\log {n})$。

- 平均时间复杂度

  首元素排好序后处在$1,2,...,n$，各种情况概率为$1/n$。

  - 首元素出现在位置$1$：  $T(0), T(n-1)$

  - 首元素出现在位置$2$：  $T(1), T(n-2)$

  - ...

  - 首元素出现在位置$n-1$: $T(n), T(1)$

  - 首元素出现在位置$n$：  $T(n-1), T(0)$

  子问题工作来量为$2[T(1)+T(2)+...+T(n-1)]$，

  划分工作两为$n-1$，

  即

  $$
    T(n) = \frac{1}{n}\sum_{k=1}^{n-1}(T(k)+T(n-k))+n-1 \\
    T(n) = \frac{2}{n}\sum_{k=1}^{n-1}T(k)+n-1 \\
    T(1) = 0 \\
    T(n) = \varTheta(n \log {n})
  $$

  首元素划分后每个位置概率相等。

## 算法实现

## 小结

- 分支策略

- 子问题划分由首元素决定

- 最坏情况时间：$O(n^2)$

- 平均情况时间：$O(n\log{n})$
