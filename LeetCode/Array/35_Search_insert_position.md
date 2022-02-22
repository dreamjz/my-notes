---
title: "35 Search insert position"
date: '2022-02-22'
categories:
 - leetcode
tags:
 - array
publish: true
---

## 1. 题目描述

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 O(log n) 的算法。

## 2. 二分查找

题中要求使用时间复杂度为 O(log n) 的算法，这里自然想到了二分查找。

在一组有序数组中，将数组一分为二，将要查询的元素和分割点进行比较，分为三种情况：

- 相等直接返回
- 元素大于分割点，在分割点右侧继续查找
- 元素小于分割点，在分割点左侧继续查找

```go
func BinarySearch(arr []int, target int) int {
    left := 0
    right := len(arr) - 1
    for left <= right {
        mid := (left + right) >> 1
        if arr[mid] < target {
            left = mid + 1
        }else if arr[mid] > target {
            right = mid - 1
        }else {
            return mid
        }
    }
    return -1
}
```

## 3. 题解

题中不仅需要找到存在的元素位置，而且针对不存在的元素需要寻找其插入位置。 分析插入位置 *pos* 需要满足的关系:

*nums[pos-1] < target <= nums[post]*

此时我们的目标就变成了「在一个有序数组中找第一个大于等于 *target* 的下标」， 我们可以使用二分法来不断逼近这个位置。

```go
func searchInsert(nums []int, target int) int {
    n := len(nums)
    left := 0
    right := n-1
    ans := n
   	
    for left <= right {
        mid := (left + right) >> 1
        if target <= nums[mid] {
            ans = mid
            right = mid - 1
        }else {
            left = mid + 1
        }
    }
}
```

将 *ans* 的默认值设置为 *len(nums)* ，可以避免进行边界值的判断。

**复杂度分析**

时间复杂度：O(logn)，其中 n 为数组的长度。二分查找所需的时间复杂度为 O(logn)。

空间复杂度：O(1)O(1)。我们只需要常数空间存放若干变量。

## Reference

[35. Search insert position](https://leetcode-cn.com/problems/search-insert-position/)
