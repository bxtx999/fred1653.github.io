---
title: 分治策略-幂乘问题.md
updated: 2021-08-25 23:46:41
categories: "算法"
tags: ["算法与数据结构", "algorithms", "data structure", "分治策略", "幂乘问题", "斐波那契"]
desc: 幂乘问题
toc: true
---

## 幂乘问题

输入：$a$为给定实数，$n$为自然数

输出：$a^n$

### 传统算法思想

顺序相乘 $a^n=(...(((a\quad a)a)a)...)a$

乘法次数：$\varTheta (n)$

<!-- more -->

### 分治算法——划分

- $n$为偶数：$\underbrace{a......a}_{\text{n/2个}}|\underbrace{a......a}_{\text{n/2个}}$

- $n$为奇数：$\underbrace{a......a}_{\text{(n-1)/2个}}|\underbrace{a......a}_{\text{(n-1)/2个}}|a$

解：

$$
a^n = \begin{cases}
    a^{n/2} \times a^{n/2} &{n为偶数} \\
     a^{(n-1)/2} \times a^{(n-1)/2} &{n为奇数}
\end{cases}
$$

## 分治算法分析

以乘法作为基本运算：

- 子问题规模：不超过$n/2$

- 两个规模近似$n/2$的子问题完全一样，只要计算1次

$$
W(n)=W(n/2) + \varTheta (1) \\
W(n)=\varTheta (\log {n})
$$

## 幂乘算法的应用

Fibonacci数列：$1,1,2,3,5,8,13,21,...$

增加$F_0=0$，得到数列$0,1,1,2,3,5,8,13,21,...$

问题：已知$F_0=0,F_1=1$，给定$n$，计算$F_n$

通常算法：从$F_0,F_1,...$开始，根据递推公式

$$
F_n=F_{n-1}+F_{n-2}
$$

持续相加可以得到$F_n$，时间复杂度为$\varTheta (n)$。

### Fibonacci数的性质

**定理1** 设${F_n}$为Fibonacci数构成的数列，那么

$$
\begin{bmatrix}
   F_{n+1} & F_{n} \\
   F_n & F_{n-1}
\end{bmatrix} =
{\begin{bmatrix}
   1 & 1 \\
   1 & 0
\end{bmatrix}}^{n}
$$

归纳证明，

$n=1$时, $左边 = \begin{bmatrix}
   F_{2} & F_{1} \\
   F_1 & F_{0}
\end{bmatrix} =
{\begin{bmatrix}
   1 & 1 \\
   1 & 0
\end{bmatrix}}^{n} = 右边$,

假设对任意正整数$n$，命题成立，即

$$
\begin{bmatrix}
   F_{n+1} & F_{n} \\
   F_n & F_{n-1}
\end{bmatrix} =
{\begin{bmatrix}
   1 & 1 \\
   1 & 0
\end{bmatrix}}^{n}
$$

那么，

$$
\begin{bmatrix}
   F_{n+2} & F_{n+1} \\
   F_{n+1} & F_{n}
\end{bmatrix} =
\begin{bmatrix}
   F_{n+1} & F_{n} \\
   F_n & F_{n-1}
\end{bmatrix}
\begin{bmatrix}
   1 & 1 \\
   1 & 0
\end{bmatrix}
= {\begin{bmatrix}
   1 & 1 \\
   1 & 0
\end{bmatrix}}^{n}
\begin{bmatrix}
   1 & 1 \\
   1 & 0
\end{bmatrix}
= {\begin{bmatrix}
   1 & 1 \\
   1 & 0
\end{bmatrix}}^{n+1}
$$

### 算法

令矩阵$M=\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}$，用幂乘算法计算$M^n$时间复杂度：

- 矩阵乘法次数 $T(n)=\varTheta (\log {n})$

- 每次矩阵乘法需要做8次元素相乘

- 总计元素相乘次数为$\varTheta (\log {n})$

使用Python实现的算法:

```python
import math
import decimal
from timeit import default_timer as timer
from typing import Callable, OrderedDict, Tuple, List
from functools import lru_cache

class Fibonacci:

    num: int
    times: OrderedDict
    memory: List[int]
    MAXSIZE: int
    
    def _get_time(self, fn: Callable) -> Tuple[float, int]:
        start = timer()
        result = fn()
        end = timer()
        return end - start, result


    def __init__(self, num: int) -> None:
        self.num = num
        self.times = {}
        self.MAXSIZE = 10000

    def print_results(self):
        print("-------------------------------------")
        print("n = {}".format(self.num))
        for k, v in self.times.items():
            print("{0}： \t时间为\t{1}微秒，\t结果为{2}。".format(k, round(decimal.Decimal(v[0] * math.pow(10, 6)), 1), v[1]))

    def _fib_1(self, n: int) -> int:
        if n == 0:
            return 0
        if n == 1:
            return 1
        return self._fib_1(n-1) + self._fib_1(n-2)

    @lru_cache(maxsize=20)
    def _fib_2(self, n: int) -> int:
        if n == 0:
            return 0
        if n == 1:
            return 1
        return self._fib_2(n-1) + self._fib_2(n-2)

    def _fib_3(self, n: int) -> int:
        pre = 0
        next = 1
        result = 0
        if n == 0:
            return pre
        if n == 1:
            return next
        while (n := n - 1) >= 1:
            # 注意n的值
            result = pre + next
            pre = next
            next = result
        return result

    def _fib_4(self, n: int) -> int:
        t = math.sqrt(5.0)
        # result = (int) ((math.pow(((1 + math.sqrt(5.0)) / 2), n) - math.pow(((1 - math.sqrt(5.0)) / 2), n)) / math.sqrt(5.0))
        result = (int) ((math.pow(((1 + t) / 2), n) - math.pow(((1 - t) / 2), n)) / t)  # 速度快
        return result

    def _fib_5(self, n: int) -> int:
        if n == 0 or n == 1:
            return n
        em = [[1, 1], [1, 0]]
        return self._matrixPow(em, n + 1)[1][1]
        

    def _fib_6(self, n: int) -> int:
        if n == 0 or n == 1:
            return n
        em = [[0, 1], [0, 0]]  # 系数
        rhm = [[0, 1], [1, 1]]

        return self._matrixMultiply(em, self._matrixPow(rhm, n - 1))[0][1]

    def _matrixMultiply(self, x: List[List[int]], y: List[List[int]]) -> List[List[int]]:
        # https://stackoverflow.com/a/10508239
        # zip_rhm = list(zip(*rhm))
        # return [[sum(map(lambda x, y: x * y, row_lhm, col_rhm)) for col_rhm in zip_rhm] for row_lhm in lhm]
        return [[x[0][0] * y[0][0] + x[0][1] * y[0][1], x[0][0] * y[0][1] + x[0][1] * y[1][1]], [x[1][0] * y[0][0] + x[1][1] * y[0][1], x[1][0] * y[1][0] + x[1][1] * y[1][1]]]

    def _matrixPow(self, m: List[List[int]], n: int) -> List[List[int]]:
        r = m
        res = [[1, 0], [0, 1]]
        # while n != 0:
        #     if n & 1 == 1:
        #         res = self._matrixMultiply(res, r)
        #     r = self._matrixMultiply(r, r)
        #     n >>= 1
        n <<= 1
        while (n := n >> 1) != 0:
            if n & 1 == 1:
                res = self._matrixMultiply(res, r)
            r = self._matrixMultiply(r, r)
        return res

    def _fib_7(self, n: int) -> int:
        if n == 0 or n == 1:
            return n
        if n == 2:
            return 1
        self.memory = [-1 for x in range(n+1)]
        self.memory[0] = 0
        self.memory[1] = 1
        self.memory[2] = 1
        for i in range(3, n + 1):
            self.memory[i] = self.memory[i-1] + self.memory[i-2]
        return self.memory[n]

    def _fib_8(self, n: int) -> int:
        if n == 0 or n == 1:
            return n
        if n == 2:
            return 1
        self.memory = [0, 1, 1]
        for i in range(3, n+1):
            self.memory[i%3] = self.memory[(i-2)%3] + self.memory[(i-1)%3]
        return self.memory[n%3]

    def _memory_reset(self):
        self.memory = [-1 for i in range(self.num + 1)]

    def _fib_9(self, n: int) -> int:
        if n == 0:
            return 0
        if n == 1:
            return 1
        if self.memory[n] == -1:
            self.memory[n] = self._fib_9(n-1) + self._fib_9(n-2)
        return self.memory[n]
    
    def run(self):
        # self.times["递归法"] = self._get_time(lambda: self._fib_1(self.num))
        self.times["递归cache法"] = self._get_time(lambda: self._fib_2(self.num))
        self.times["迭代法"] = self._get_time(lambda: self._fib_3(self.num))
        self.times["公式法"] = self._get_time(lambda: self._fib_4(self.num))
        self.times["矩阵法1"] = self._get_time(lambda: self._fib_5(self.num))
        self.times["矩阵法2"] = self._get_time(lambda: self._fib_6(self.num))
        self.times["动态规划"] = self._get_time(lambda: self._fib_7(self.num))
        self.times["动态规划压缩"] = self._get_time(lambda: self._fib_8(self.num))
        self._memory_reset()
        self.times["记忆法"] = self._get_time(lambda: self._fib_9(self.num))
        

if __name__ == '__main__':
    f_1 = Fibonacci(25)
    f_1.run()
    f_1.print_results()

    f_2 = Fibonacci(44)
    f_2.run()
    f_2.print_results()
```

输出结果：

```Python
-------------------------------------
n = 25
递归法：        时间为  17734.4微秒，   结果为75025。
递归cache法：   时间为  14.9微秒，      结果为75025。
迭代法：        时间为  3.3微秒，       结果为75025。
公式法：        时间为  12.2微秒，      结果为75025。
矩阵法1：       时间为  9.5微秒，       结果为75025。
矩阵法2：       时间为  6.7微秒，       结果为75025。
动态规划：      时间为  8.5微秒，       结果为75025。
动态规划压缩：  时间为  7.9微秒，       结果为75025。
记忆法：        时间为  14.7微秒，      结果为75025。
-------------------------------------
n = 44
递归法：        时间为 212522669.9微秒，结果为701408733。
递归cache法：   时间为  32.4微秒，      结果为701408733。
迭代法：        时间为  16.9微秒，      结果为701408733。
公式法：        时间为  2.0微秒，       结果为701408733。
矩阵法1：       时间为  13.6微秒，      结果为701408733。
矩阵法2：       时间为  14.7微秒，      结果为701408733。
动态规划：      时间为  12.2微秒，      结果为701408733。
动态规划压缩：  时间为  16.8微秒，      结果为701408733。
记忆法：        时间为  53.0微秒，      结果为701408733。
```

本文中介绍的矩阵是`矩阵法2`。
