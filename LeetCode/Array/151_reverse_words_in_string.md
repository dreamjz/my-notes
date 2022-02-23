---
title: "151. Reverse words in a string"
date: '2022-02-23'
categories:
 - leetcode
tags:
 - array
publish: true
---

## 1. 题目描述

给你一个字符串 s ，逐个翻转字符串中的所有 单词 。

单词 是由非空格字符组成的字符串。s 中使用至少一个空格将字符串中的 单词 分隔开。

请你返回一个翻转 s 中单词顺序并用单个空格相连的字符串。

说明：

- 输入字符串 s 可以在前面、后面或者单词间包含多余的空格。
- 翻转后单词间应当仅用一个空格分隔。
- 翻转后的字符串中不应包含额外的空格。

Example：

```
示例 1：
输入：s = "the sky is blue"
输出："blue is sky the"

示例 2：
输入：s = "  hello world  "
输出："world hello"
解释：输入字符串可以在前面或者后面包含多余的空格，但是翻转后的字符不能包括。

示例 3：
输入：s = "a good   example"
输出："example good a"
解释：如果两个单词间有多余的空格，将翻转后单词间的空格减少到只含一个。

示例 4：
输入：s = "  Bob    Loves  Alice   "
输出："Alice Loves Bob"

示例 5：
输入：s = "Alice does not even like bob"
输出："bob like even not does Alice"
```

## 2. 题解

### 2.1 使用 API

```go
func reverseWords(s string) string {
    strs := strings.Fields(s)
    reverseStrSlice(strs)
    return strings.Join(strs, " ")
}

func reverseStrSlice(strs []string) {
    n := len(strs)
    for i := 0; i < n/2; i++ {
        strs[i], strs[n-1-i] = strs[n-1-i], strs[i]
    }
}
```

### 2.2 自定义函数

```go
func reverseWords(s string) string {
    // 去除所有多余的空格
    b := trim(s)
    // 反转整个字符串
    reverseByteSlice(b, 0, len(b) - 1)
    // 反转单词
    reverseEachWord(b)
    return string(b)
}

func reverseEachWord(b []byte) {
    n := len(b)
    start, end := 0, 0
    for start < n {
        // 寻找单词边界
        for end < n && b[end] != ' ' {
            end ++
        }
        // 翻转单词
        reverseByteSlice(b, start, end-1)
        // 更新索引
        start = end + 1
        end = start
    }
}

func reverseByteSlice(b []byte, start, end int) {
    for start < end {
        b[start], b[end] = b[end], b[start]
        start++
        end--
    }
}

func trim(s string) []byte {
    var b []byte
    // 去除两边的空格
    left, right := 0, len(s)-1
    for s[left] == ' ' {
        left++
    }
    for s[right] == ' ' {
        right--
    }
    // 去除中间的多余空格
    for left <= right {
        c := s[left]
        if s[left] != ' ' {
            b = append(b, c)
        } else if b[len(b) - 1] != ' ' {
            b = append(b, c)
        }
        left++
    }
    return b
}
```

### 2.3 Stack

