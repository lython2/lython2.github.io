---
title: 【算法基础】斐波拉契数列
tags:
  - python
  - leetcode
abbrlink: 5d612136
date: 2020-12-27 02:06:46
---

经典的斐波拉契数列，衍生出来很多问题，比如爬楼梯问题，走格子问题。


<!-- more -->


#### 1. 递归加记忆化搜索

```python
# 使用字典做缓存
fibonacci = {0: 0, 1: 1}

def fib(n: int) -> int:
    if n <= 1:
        return n
    if n in fibonacci:
        return fibonacci[n]
    value = fib(n-1) + fib(n-2)
    fibonacci[n] = value
    return value

# 使用 lru_cache 缓存
from functools import lru_cache

@lru_cache
def fib(self, n: int) -> int:
    value = n if n <= 1 else fib(n-1) + fib(n-2)
    return value
```

#### 2. 动态规划

```python
def fib(n: int) -> int:
    if n == 0 or n == 1:
        return n
    fib_list = [0] * (n+1)
    fib_list[0] = 0
    fib_list[1] = 1
    for i in range(2, n + 1):
        fib_list[i] = fib_list[i-1] + fib_list[i-2]
    return fib_list[n]

# 第二种写法
def fib(n: int) -> int:
    if n == 0:
        return 0
    a, b = 0, 1
    while n > 1:
        a, b = b, a + b
        n -= 1
    return b
```

#### 3. 通向公式

![斐波拉契数列通项公式](/images/20210105095453467.png)

```python
def fib(n: int) -> int:
   sqrt5 = 5**0.5
   fibn = (((1 + sqrt5) / 2) ** n - ((1 - sqrt5) / 2) ** n) / sqrt5
   return round(fibn)
```

*leetcode链接*: https://leetcode-cn.com/problems/fibonacci-number/ 
