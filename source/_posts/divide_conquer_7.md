---
title: 分治策略-卷积
date: 2021-09-01 22:52:49
tags: ["算法与数据结构", "algorithms", "data structure", "分治策略"]
categories: "算法"
desc: 分治策略的卷积算法
toc: true
---

## 卷积及其应用

### 向量计算

给定向量：

- $a=(a_0, a_1, ..., a_{n-1})$

- $b=(b_0, b_1, ..., a_{n-1})$

向量和： $a+b=(a_0+b_0,a_1+b_1,...,a_{n-1}+b_{n-1}$

<!-- more -->

内积： $a \cdot b=a_0b_0+a_1b_1+...+a_{n-1}b_{n-1}$

卷积： $a*b=(c_0,c_1,...,c_{2n-2})$，其中$\large c_k=\sum\limits_{i+j=k \atop i,j<n}a_ib_j, \\ \quad k=0,1,2,...,2n-2$

**卷积**的意义：在下述矩阵中，每个*斜线*的项之和恰好是卷积中的各个分量。

$$
\Large {\begin{array}{cc}
 & \color{blue}{ab_0} & \color{blue}{ab_1} & \color{blue}{...} & \color{blue}{ab_{n-2}} & \color{blue}{ab_{n-1}} \\
\color{blue}{a_0b} & a_0b_0 & a_0b_1 & ... & a_0b_{n-2} & a_0b_{n-1} \\
\color{blue}{a_1b} & a_1b_0 & a_1b_1 & ... & a_1b_{n-2} & a_1b_{n-1} \\
\color{blue}{\vdots} & \vdots & \vdots & & \vdots & \vdots \\
\color{blue}{a_{n-2}b} & a_{n-2}b_0 & a_{n-2}b_1 & ... & a_{n-2}b_{n-2} & a_{n-2}b_{n-1} \\
\color{blue}{a_{n-1}b} & a_{n-1}b_0 & a_{n-1}b_1 & ... & a_{n-1}b_{n-2} & a_{n-1}b_{n-1} \\
\end{array}}
$$

计算实例：

向量：$a=(1,2,4,3),\quad b=(4,2,8,0)$,

则：

- $a+b=(5,3,12,3)$

- $a\cdot b=1\times 4+2\times 2+4\times 8+3\times 0=40$

- $a*b=(4, 10,{\color{red}{28}},36,38,24,0)$

  其中，$28$是这么计算出来的：

  $$
  \large
  \begin{array}{cc}
  & ab_0 & ab_1 & ab_2 & ab_3 \\
  a_0b & 1\times 4 & 1\times 2 & \color{red}{1\times 8} & 1 \times 0 \\
  a_1b & 2\times 4 & \color{red}{2\times 2} & 2 \times 8 & 2\times 0 \\
  a_2b & \color{red}{4\times 4} & 4 \times 2 & 4 \times 8 & 4 \times 0 \\
  a_3b & 3 \times 4 & 3 \times 2 & 3 \times 8 & 3 \times 0 \\
  \end{array}
  $$

**卷积与多项式乘法**的关系

多项式乘法：$C(x)=A(x)B(x)$

$A(x)=a_0+a_1x+a_2x^2+\dots+a_{m-1}x^{m-1}$

$B(x)=b_0+b_1x+b_2x^2+\dots+b_{n-1}x^{n-1}$

$C(x)=a_0b_0+(a_0b_1+a_1b_0)x+\dots+a_{m-1}b_{n-1}x^{m+n-2}$

其中$x^k$的系数

$c_k=\sum\limits_{i+j=k \atop i \in \{0,1,,...,m-1\}, j\in \{0,1,...,n-1\}} a_ib_j,\quad k=0,1,\dots,m+n-2$

### 平滑处理

信号向量：$a=(a_0,a_1,\dots,a_{m-1})$

$b=(b_{2k},b_{2k-1},...,b_0)=(w_{-k},...,w_k)$

$a_i'=\sum_{s=-k}^{k}a_{i+s}b{k-s}=\sum_{s=-k}^{k}a_{i+s}w_s$

把向量$b$看作$2k+1$长度窗口在$a$上移动计算$a*b$，得到$(a_0',a_1',...,a_{m-1}')$。有少数向有误差。

**实例**：

信号向量：$a=(a_0, a_1,...,a_8)$

步长：$k=2$

权值：$w=(w_{-2},w_{-1},w_0,w_1,w_2) \\ =(b_4,b_3,b_2,b_2,b_1,b_0)$

${a_i}'=a_{i-2}b_4+a_{i-1}b_3+a_ib_2+a_{i+1}b_1+a_{i+2}b_0$

下标之和为$i+k$

## 卷积计算

### 蛮力算法

向量$a=(a_0,a_1,....,a_{n-1})$和$b=(b_0,b_1,...,b_{n-1})$

$A(x)=a_0+a_1x+a_2x^2+...+a_{n-1}x^{n-1}$

$B(x)=b_0+b_1x+b_2x^2+...+b_{n-1}x^{n-1}$

$C(x)=A(x)B(x) \\ = a_0b_0+(a_0b_1+a_1b_0)x+...+a_{n-1}b_{n-1}x^{2n-2}$

$C(x)$的系数向量就是$a*b$，即卷积$a*b$计算等价于多项式相乘。

蛮力算法时间为$O(n^2)$

计算2n-1次多项式$C(x)$:

1. 选择值$x_0,x_1,...,x_{2n-1}$，求出$A(x_j)$和$B(x_j)$，$j=0,1,...,2n-1$

2. 对每个$j$，计算$C(x_j)=A(x_j)B(x_j)$

3. 利用多项式插值的方法，由$C(x)$在$x=x_0,x_1,...,x_{2n-1}$的值求出多项式$C(x)$的系数

> 主要步骤是多项式求值。如何选择$x_0,x_1,x_2,...$的值就是高效求取多项式关键问题。

高斯滤波的权值函数为：

$w_s = \frac{1}{z}e^{-s^2},\quad s=0,\pm 1,...,\pm k$

$w=(w_{-k},...,w_{-1},w_0,w_1,...,w_k)$

其中z用于归一化处理，使所有的权值之和为1,处理结果：

${a_i}'=\sum\limits_{s=-k}^{k}a_{i+s}w_s$

2n个数的选择：**1的2n次根**

$\Large {\omega}_j=e^{\frac{2\pi j}{2n}i}=e^{\frac{\pi j}{n}i} \\ \cos {\frac{\pi j}{n}} + i\sin \frac{\pi j}{n}$

$j=0,1,...,2n-1,\quad i=\sqrt{-1}$

当$n=4$时，

- $\Large{\omega}_0=1,\quad {\omega}_1=e^{\frac{\pi}{4}i}={\sqrt{2}}/2+{\sqrt{2}}/2\cdot i$

- $\Large{\omega}_2=e^{\frac{\pi}{2}i}=i,\quad {\omega}_3=e^{\frac{3\pi}{4}i}=-{\sqrt{2}}/2+{\sqrt{2}}/2\cdot i$

- $\Large{\omega}_4=e^{\pi i}=-1,\quad {\omega}_5=e^{\frac{5\pi}{4}i}=-{\sqrt{2}}/2-{\sqrt{2}}/2\cdot i$

- $\Large{\omega}_6=e^{\frac{3\pi}{2}i}=-i,\quad {\omega}_7=e^{\frac{7\pi}{4}i}={\sqrt{2}}/2-{\sqrt{2}}/2\cdot i$

## 快速傅里叶变换FFT算法

### 设计思想

1. 对$x=1,{\omega}_1,{\omega}_2,...,{\omega}_{2n-1}$，分别计算$A(x),B(x)$；

1. 利用第1步中的结果堆每个$x=1,{\omega}_1,{\omega}_2,...,{\omega}_{2n-1}$，计算得到$C(x)$，得到$C(1)=d_0,C({\omega}_1)=d_1,...,C({\omega}_{2n-1}=d_{2n-1}$；

1. 构造多项式：$D(x)=d_0+d_ix+d_2x^2+...+d_{2n-1}x^{2n-1}$。

1. 对$x=1,{\omega}_1,{\omega}_2,...,{\omega}_{2n-1}$，计算$D(x),D(1),D({\omega}_1),...,D({\omega}_{2n-1})$

可以证明：

$$
\begin{gathered}
D(1)=2nc_0 \\
D({\omega}_1)=2n c_{2n-1} \\
... \\
D({\omega}_{2n-1})=2nc_1
\end{gathered}
\quad
\Harr
\quad
\begin{gathered}
c_0=D(1)/2n \\
c_{2n-1} = D({\omega}_1) / 2n  \\
... \\
c_1 = D({\omega}_{2n-1}) / 2n
\end{gathered}
$$

**算法的关键**：

令$x=1,{\omega}_1,{\omega}_2,...,{\omega}_{2n-1}$，

- 步1对$2n$个$x$值分别求**多项式**$A(x),B(x)$

- 步2做$2n$次乘法

- 步3对$2n$个$x$值求值**多项式**$D(x)$

**关键**：一个对所有的$x$快速多项式求值算法。

### 多项式求值算法

给定多项式：

$\quad A(x)=a_0+a_1x+...+a_{n-1}x^{n-1}$

设$x$为$1$的$2n$次根，对所有的$x$计算$A(x)$的值。

**算法1**：对每个$x$做下述运算：

依次计算每个项$a_ix^i,i=1,...,n-1$对$a_ix^i(i=0,1,...,n-1)$求和。

蛮力算法的时间复杂度：$T_1(n)=O(n^3)$

**改进的算法2**：对每个$x$做下述运算：

$A_1(x)=a_{n-1}$

$A_2(x)=a_{n-2}+xA_1(x)=a_{n-2}+a_{n-1}x$

$A_3(x)=a_{n-3}+xA_2(x)=a_{n-3}+a_{n-2}x+a_{n-1}x^2$

$\quad ... \quad$

$A_n(x)=a_0+xA_{n-1}(x)=A(x)$

带入已经计算的值，得到新的算法。

算法的时间复杂度：$T_2(n)=O(n^2)$

## 平面点集的凸包

问题（平面点集的凸包）

给定大量离散点的集合$Q$，求一个最小的凸多边形，使得$Q$中的点在该多边形内或者边上。

**应用背景**：

图形处理中用于形状识别：字形识别、碰撞检测等。

### 分治算法

1. 以连接点最大纵坐标点$y_{max}$和最小纵坐标点$y_{min}$的线段$d=\{y_{max},y_{min}\}$划分$L$为左点集$L_{left}和右点集$L_{right}$。

2. $Deal(L_{left}=Deal(L_{right})$

#### Deal(L_left)

考虑$L_{left}$：确定距d最远的点P

在三角形内的点，删除：

- a外的点与a构成$L_{left}$的子问题；

- b外的点与b构成$L_{left}$的子问题。
