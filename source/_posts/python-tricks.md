---
title: python tricks
date: 2021-01-30 15:32:26
tags: ["tricks"]
categories: ["Python"]
desc: Python 编程记录
---

## 1. Python set操作

`difference` 和 `symmetric_difference` 的区别：

如果 `a` 和 `b` 是集合类型，`a-b`是指所有在`a`但不在`b`中的元素。

```python
>>> a = {1, 2, 3}
>>> b = {1, 4, 5}
>>> a - b
{2, 3}
>>> b - a
{4, 5}
```

`a.symmetric_difference(b)` 或 `a & b` 的结果是放在一个集合中，是 `a-b` 和 `b-a`的并集。

```python
>>> a.symmetric_difference(b)
{2, 3, 4, 5}
>>> (a - b).union(b - a)
{2, 3, 4, 5}
```

判断列表之间的差异：

```python
list1 = ['Scott', 'Eric', 'Kelly', 'Emma', 'Smith']
list2 = ['Scott', 'Eric', 'Kelly']
​
list3 = list(set(list1) ^ set(list2))
print(list3)
​
# output
['Emma', 'Smith']
```
