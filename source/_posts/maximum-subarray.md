---
title: 【算法基础】Leetcode题目：最大子序列和
tags:
  - leetcode
abbrlink: cf78f348
date: 2021-01-03 23:06:46
---

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

<!-- more -->

#### 示例:

输入: [-2,1,-3,4,-1,2,1,-5,4]
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。

#### 进阶:

如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。

#### 动态规划

方法1：用两个变量，一个记录最大的和，一个记录当前的和。时间复杂度 O(n^2)。

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        max_value = nums[0]  # 最大的和
        max_sub = nums[0]  # 子最大的和
        n = len(nums)
        for i in range(1, n):
            # 如果当前值为非负数，累加继续
            if nums[i] >= 0:
                max_sub += nums[i]
            else:
                # 如果当前值为负数，那么重新开始
                max_sub = nums[i]
            max_value = max(max_value, max_sub)
                
        return max_value
```

方法2：这个解法巧妙很多，也难以理解很多。

- 分治（子问题），如果第 i 个元素被累加，那么它满足 max_sum(i) = max(max_sum(i-1), 0) + nums[i] ;
- DP方程：max_sum[i] = max(max_sum[i-1], 0) + nums[i] ;
- 找到数组 max_sum 的最大值，即是我们要找的值，max_sum 可以直接服用 nums ;

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        for i, v in enumerate(nums):
            numus[i] = max(0, numus[i-1]) + nums[i]
        return max(numus)
    
# 便于理解的写法
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        dp = numus
        for i, v in enumerate(nums):
            dp[i] = max(0, dp[i-1]) + nums[i]
        return max(dp)
```



*原题*：https://leetcode-cn.com/problems/maximum-subarray/
