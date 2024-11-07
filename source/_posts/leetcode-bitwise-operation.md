---
title: 【算法基础】Leetcode中的位运算题目
date: 2021-01-17 17:32:00
tags:
  - leetcode
abbrlink: bfde335e
---

191. 位1的个数; 231. 2的幂; 190. 颠倒二进制位。

<!-- more -->

#### 题目191. 位1的个数

[描述](https://leetcode-cn.com/problems/number-of-1-bits/)：编写一个函数，输入是一个无符号整数（以二进制串的形式），返回其二进制表达式中数字位数为 '1' 的个数（也被称为汉明重量）。

解法1：简单的按位与和移位操作

```python
def hammingWeight(self, n: int) -> int:
    count = 0
    while n != 0:
        count += n & 1
        n >> 1
    return count
```

解法2：利用位运算公式，```n & (n - 1)```可以打掉最低位的1

```python
def hammingWeight(self, n: int) -> int:
    count = 0
    while n != 0:
        count += 1
        n = n & (n -1)
    return count
```

#### 题目231. 2的幂

[给定一个整数，编写一个函数来判断它是否是 2 的幂次方。](https://leetcode-cn.com/problems/power-of-two/)

题解：2的幂，也就是二进制数只有一个1，那么继续使用去掉最低位的1的公式

```python
def isPowerOfTwo(self, n: int) -> bool:
        return n != 0 and n & (n - 1) == 0
```

#### 题目190. 颠倒二进制位

[颠倒给定的 32 位无符号整数的二进制位。](https://leetcode-cn.com/problems/reverse-bits/)

这个题目就是简单的移位操作

```python
def reverseBits(self, n: int) -> int:
    value = 0
    bits = 31
    while n:
        value += (n & 1) << bits
        n = n >> 1
        bits -= 1
    return value
```

