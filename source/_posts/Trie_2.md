---
title: 字典树Trie2
date: "2016-04-26 23:20:49"
categories: "算法与数据结构"
tags: ["算法与数据结构", "algorithms", "data structure" "字典树", "Trie", "前缀树"]
toc: true
---

接着上一篇[字典树结构](http://hoooo.org/2016/04/25/Trie_1/)的讲解，我们接着使用`C++`和`Python`来实现字典树。

## 一、LeetCode的字典树

 在[LeetCode 208](https://leetcode.com/problems/implement-trie-prefix-tree/) 要求实现字典树。

> Implement a trie with `insert`, `search`, and `startsWith` methods.
> **Note**:
You may assume that all inputs are consist of lowercase letters `a-z`.

<!-- more -->

## 二、Python实现

```Python
# coding:utf-8
"""
Implement a trie with insert, search, and startsWith methods.

Note:
You may assume that all inputs are consist of lowercase letters a-z.

Subscribe to see which companies asked this question
"""


class TrieNode(object):
    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.data = {}
        self.is_word = False


class Trie(object):
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        """
        Inserts a word into the trie.
        :type word: str
        :rtype: void
        """
        node = self.root
        for letter in word:
            child = node.data.get(letter)
            if not child:
                node.data[letter] = TrieNode()
            node = node.data[letter]
        node.is_word = True

    def search(self, word):
        """
        Returns if the word is in the trie.
        :type word: str
        :rtype: bool
        """
        node = self.root
        for letter in word:
            node = node.data.get(letter)
            if not node:
                return False
        return node.is_word  # 判断单词是否是完整的存在在trie树中

    def starts_with(self, prefix):
        """
        Returns if there is any word in the trie
        that starts with the given prefix.
        :type prefix: str
        :rtype: bool
        """
        node = self.root
        for letter in prefix:
            node = node.data.get(letter)
            if not node:
                return False
        return True

    def get_start(self, prefix):
        """
        Returns words started with prefix
        :param prefix:
        :return: words (list)
        """
        def _get_key(pre, pre_node):
            words_list = []
            if pre_node.is_word:
                words_list.append(pre)
            for x in pre_node.data.keys():
                words_list.extend(_get_key(pre + str(x), pre_node.data.get(x)))
            return words_list

        words = []
        if not self.starts_with(prefix):
            return words
        if self.search(prefix):
            words.append(prefix)
            return words
        node = self.root
        for letter in prefix:
            node = node.data.get(letter)
        return _get_key(prefix, node)


# Your Trie object will be instantiated and called as such:
trie = Trie()
trie.insert("somestring")
trie.insert("somebody")
trie.insert("somebody1")
trie.insert("somebody3")
print trie.search("key")
print trie.search("somebody3")
print trie.get_start('some')

```

### 输出

```python
# print trie.search("key")
False
# print trie.search("somebody3")
True
# print trie.get_start('some')
['somestring', 'somebody', 'somebody1', 'somebody3']
```

采用`Class`来实现字典树：https://github.com/bdimmick/python-trie/blob/master/trie.py

## 三、C++实现

---------

[1]: https://leetcode.com/articles/implement-trie-prefix-tree/