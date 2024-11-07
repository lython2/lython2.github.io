---
title: 【算法基础】字典树的基本特征
tags:
  - leetcode
abbrlink: d8052ccb
date: 2021-01-14 23:06:46
---

字典树，即 Trie树，又称单词查找树或键树，是一种树形结构。典型应用是用于统计和排序大量的字符串（但不仅限于字符串），所以经常搜索引擎系统用于文本词频统计。它的优点是：最大限度地减少无谓的字符串比较，查询效率比哈希表高。


<!-- more -->

#### 基本结构

![结构示意](/images/image-20210110230804739.png)

#### 基本特征

1. 结点本身不存完整单词；
2. 从根结点到某一结点，路径上经过的字符连接起来，为该结点对应的 字符串； 
3. 每个结点的所有子结点路径代表的字符都不相同；
4. 节点可以存储额外信息，比如词频；


#### 节点内部实现

![节点内部实现](/images/image-20210110231246455.png)

Trie 树的核心思想是空间换时间。 利用字符串的公共前缀来降低查询时间的开销以达到提高效率的目的。

#### Leetcode实战题目

[208.实现Trie(前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

```python
class Trie:
    def __init__(self):
        self.root = {}
        self.end_of_word = "#"

    def insert(self, word):
        node = self.root
        for char in word:
            node = node.setdefault(char, {})
        node[self.end_of_word] = self.end_of_word

    def search(self, word):
        node = self.root
        for char in word:
            if char not in node:
                return False
            node = node[char]
        return self.end_of_word in node

    def startsWith(self, prefix):
        node = self.root
        for char in prefix:
            if char not in node:
                return False
            node = node[char]
        return True
```

