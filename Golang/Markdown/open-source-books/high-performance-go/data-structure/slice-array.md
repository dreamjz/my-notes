---
title: 'Slice 性能及陷阱'
date: '2022-04-23'
categories:
 - 'golang'
publush: true
---

## 1. 切片操作及性能

Go 官方的 [SliceTricks](https://github.com/golang/go/wiki/SliceTricks) 介绍了常见的操作技巧，[Go Slice Tricks Cheat Sheet](https://ueokande.github.io/go-slice-tricks/) 将操作以图片形式呈现。

### AppendVector

![image-20220413175807111](image/image-20220413175807111.png)

#### Copy

![image-20220413180015151](image/image-20220413180015151.png)

使用  `append([]T(nil), a...)` 或 `apppend(a[:0:0], a...)`  要比  `copy(b, a)` 慢一些，

但是前者在拷贝之后的 append 操作要比后者更加高效。

###  Cut

![image-20220413184139741](image/image-20220413184139741.png)

### Delete

![image-20220413184216733](image/image-20220413184216733.png)

### Delete without preserving order

![image-20220413184412360](image/image-20220413184412360.png)

### Cut (GC)

![image-20220413184615042](image/image-20220413184615042.png)

### Delete (GC)

![image-20220413184641542](image/image-20220413184641542.png)

### Delete without preserving order (GC)

![image-20220413184725492](image/image-20220413184725492.png)

### Expand

![image-20220413184903466](image/image-20220413184903466.png)



## Reference

1. 
2. [golang slice, slicing a slice with slice abc](https://stackoverflow.com/questions/27938177/golang-slice-slicing-a-slice-with-sliceabc)

