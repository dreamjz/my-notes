---
title: "1. Two Sum"
date: '2022-03-11'
categories:
 - leetcode
tags:
 - array
publish: true
---

## 1. 题目描述

给定一个整数数组 `nums` 和一个整数目标值 `target`，请在该数组中找出 和为目标值 `target` 的 两个 整数，并返回其数组下标。

- 每种输入只会对应一个答案
- 数组中同一元素不会在答案中重复出现
- 可按任意顺序返回答案

```
示例 1：
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1]

示例 2：
输入：nums = [3,2,4], target = 6
输出：[1,2]

示例 3：
输入：nums = [3,3], target = 6
输出：[0,1]
```

## 2. 分析

题中要求返回两数之和为 `target`，即 `nums[i] + nums[j] = target`，那么我们的目标是固定一个数字 `nums[i]`，在数组中寻找 `target - nums[j]`。

## 3. 题解

### 3.1 暴力枚举

枚举数组中的每一个数 `x`，寻找数组中是否存在 `target - x`。当遍历正个数组需找时，需要注意每一个位于 `x` 之前的元素已经和 `x` 匹配过，无需在进行匹配。因为一个元素不能用两次，所以只需要在 `x` 之后寻找即可。

```go
func twoSum(nums []int, target int) []int {
    for i, x := range nums {
        for j := i + 1; j < len(nums); j++ {
            if x + nums[j] == target {
                return []int{i, j}
            }
        }
    }
    return nil
}
```

复杂度分析：

- 时间复杂度：*O(N^2^)*，N 为数组元素数量。最坏情况下，任意两个数都要被匹配一次
- 空间复杂度：*O(1)*。

### 3.2 哈希表

使用哈希表可以将寻找 `target - x` 的时间复杂度从 *O(N)* 降低到 *O(1)*。

创建一个哈希表，对于每一个 `x` ，首先查询哈希表中是否存在 `target - x` ，然后将 `x` 插入哈希表中，可以保证 `x` 不和自己匹配。

```go
func twoSum(nums []int, target int) []int {
    hashTable := map[int]int{}
    for i, x := range nums {
        if j, ok := hashTable[target - x]; ok {
            return []int{i, j}
        }
        hashTable[x] = i
    } 	   
    return nil
}
```

复杂度分析：

- 时间复杂度：*O(N)*，N 为数组元素数量。对于每个元素 `x`，寻找 `target - x` 的复杂度为 *O(1)*
- 空间复杂度：*O(N)*，N 为数组元素数量。主要是哈希表开销

## Reference

1. [leetcode 题解](https://leetcode-cn.com/problems/two-sum/solution/liang-shu-zhi-he-by-leetcode-solution/)
2. [两数之和](https://leetcode-cn.com/problems/two-sum/)

