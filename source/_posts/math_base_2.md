---
title: 算法数学基础 - 2
categories: "算法"
date: "2020-05-13 17:49:10"
tags: ["算法与数据结构", "algorithms", "data structure"]
---

## 算法数学基础 - 2

-----

1. 函数渐近界的定理

<!-- more -->

   + **定理1**
     
     > 设$f$和 $g$是定义域为自然数集合的函数：
     > 
     > （1）如果$\lim\limits_{n\rightarrow\infty}f(n)/g(n)$存在，并且等于某个常数$c>0$，那么$f(n)=\Theta(g(n))$。
     > 
     > （2）如果$\lim\limits_{n\rightarrow\infty}f(n)/g(n)=0$，那么$f(n)=o(g(n))$。
     > 
     > （3）如果$\lim\limits_{n\rightarrow\infty}f(n)/g(n)=+\infty$，那么$f(n)=\omega(g(n))$。
     
     推理1 → **多项式函数的阶**低于**指数函数的阶**，$n^d=o(r^n),r>1,d>0$。
     
     推理2 →**对数函数的阶**低于**幂函数的阶**，$\ln n=o(n^d),d>0$。
   
   + **定理2**
     
     > 设函数$f$，$g$，$h$的定义域为自然数集合，
     > 
     > （1）如果$f=O(g)$且$g=O(h)$，那么$f=O(h)$；
     > 
     > （2）如果$f=\Omega(g)$且$g=\Omega(h)$ ，那么$f=\Omega(h)$；
     > 
     > （3）如果$f=\Theta(g)$且$g=\Theta(h)$，那么$f=\Theta(h)$。
     > 
     > 函数的阶之间的关系具有传递性。
   
   + **定理3**
     
     > 假设函数$f$和$g$的定义域为自然数集，若对某个其他函数$h$，有$f=O(h)$和$g=O(h)$，那么$f+g=O(h)$。
     
     该定理可以推广到有限个函数。即算法由有限步骤构成，若每一步的时间复杂度函数的上界都是$h(n)$，那么该算法的时间复杂度函数可以写作$O(h(n))$。在常数步的情况下取最高阶函数即可。

2. 几种重要函数的性质
   
   + 基本函数类
     
     - 至少指数级：$2^n,3^n,n!~,\dots$
     
     - 多项式级：$n,n^2,n\log n,n^{\frac{1}{2}},\dots$
     
     - 对数多项式级：$\log n, \log^2n,\log\log n,\dots$
   
   + 对数函数
     
     + 符号：
       
       $\log n = \log_2n$
       
       $\log^kn=(\log n)^k$
       
       $\log \log n=\log(\log n)$
     
     + 性质：
       
       （1）$\log_2n=\Theta(\log_ln)$
       
       （2）$\log_bn=o(n^\alpha), (\alpha>0)$
       
       （3）$a^{\log_bn}=n^{\log_ba}$
   
   + 指数函数与阶乘
     
     Stirling公式  $n!=\sqrt{2\pi n}(\frac{n}{e})^n(1+\Theta(\frac{1}{n}))$
     
     $n!=o(n^n)$
     
     $n!=\omega(2^n)$
     
     $\log(n!)=\Theta(n\log n)$
   
   + 取整函数
     
     + 定义：
       
       $\lfloor x\rfloor$：表示小于等于$x$的最大整数
       
       $\lceil x\rceil$：表示大于等于$x$的最小整数
     
     + 举例：
       
       $\lfloor 2.6\rfloor=2, \lceil 2.6\rceil=3,\lfloor 2\rfloor=\lceil 2\rceil=2$
     
     + 应用：二分搜索
       
       输入数组长度$n$，中位数位置：$\lfloor n/2\rfloor$，与中位数比较后子问题大小：$\lfloor n/2\rfloor$
     
     + 性质
       
       （1）$x-1<\lfloor x\rfloor \le x \le \lceil x\rceil < x+1$
       
       （2）$\lfloor x+n \rfloor=\lfloor x\rfloor+n,\lceil x+n\rceil=\lceil x\rceil+n, n为整数$
       
       （3）$\lceil \frac{n}{2}\rceil+\lfloor \frac{n}{2}\rfloor=n$
       
       （4）$\lceil \frac{\lceil\frac{n}{a} \rceil}{b}\rceil=\lceil\frac{n}{ab} \rceil, \lfloor \frac{\lfloor \frac{n}{a}\rfloor}{b}\rfloor =\lfloor \frac{n}{ab}\rfloor$
   
   + 按照阶排序
     
     $$
     2^{2^n},\quad n!,\quad n2^n,\quad (3/2)^n,\quad (\log n)^{\log n} =n^{\log\log n}, \\
     n^3,\quad \log(n!)=\Theta(n\log n),\quad n=2^{\log n}, \\
     \log^2n,\quad \log n,\quad \sqrt{\log n}, \quad \log\log n, \\
     n^{1/\log n}=1
     $$


### 来源

-----

1. 北京大学-算法设计与分析
