---
title: "724. Find Pivot Index"
date: '2022-02-22'
categories:
 - leetcode
tags:
 - array
publish: true
---

## 1. 题目描述

给你一个整数数组 nums ，请计算数组的 中心下标 。

数组 中心下标 是数组的一个下标，其左侧所有元素相加的和等于右侧所有元素相加的和。

如果中心下标位于数组最左端，那么左侧数之和视为 0 ，因为在下标的左侧不存在元素。这一点对于中心下标位于数组最右端同样适用。

如果数组有多个中心下标，应该返回 最靠近左边 的那一个。如果数组不存在中心下标，返回 -1 。

## 2. 示例

示例 1：

```
输入：nums = [1, 7, 3, 6, 5, 6]
输出：3
解释：
中心下标是 3 。
左侧数之和 sum = nums[0] + nums[1] + nums[2] = 1 + 7 + 3 = 11 ，
右侧数之和 sum = nums[4] + nums[5] = 5 + 6 = 11 ，二者相等。
```

示例 2：

```
输入：nums = [1, 2, 3]
输出：-1
解释：
数组中不存在满足此条件的中心下标
```

示例 3：

```
输入：nums = [2, 1, -1]
输出：0
解释：
中心下标是 0 。
左侧数之和 sum = 0 ，（下标 0 左侧不存在元素），
右侧数之和 sum = nums[1] + nums[2] = 1 + -1 = 0 。
```

## 3. 初见思路

```go
func pivotIndex(nums []int) int {
    for i := 0; i < len(nums); i++ {
        left := calSlice(nums[:i])
        right := calSlice(nums[i+1:])
        if left == right {
            return i
        }
    }
    return -1
}

func calSlice(nums []int) int {
    sum := 0
    for _, v := range nums {
        sum += v
    }
    return sum
}
```

遍历数组，计算每个元素两边的和是否相等，此方式的时间复杂度较高。

## 4. 题解

记数组全部元素之和为 *total*, 当遍历到第 *i* 个元素时，设其左边的元素和为 *sum*, 则右边的元素和为 *total - nums~i~ - sum*, 若左右两边之和相等则有 *sum = total - nums~i~ - sum*, 即 *2\*sum + num~i~  =  total* 。

```go
func pivotIndex(nums []int) int {
    total := 0
    for _, v := range nums {
        total += v
    }

    sum := 0
    for i, v := range nums {
        if 2*sum + v == total {
            return i
        }
        sum += v
    }
    return -1
}
```

**复杂度分析**

- 时间复杂度：O(n)*O*(*n*)，其中 *n* 为数组的长度。
- 空间复杂度：O(1)*O*(1)。

## Reference

1. [724. find_pivot_index](https://leetcode-cn.com/problems/find-pivot-index/)