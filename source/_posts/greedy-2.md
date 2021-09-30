---
title: 贪心算法基础-2
date: 2021-09-30 17:06:50
categories: "算法"
tags: ["算法与数据结构", "algorithms", "data structure", "贪心算法"]
desc: 贪心算法（greedy algorithm），又称贪婪算法，是一种在每一步选择中都采取在当前状态下最好或最优（即最有利）的选择，从而希望导致结果是最好或最优的算法。
toc: true
---

本文继续介绍重要的贪心算法，比如哈夫曼编码、最小生成树等算法。

## 最优前缀码及哈夫曼编码

### 二元前缀码

**二元前缀码**：用0-1字符串作为代码表示字符，要求任何字符的代码都不能作为其它字符代码的前缀。

**非前缀码的例子**：

`a: 001, b: 00,    c: 010, d: 01`

解码的歧义，例如字符串`0100001`

- 解码1：01,00,001 $\implies$ d,b,a

- 解码2：010,00,01 $\implies$ c,b,d

<!-- more -->

### 前缀码的二叉树表示

**前缀码**：`{00000,00001,0001,001,01,100,101,11}`

构造树：

- `0` —— 左子树

- `1` —— 右子树

- 码对应着一片树叶

- 最大位数为数深

![二叉树表示](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_17-46-25.png)

平均传输位数计算公式：$B=\sum\limits_{i=1}^{n}f(x_i)d(x_i)$

![平均传输位数](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_17-47-29.png)

计算得到：$B=[(5+5)\times 5+10\times 4+(15+10+10)\times 3+(25+20)\times 2]/100=2.85$

*问题：*给定字符集$C=\{x_1,x_2,...,x_n\}$和每个字符的频率$f(x_i),i=1,2,...,n$。求关于C的一个最有前缀码（平均传输位数最小）。

### 哈夫曼算法

#### 伪代码

算法 Huffman(C)

输入：$C=\{x_1,x_2,...,x_n)\},f(x_i)=1,2,...,n$。

输出：$Q$ // 队列

1. $n \gets |C|$

2. $Q \gets C$ // 频率递增队列$Q$

3. $for \quad i\gets 1 \quad to \quad n-1 \quad do$

4. $\quad \quad z \gets$ Allocate-Node()  // 生成节点$z$

5. $\quad \quad z.left() \gets Q$中的最小元  // 最小作$z$左儿子

6. $\quad \quad z.right \gets Q$中的最小元  // 最小作$z$右儿子

7. $\quad \quad f(z)\gets f(x) +f(y)$

8. $\quad \quad$Insert$(Q,z)$ // 将$z$插入$Q$

9. $return Q$

#### 实例

输入：`a:45;b13;c:12;d:16;e:9;f:5`

能够得到下面的二叉树：

![二叉树](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_17-47-37.png)

编码为：

- f `0000`

- e `0001`

- d `001`

- c `010`

- b `011`

- a `1`

平均传输位数：2.24

#### 最优前缀码性质

**引理1**：$C$是字符集，$\forall c \in C$，$f(c)$为频率，$x,y \in C$,$f(x),f(y)$频率最小，那么存在最优二元前缀码使得$x,y$码字等长且仅在最有一位不同。

![引理1](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_17-47-52.png)

$B(T)-B(T')=\sum\limits_{i \in C}f[i]d_T(i)-\sum\limits_{i \in C}f[i]d_{T'}(i) \ge 0$，其中$d_T(i)$为$i$在$T$中的层数（$i$到根的距离）。

**引理2**：设$T$是二元前缀码的二叉树，$\forall x,y \in T$，$x,y$是树叶兄弟，$z$是$x,y$的父亲，令$T'=T-{x,y}$,且令$z$的频率$f(z)=f(x)+f(y)$。$T'$是对应二元前缀码$C'=(C-\{x,y\})\cup \{z\}$的二叉树，那么$B(T)=B(T')+f(x)+f(y)$。

![引理2](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_17-48-02.png)

**引理2证明**：

证 $\forall c \in C - \{x,y\}$，有

$d_T(c)=d_T,(c)\implies f(c)d_T(c)=f(c)d_T,(c) \\ d_T(x)=d_T(y)=d_T,(z)+1$

$B(T)=\sum\limits_{i \in T}f(i)d_T(i)\\ =\sum\limits_{i \in T, i \neq x,y}f(i)d_T(i)+f(x)d_T(x)+f(y)d_T(y) \\ =\sum\limits_{i \in T', i\neq z}f(i)d_{T'}(i)+f(z)d_{T'}(z)+(f(x)+f(y)) \\ =B(T')+f(x)+f(y)$

## 哈夫曼编码的正确性证明

### 算法正确性证明思路

**定理**：Huffman算法对任意规模为$n\quad (n\ge 2)$的字符集$C$都得到关于$C$的最优前缀码的二叉树。

**归纳基础** 证明：对于$n=2$的字符集，Huffman算法得到最优前缀码。

## 最小生成树

## Prim算法

## Kruskal算法

## 单元最短路径问题及算法

## Dijkstra算法的证明


