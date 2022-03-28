---
title: '总结'
date: '2021-03-28'
categories:
 - golang
publish: true
---

## 3.1 数组实现原理

**数组** 是 Go 语言基本数据类型之一，通常用**元素类型**和**最大能存储的元素个数**来描述。

### 初始化

数组由两种初始化方式：

1. **显式指定数组大小**：`arr := [3]int{1, 2, 3}`
2. 使用 **`[...]T`** 在编译期推导数组大小：`arr := [...]int{1, 2, 3}`；此种方式将在编译器被转换成第一种形式；

对于由字面量组成的数组，根据数组元素数量不同，编译器会做出两种不同的优化：

1. 若元素个数**小于或等于** 4 个，所有的变量将会在栈上初始化；
2. 若元素个数**大于** 4 个，变量会在**静态存储区**初始化然后**拷贝**到**栈**上。

若使用**字面量**或**常量**访问数组元素，Go 语言会在编译期间的静态类型检查中判断是否越界：

1. 访问数组的索引是**非整数**时，报错 “non-integer array index %v”；
2. 访问数组的索引是**负数**时，报错 “invalid array index %v (index must be non-negative)” ；
3. 访问数组的索引**越界**时，报错 “invalid array index %v (out of bounds for %d-element array)”；

若使用**变量**访问数组元素时，编译器在 SSA 代码生成期间，插入运行时方法 [`runtime.panicIndex`](https://draveness.me/golang/tree/runtime.panicIndex) 以防止发生越界错误，并触发 Panic。

## 3.2 切片的实现原理

运行时的**切片**可由  [`reflect.SliceHeader`](https://draveness.me/golang/tree/reflect.SliceHeader) 结构体表示：

```go
type SliceHeader struct {
    Data uintptr
    Len int
    Cap int
}
```

- `Data`：指向数组的**指针**；
- `Len`：当前切片的**长度**；
- `Cap`：当前切片**容量**，即 `Data` 数组的大小；

切片引入了一个**抽象层**，可以在运行期间修改其长度和范围，当底层数组长度不足时会发生扩容，此时底层的数组会发生变化，但是对使用者来说是不可见的。使用者只需和切片打交道即可。

### 初始化

切片的**初始化**有三种初始化方式：

1. 通过**下标**的方式获取数组或切片的一部分：`arr[0:3] or slice[0:3]`，是三种方法中最为底层的一种；

2. 使用**字面量**初始化：`slice := []int{1, 2, 3}` ，底层会先创建数组然后再使用 `[:]` 方式获取切片；

3. 使用 **make** 关键字：`slice := make([]int, 3)`；
   在**编译期**的类型检查期间，会保证 `cap` 大于或等于 `len`；除了校验参数，还会根据下列条件采用不同方式初始化：

   1. 切片的大小和容量是否足够小；
   2. 切片是否发生了逃逸，最终在堆上初始化；

   当切片发生**逃逸**或者**非常大**时，此时会在**堆**上初始化；若**不发生逃逸**并且**非常小**时，`make([]int, 3, 4)` 会被转换成如下代码：

   ```go
   var arr [4]int
   n := arr[:3]
   ```

   上述代码初始化数组并通过下标的方式获得对应的切片。

   在**运行时**，首先会计算切片占用内存空间并在**堆**上申请一片连续，并按如下方式计算：

   ​												$内存空间 = 切片中元素大小 \times 切片容量$​

   创建切片的过程中如果发生了以下错误将直接触发运行时错误并崩溃：

   1. 内存空间打下发生溢出；
   2. 申请的内存大于最大可分配内存；
   3. 传入的长度小于 0 或长度大于容量；

   最后申请内存时，若遇到比较小的对象会直接初始化在 Go 调度器中的 P 结构中，而大于 32 KB 的对象会在堆上初始化。

### 访问

**访问**切片的长度、容量或元素，以及将 `range` 关键字转换成更简单的循环，均在**编译期**完成。

### 追加和扩容

使用 **`append`**时，会由两种情况：

1. 新切片不需要赋值回原变量
2. 新切片赋值回原变量

最大的区别在于**覆盖原变量**则无需担心切片发生**拷贝**而影响性能。

当切片容量不足时，会调用  [`runtime.growslice`](https://draveness.me/golang/tree/runtime.growslice) 函数为**切片扩容**（为切片分配新的内存空间并拷贝原切片元素的过程）。

**运行时**根据切片的容量选择不同的策略：

1. 若**期望容量大于**当前容量的**两倍**则会使用期望容量；
2. 若当前切片的**长度**小于 1024 就会将容量翻倍；
3. 若当前切片的**长度**大于 1024 则会每次增加 25% 的容量，直到大于期望容量；

上述过程确定切片的大致容量，之后会根据切片的元素大小**对齐内存**，当元素所占字节大小为 1、8 或 2 的倍数时，将待申请的内存按数组 [`runtime.class_to_size`](https://draveness.me/golang/tree/runtime.class_to_size) 向上取整，可以提高内存分配效率并减少碎片：

```go
var class_to_size = [_NumSizeClasses]uint16{
  0, 8, 16, 24, 32, 48, 64, 80, 96, 112, 128, ...,
}
```

简单示例：

```go
func appendTest1() {
	fmt.Println("------------------------ Int64 8 Byte ------------------------")
	var arr []int64
	arr = append(arr, 1, 2, 3, 4, 5)
	fmt.Printf("Arr: %v, Len: %d, Cap: %d\n", arr, len(arr), cap(arr))
	fmt.Println("------------------------ Int32 4 Byte ------------------------")
	var arr2 []int32
	arr2 = append(arr2, 1, 2, 3, 4, 5)
	fmt.Printf("Arr: %v, Len: %d, Cap: %d\n", arr2, len(arr2), cap(arr2))
	fmt.Println("------------------------ Int16 2 Byte ------------------------")
	var arr3 []int16
	arr3 = append(arr3, 1, 2, 3, 4, 5)
	fmt.Printf("Arr: %v, Len: %d, Cap: %d\n", arr3, len(arr3), cap(arr3))
	fmt.Println("------------------------ Int8 1 Byte ------------------------")
	var arr4 []int8
	arr4 = append(arr4, 1, 2, 3, 4, 5)
	fmt.Printf("Arr: %v, Len: %d, Cap: %d\n", arr4, len(arr4), cap(arr4))
}
/*
------------------------ Int64 8 Byte ------------------------
Arr: [1 2 3 4 5], Len: 5, Cap: 6
------------------------ Int32 4 Byte ------------------------
Arr: [1 2 3 4 5], Len: 5, Cap: 6
------------------------ Int16 2 Byte ------------------------
Arr: [1 2 3 4 5], Len: 5, Cap: 8
------------------------ Int8 1 Byte -------------------------
Arr: [1 2 3 4 5], Len: 5, Cap: 8
*/
```

- 期望容量为 5 ，大于当前容量的两倍（$0 \times 2$），扩容容量即为 5；
- 期望分配内存 40 ($5 \times 8$)；
- 元素大小等于 `sys.PtrSize` ，此时向上取整到 48 字节；
- 最后新切片容量为 6 ($48 \div 6$)

### 拷贝切片

使用 `copy(a, b)` 进行切片的拷贝时，分为两种情况：

1. `copy` 在**编译期**间调用，则 `copy(a, b)`会被转换成如下代码：

   ```go
   n := len(a)
   if n > len(b) {
       n = len(b)
   }
   if a.ptr != b.ptr {
       memmove(a.ptr, b.ptr, n*sizeof(elem(a)))
   }
   ```

   上述代码中的 [`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 会负责拷贝内存。

2. `copy` 在**运行时**调用，如 `go copy(a, b)` 编译器会使用 [`runtime.slicecopy`](https://draveness.me/golang/tree/runtime.slicecopy) 替换运行期间调用的 `copy`：

   ```go
   func slicecopy(to, fm slice, width uintptr) int {
   	if fm.len == 0 || to.len == 0 {
   		return 0
   	}
   	n := fm.len
   	if to.len < n {
   		n = to.len
   	}
   	if width == 0 {
   		return n
   	}
   	...
   
   	size := uintptr(n) * width
   	if size == 1 {
   		*(*byte)(to.array) = *(*byte)(fm.array)
   	} else {
   		memmove(to.array, fm.array, size)
   	}
   	return n
   }
   ```


无论是**编译期**间还是**运行时**拷贝，都会通过 [`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 将整块内存的内容拷贝到目标的内存区域中：

![golang-slice-copy](image/2019-02-20-golang-slice-copy.png)

相较于依次拷贝元素，[`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 能够提供更好的性能。需要注意的是，整块拷贝内存仍会占用非常多的资源，在大切片上执行拷贝时要注意对性能的影响。