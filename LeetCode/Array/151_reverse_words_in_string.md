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

- `strings.Fields(s string) []string`: Fields splits the string `s` around each instance of one or more consecutive white space characters, as defined by `unicode.IsSpace`, returning a slice of substrings of s or an empty slice if s contains only white space.

  ```go
  fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))
  // output
  // Fields are: ["foo" "bar" "baz"]
  ```

- `strings.Join(elems []string, sep string) string`: Join concatenates the elements of its first argument to create a single string. The separator string `sep`  is placed between elements in the resulting string.

  ```go
  s := []string{"foo", "bar", "baz"}
  fmt.Println(strings.Join(s, ", "))
  // foo, bar, baz
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

### 2.3 Deque

使用 slice 实现的双端队列，将每个单词插入队首，最后队列中的单词顺序即为反向的。

```go
func reverseWords(s string) string {
    // trim whitspace
    left, right := 0, len(s)-1
    if s[left] == ' ' {
        left++
    }
    if s[right] == ' ' {
        right--
    }
    // 单词入队
    b := make([]byte, 0)
    sdq := make(strDeque, 0)
    for left <= right {
        c := s[left]
        if c != ' ' {
            b = append(b, c)
        } else if len(b) != 0 {
            sdq = sdq.AddFront(string(b))
            b = b[:0]
        }
        left++
    }
    // 最后一个单词入队
    sdq = sdq.AddFront(string(b))
    return strings.Join(sdq, " ")
}

// 使用双端队列
type strDeque []string

func (sdq strDeque) AddFront(s string) strDeque {
    rcv := []string{s}
    return append(rcv, sdq...)
}
```

## 3.  String In Golang 

在内存中，`string` 是一个双字结构，即一个指向实际数据的**指针**和记录字符串**长度**的整数。因为指针对用户来说完全不可见，因此将 `string` 视为**值类型**（字符数组）。

字符串 `var s string = "hello"` 及其子串 `s[2:3]`， 内存结构如图所示

![7.6_fig7.4](image/7.6_fig7.4.png)

在 Golang 中字符串是**不可变的**，尝试对 `str[i]` 进行赋值会得到错误 `cannot assign to str[i]`

因此需要现将字符串转化成字节数组，然后修改数组中的元素再将其转换回字符串格式以大到修改字符串的目的。

## Reference

1. [LeetCode 官方题解](https://leetcode-cn.com/problems/reverse-words-in-a-string/solution/fan-zhuan-zi-fu-chuan-li-de-dan-ci-by-leetcode-sol/)
2. [The_way_to_go](https://github.com/unknwon/the-way-to-go_ZH_CN)
