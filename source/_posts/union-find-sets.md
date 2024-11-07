---
title: 【算法基础】并查集
tags:
  - leetcode
abbrlink: 85dc8b23
date: 2021-01-13 23:06:46
---

并查集实用于组团、配对问题。


<!-- more -->

#### 基本操作

- makeSet(s)：建立一个新的并查集，其中包含 s 个单元素集合；
- unionSet(x, y)：把元素 x 和元素 y 所在的集合合并，要求 x 和 y 所在 的集合不相交，如果相交则不合并；
- find(x)：找到元素 x 所在的集合的代表，该操作也可以用于判断两个元 素是否位于同一个集合，只要将它们各自的代表比较一下就可以了；

#### 初始化

![初始化](/images/image-20210110230804740.png)


#### 查询与合并

![查询与合并](/images/image-20210110230804741.png)

#### 路径压缩

![路径压缩](/images/image-20210110230804742.png)

#### 代码模板

```python
class DisjointSet:
    
    def init(self, n):
        p = [i for i in range(n)]
        return p

    def union(self, p, i, j):
        p1 = self.parent(p, i)
        p2 = self.parent(p, j)
        p[p1] = p2

    def parent(self, p, i):
        root = i
        while p[root] != root:
            root = p[root]

        # 路径压缩
        while p[i] != i:
            i, p[i] = p[i], root
        return root
```

