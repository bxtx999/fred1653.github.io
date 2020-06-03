---
title: Trie字典树
date: "2016-04-25 23:28:56"
categories: "算法"
tags: ["算法"]
---

## 一、字典树
**字典树**——Trie树，又称为前缀树（Prefix Tree）、单词查找树或键树，是一种多叉树结构。
![字典树](http://7xkj4r.com1.z0.glb.clouddn.com/trie1.png)
上图是一棵__Trie__树，表示了关键字集合{“a”, “to”, “tea”, “ted”, “ten”, “i”, “in”, “inn”} 。从上图可以归纳出Trie树的基本性质：
1. 根节点不包含字符，除根节点外的每一个子节点都包含一个字符。
1. 从根节点到某一个节点，路径上经过的字符连接起来，为该节点对应的字符串。
1. 每个节点的所有子节点包含的字符互不相同。
 * 通常在实现的时候，会在结点结构中设置一个标志，用来标记该节点处是否构成一个单词（关键字）。
 * 可以看出，Trie树的关键字一般都是字符串，而且Trie树把每个关键字保存在一条路径上，而不是一个节点中。另外，有两个公共前缀的关键字，在Trie树种前缀部分的路径相同。所以Trie又称为前缀树。

<!-- more --> 

## 二、字典树的优缺点
### 优点
1. 插入和查询的效率很高，均是O（m），其中m是待插入/查询的字符串长度。
 * 关于查询，有人会说hash表时间复杂度是O（1）不是更快？但是哈希搜索的效率取决于哈希函数的好坏，若一个坏的hash函数导致了很多冲突，效率不一定比Trie树高
1. Trie树中不同的关键字不会产生冲突。
1. Trie树中只有在允许一个关键字关联多个值的情况下才有类似hash碰撞发生。
1. Trie树不用求hash值，对短字符串有更快的速度。通常，求hash值也是需要遍历字符串的。
1. Trie树可以对关键字按照字典序排序。
 * 字典排序（lexicographical order）是一种对于随机变量形成序列的排序方法。其方法是，按照字母顺序，或者数字小大顺序，由小到大的形成序列。
1. 每一颗Trie树都可以被看做一个简单版的确定有限状态的自动机（DFA，deterministic finite automation），也就是说，对于一个任意给定属于该自动机的状态（①）和一个属于该自动机字母表的字符（②），都可以根据给定的转移函数（③）转到下一个状态。其中：
 * ① 对于Trie树的每一个节点都确定一个自动机的状态。
 * ② 给定一个属于该自动机字母表的字符，在图中可以看到根据不同字符形成的分支；
 * ③ 从当前节点进入下一层次节点的过程进过状态转移函数得出。


 核心思想是：空间换时间，利用字符串的公共前缀来减少无谓的字符串比较以达到提高查询效率的目的。

### 缺点
1. 当hash函数很好时，Trie树的查找效率低于哈希搜索。
1. 空间消耗大。

## 三、Trie树的应用
1. 字符串检索
 检索、查询功能是Trie树最原始功能，思路就是从根节点开始一个一个字符进行比较。
 * 如果沿路比较，发现不同的字符，则表示该字符串在集合中不存在。
 * 如果所有的字符全部比较并且完全相同，还需要判断最后一个节点标识位（标记该节点是否为一个关键字）。
1. 词频统计
 Trie树常被搜索引擎用于文本词频统计。
 思路：为了实现词频统计，我们修改了节点结构，用一个整型变量`count`来计数。对每一个关键字执行插入操作，若已存在，计数加1，若不存在，插入后`count`置 1。
 (1. 2. 都可以用hash table做)
1.  字符串排序
 Trie树可以对大量字符串按字典序进行排序，思路也很简单：遍历一次所有关键字，将它们全部插入trie树，树的每个结点的所有儿子很显然地按照字母表排序，然后先序遍历输出Trie树中所有关键字即可。
1. 前缀匹配
 例如：找出一个字符串集合中所有以ab开头的字符串。我们只需要用所有字符串构造一个trie树，然后输出以a->b->开头的路径上的关键字即可。 trie树前缀匹配常用于搜索提示。如当输入一个网址，可以自动搜索出可能的选择。当没有完全匹配的搜索结果，可以返回前缀最相似的可能。
1. 作为辅助结构
 如后缀树，AC自动机
 有穷自动机 参考资料：http://blog.csdn.net/yukuninfoaxiom/article/details/6057736
1. **与哈希表相比**
 优点： 

  * trie数据查找与不完美哈希表（链表实现）在最坏情况下更快；对于trie树，最差为O（m），m为查找字符串的长度；对于不完美哈希表，会有键值冲突（不同键哈希相同），最坏为O（N），N为全部字符产生的个数。典型情况是O（m）用于哈希计算，O（1）用于数据查找。

  * trie中不同键没有冲突

  * trie的桶与哈希表用于存储键冲突的桶类似，仅在单个键与多个值关联时需要

  * 当更多的键加入到trie中，无需提供hash方法或改变hash方法

  * trie通过键为条目提供字母顺序
 缺点：

  * trie数据查找在某些情况下（磁盘或随机访问时间远远高于主存）比哈希表慢

  * 当键值为某些类型（如浮点型），前缀链很长且前缀不是特别有意义。

  * 一些trie会比hash表更消耗内存。对于trie，每个字符串的每个字符都要分配内存；对于大多数hash，只需要为整个条目分配一块内存。
1. **与二叉搜索树**相比

 二叉搜索树，又称二叉排序树，它满足：

  * 任意节点如果左子树不为空，左子树所有节点的值都小于根节点的值；

  * 任意节点如果右子树不为空，右子树所有节点的值都大于根节点的值；

  * 左右子树也都是二叉搜索树；

  * 所有节点的值都不相同。

 其实二叉搜索树的优势已经在与查找、插入的时间复杂度上了，通常只有O(log n)，很多集合都是通过它来实现的。在进行插入的时候，实质上是给树添加新的叶子节点，避免了节点移动，搜索、插入和删除的复杂度等于树的高度，属于O(log n)，最坏情况下整棵树所有的节点都只有一个子节点，完全变成一个线性表，复杂度是O(n)。

 Trie树在最坏情况下查找要快过二叉搜索树，如果搜索字符串长度用m来表示的话，它只有O(m)，通常情况（树的节点个数要远大于搜索字符串的长度）下要远小于O(n)。



## 四、实现


```

#include <iostream>

#include <string>

using namespace std;

#define ALPHABET_SIZE 26

typedef struct trie_node
{
    int count;   // 记录该节点代表的单词的个数
    trie_node *children[ALPHABET_SIZE]; // 各个子节点 
}*trie;

trie_node* create_trie_node()
{
    trie_node* pNode = new trie_node();
    pNode->count = 0;
    for(int i=0; i<ALPHABET_SIZE; ++i)
        pNode->children[i] = NULL;
    return pNode;
}

void trie_insert(trie root, char* key)
{
    trie_node* node = root;
    char* p = key;
    while(*p)
    {
        if(node->children[*p-'a'] == NULL)
        {
            node->children[*p-'a'] = create_trie_node();
        }
        node = node->children[*p-'a'];
        ++p;
    }
    node->count += 1;
}

/**
 * 查询：不存在返回0，存在返回出现的次数
 */

int trie_search(trie root, char* key)
{
    trie_node* node = root;
    char* p = key;
    while(*p && node!=NULL)
    {
        node = node->children[*p-'a'];
        ++p;
    }

    if(node == NULL)
        return 0;
    else

                return node->count;

}

int main()
{
    // 关键字集合

        char keys[][8] = {"the", "a", "there", "answer", "any", "by", "bye", "their"};

    trie root = create_trie_node();

    // 创建trie树

        for(int i = 0; i < 8; i++)

        trie_insert(root, keys[i]);

    // 检索字符串

        char s[][32] = {"Present in trie", "Not present in trie"};

    printf("%s --- %s\n", "the", trie_search(root, "the")>0?s[0]:s[1]);
    printf("%s --- %s\n", "these", trie_search(root, "these")>0?s[0]:s[1]);
    printf("%s --- %s\n", "their", trie_search(root, "their")>0?s[0]:s[1]);
    printf("%s --- %s\n", "thaw", trie_search(root, "thaw")>0?s[0]:s[1]);

    return 0;
}

```

对于Trie树，我们一般只实现插入和搜索操作。这段代码可以用来检索单词和统计词频。



## 五、Trie树改进



1. **按位树（Btiwise Trie）**：原理上和普通Trie树差不多，只不过普通Trie树存储的最小单位是字符，但是Bitwise Trie存放的是位而已。位数据的存取由CPU指令一次直接实现，对于二进制数据，它理论上要比普通Trie树快。

1. **节点压缩**

 * ①分支压缩： 对于稳定的Trie树，基本上都是查找和读取的操作，完全可以把一些分支进行压缩。例如，下图中最右侧分支inn可以直接压缩成一个节点“inn”，而不需要作为一个常规子树存在。**Radix树**就是根据这个原理来解决Trie树过深的问题。
![分支压缩](http://7xkj4r.com1.z0.glb.clouddn.com/trie2.jpg)


 * ②节点映射表：这种方式也是Trie树节点可能几乎完全确定下采用的，针对Trie树节点的每一个状态，如果状态总数重复很多的话，通过一个元素为数字的多维数组（比如Triple Array Trie）来表示，这样存储Trie树本身的空间开销会小一些，虽然引入了额外的映射表。

1. **双数组TRIE树（Double Array Trie）**

 它在保证Trie树检索速度的前提下，提高空间利用率而提出的一种数据结构，本质上还是一个确定有限自动机。（所谓DFA就是一个能够实现状态专一的自动机，对于一个给定的属于该自动机的状态和一个数据该自动机字符表Σ的字符，它能够根据预先给定的状态转移函数转移到下一个状态。）

 对于DAT来说，每个节点代表自动机的一个状态， 根据变量的不同，进行状态转移，当达到结束状态或者无法转移时，完成查询。

 参考资料：http://blog.csdn.net/zzran/article/details/8462002



 



## 六、Trie树的其他形式

![Trie树变形](http://7xkj4r.com1.z0.glb.clouddn.com/trie3.jpg)

上图主要说明下这些算法数据结构之间的关系。图中黄色部分主要写明了这些算法和数据结构的一些关键点。

图中可以看到这样一些关系：extend-kmp 是kmp的扩展；ac自动机是kmp的多串形式；它是一个有限自动机；而trie图实际上是一个确定性有限自动机；ac自动机，trie图，后缀树实际上都是一种trie；后缀数组和后缀树都是与字符串的后缀集合有关的数据结构；trie图中的后缀指针和后缀树中的后缀链接这两个概念及其一致。



## 七、Trie树的性能比较
![性能比较](http://7xkj4r.com1.z0.glb.clouddn.com/trie4.jpg)


参考博客 http://www.hankcs.com/nlp/performance-comparison-of-several-trie-tree.html







## 参考资料

1. Trie树 http://www.raychase.net/1783?replytocom=264917

1. Trie树 http://blog.csdn.net/v_july_v/article/details/6897097

1. BitWise Trie http://blog.csdn.net/breeze_gao/article/details/8461856

1. AC自动机 http://www.cppblog.com/menjitianya/archive/2014/07/10/207604.html
