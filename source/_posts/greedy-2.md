---
title: 贪心算法基础-2
date: 2021-09-30 17:06:50
categories: "算法与数据结构"
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

#### Python实现哈夫曼树

```python
from typing import Optional, Tuple, Any, List

class Node:
    _name: str
    _value: int
    _left: Any
    _right: Any

    def __init__(self, name: Optional[str], value: Optional[str]):
        self._name = name
        self._value = value
        self._left = Optional[Node]
        self._right = Optional[Node]


class HuffmanTree(object):
    Node_Queue: List[Node]
    huffman_code: List[int]
    root: Node

    def __init__(self, char_weights: Tuple[str, int]):
        self.Node_Queue = [Node(weight[0], weight[1]) for weight in char_weights]
        while len(self.Node_Queue) != 1:
            self.Node_Queue.sort(key=lambda node: node._value, reverse=True)
            tmp = Node(name=None, value=(self.Node_Queue[-1]._value+self.Node_Queue[-2]._value))
            tmp._left = self.Node_Queue.pop(-1)
            tmp._right = self.Node_Queue.pop(-1)
            self.Node_Queue.append(tmp)

        self.root = self.Node_Queue[0]
        self.huffman_code = [0 for i in range(10)]

    def pre(self, tree: Node, length: int):
        node = tree
        if not node:
            return
        elif node._name:
            print(node._name + '的编码为' + ''.join([str(i) for i in self.huffman_code[0:length]]))
            return
        
        self.huffman_code[length] = 0
        self.pre(node._left, length + 1)
        self.huffman_code[length] = 1
        self.pre(node._right, length + 1)
    
    def get_code(self):
        self.pre(self.root, 0)
    
if __name__ == '__main__':
    char_weights = [('a', 5), ('b', 4), ('c', 10), ('d', 8), ('f', 15), ('g',2)]
    tree = HuffmanTree(char_weights)
    tree.get_code()
```

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

**归纳步骤** 证明：假设Huffman算法对于规模为$k$的字符集都得到最优前缀码，那么对于规模为$k+1$的字符集也得到最有前缀码。

### 归纳基础

$n=2$，字符集$C=\{x_1,x_2\}$

对任何代码的字符至少都需要1位二进制数字。Huffman算法得到的代码是0和1，是最优前缀码。


![最优前缀码](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_22-43-56.png)

### 归纳步骤

假设Huffman算法对于规模为$k$的字符集都得到最有前缀码。考虑规模为$k+1$的字符集

$C=\{x_1,x_2,...,x_{k+1}\}$，

其中$x_1,x_2\in C$是频率最小的两个字符。

令 $C'=(C-\{x_1,x_2\}) \cup \{z\}, \\ \quad \quad f(z)=f(x_1)+f(x_2)$

根据归纳假设，算法得到一颗关于字符集$C'$，频率$f(z)$和$f(x_i)(i=3,4,...,k+1)$的最有前缀码的二叉树$T'$。

把$x_1,x_2$作为$z$的儿子附到$T'$上，得到树$T$，那么$T$是关于$C=(C'-\{z\}\cup \{x_1,x_2\})$的最有前缀码的二叉树。

![最优前缀码](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_22-44-05.png)

如果不然，则存在更优树$T^*$，$B(T^*)<B(T)$，且由引理1，其树叶兄弟是$x_1$和$x_2$。

去掉$T^*$中$x_1$和$x_2$，得到$T^{*'}$。根据引理2，

$B(T^{*'})=B(T^*)-(f(x_1)+f(x_2)) \\ < B(T)-(f(x_1)+f(x_2)) \\ =B(T')$

与$T'$是一颗关于$C'$的最有前缀码的二叉树矛盾。

### 应用：文件归并

问题：给定一组不同长度的排好序文件构成的集合$S=\{f_1,...,f_n\}$，其中$f_i$表示第$i$个文件含有的项数。使用二分归并将这些文件归并成一个有序文件。

归并过程对应于二叉树：文件为树叶，$f_i$与$f_j$归并的文件是它们的父节点。

### 两两顺序归并

实例：$S=\{21,10,32,41,18,70\}$

![两两顺序归并](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_22-44-19.png)

(1) $(21+10-1)+(32+41-1)+(18+70-1)+(31+73-1)+(104+88-1)=483$

(2) $(21+10+32+41)\times 3+(18+70)\times 2-5=483$

代价计算公式：$\sum\limits_{i\in S}d(i)f_i-(n-1)$

### 实例：Huffman树归并

输入：$S=\{21,10,32,41,18,70\}$

![Huffman树归并](https://cdn.jsdelivr.net/gh/jnhu76/Image-Hosting@master/img/Snipaste_2021-09-30_22-44-31.png)

代价：$(10+18)\times 4+21\times 3+(70+41+32)\times 2 - 5=456$

## 最小生成树

## Prim算法

## Kruskal算法

## 单源最短路径问题及算法

## Dijkstra算法的证明


