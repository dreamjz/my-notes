---
title: 'Benchmark'
date: '2022-04-12'
categories:
 - 'golang'
publish: true
---

## 1. Benchmark 基准测试

Go 标准库的 `testing` 框架提供了基准测试 (benchmark) 的能力，可以很容易对某一段代码进行测试。

性能测试易受环境影响，所以需要保证测试环境的稳定。

## 2. Benchmark 

### 2.1 计算第 N 个斐波那契数

新增 `fib.go` 实现函数 `fib` 用于计算第 N 个斐波那契数。

```go
func fib(n int) int {
    if n == 0 || n == 1 {
        return n
    }
    return fib(n-2) + fib(n-1)
}
```

在 `fib_test.go` 中添加测试用例：

```go
func BenchmarkFib(b *testing.B) {
    for n := 0; n < b.N; n++ {
        fib(30)
    }
}
```

- Benchmark 和单元测试一样，位于 `_test.go` 文件中；
- 函数名以 `Benchmark` 开头，参数为 `*testing.B`；

### 2.2 基准测试

`go test [module_name/package_name]` 可用于执行指定 package 内的所有测试用例。

- 运行当前 package 用例： `go test .`；
- 运行子 package 用例：`go test module_name/package_name` or `go test ./package_name`；
- 递归运行所有的 package 测试用例：`go test ./...` or `go test module_name/...`

但是 `go test` 默认不运行 benchmark 用例，需要使用参数 `-bench regexp`：

```sh
# 执行当前目录的所有 benchmark
$ go test -bench '.' .
# 执行当前目录以 Fib 开头的 benchmark
$ go test -bench '^Fib' 
# 执行当前目录以 Fib 结尾的 benchmark
$ go test -bench 'Fib$'
# 执行当前目录包含 Fib 的 benchmark
$ go test -bench 'Fib'

goos: linux
goarch: amd64
pkg: benchmark-example/fib
cpu: Intel(R) Core(TM) i7-4710HQ CPU @ 2.50GHz
BenchmarkFib-8               208           5652927 ns/op
PASS
ok      benchmark-example/fib   1.759s
```

- `BenchmarkFib-8`：测试函数名，`8` 为 `GOMAXPROCS`，默认为 CPU 核数，可通过 `-cpu  ` 设置，如 `-cpu 1,2,4`；
- `208`：表示执行次数，即 `b.N`；
- `5652927 ns/op`：表示每次执行耗时；

### 2.3 测试次数

上述的测试用例执行次数 `b.N` ，对于每个用例均不同；`N` 从 1 开始，若测试能够在 1s 内完成，`N` 的值便会增加，再次执行，并且增加量会随着增加次数变大，。

### 2.4 提升准确度

提升测试准确度的一个重要手段就是增加测试次数。

可以使用 `-benchtime` 设置测试时间和 N (执行次数) 或 `-count` 直接增加测试次数。

benchmark 的默认时间为 1s，可以使用 `-benchtime t` 来指定测试时间，例如：

```sh
$ go test -bench 'Fib' -benchtime 5s .
goos: linux
goarch: amd64
pkg: benchmark-example/fib
cpu: Intel(R) Core(TM) i7-4710HQ CPU @ 2.50GHz
BenchmarkFib-8              1012           5787959 ns/op
PASS
ok      benchmark-example/fib   6.459s
```

实际时间为 6.459s ，其包含了编译、执行、销毁等操作的耗时。

`-benchtime N` 可以直接指定执行次数，即 `b.N` 的值：

```sh
# 设置次数为 100
$ go test -bench 'Fib' -benchtime 100x .
goos: linux
goarch: amd64
pkg: benchmark-example/fib
cpu: Intel(R) Core(TM) i7-4710HQ CPU @ 2.50GHz
BenchmarkFib-8               100           5945347 ns/op
PASS
ok      benchmark-example/fib   0.604s

```

`-count n ` 可以设置 benchmark 的次数：

```sh
$ go test -bench 'Fib' -count 3 . 
goos: linux
goarch: amd64
pkg: benchmark-example/fib
cpu: Intel(R) Core(TM) i7-4710HQ CPU @ 2.50GHz
BenchmarkFib-8               205           5704860 ns/op
BenchmarkFib-8               208           5704035 ns/op
BenchmarkFib-8               199           5917521 ns/op
PASS
ok      benchmark-example/fib   5.314s
```

### 2.5 内存分配情况

`-benchmem` 可以用于查看内存分配情况。

示例：

函数 `generateWithCap` 和 `generate` 都是生成一组长度为 n 随机序列。`generateWithCap` 会在初始化切片时设置容量为元素数量 n，而 `generate` 不设置容量。

```go
func generateWithCap(n int) []int {
	rand.Seed(time.Now().UnixNano())
	nums := make([]int, 0, n)
	for i := 0; i < n; i++ {
		nums = append(nums, rand.Int())
	}
	return nums
}

func generate(n int) []int {
	rand.Seed(time.Now().UnixNano())
	nums := make([]int, 0)
	for i := 0; i < n; i++ {
		nums = append(nums, rand.Int())
	}
	return nums
}
```

测试用例：

```go
const size = 1000000

func BenchmarkGenerate(b *testing.B) {
	for n := 0; n < b.N; n++ {
		generate(size)
	}
}

func BenchmarkGenerateWithCap(b *testing.B) {
	for n := 0; n < b.N; n++ {
		generateWithCap(size)
	}
}
```

```sh
$ go test -bench 'Generate' -benchmem ./generate
goos: linux
goarch: amd64
pkg: benchmark-example/generate
cpu: Intel(R) Core(TM) i7-4710HQ CPU @ 2.50GHz
BenchmarkGenerate-8                   34          39161382 ns/op        45188500 B/op         41 allocs/op
BenchmarkGenerateWithCap-8            40          25748104 ns/op         8003584 B/op          1 allocs/op
PASS
ok      benchmark-example/generate      3.402s
```

同样生成元素个数为 $1 \times 10^6$ ， `generateWithCap` 的耗时减少了约 30%，而且内存分配只有一次。

### 2.6 测试不同的输入

不同的函数时间复杂度不同，可以通过不同的输入来验证时间复杂度。

```go
func benchGenerate(i int, b *testing.B) {
	for n := 0; n < b.N; n++ {
		generate(i)
	}
}

func BenchmarkGenerateE3(b *testing.B) {
	benchGenerate(1000, b)
}

func BenchmarkGenerateE4(b *testing.B) {
	benchGenerate(10000, b)
}

func BenchmarkGenerateE5(b *testing.B) {
	benchGenerate(100000, b)
}

func BenchmarkGenerateE6(b *testing.B) {
	benchGenerate(1000000, b)
}
```

```sh
$ go test -bench 'E\d$' -benchmem ./generate
goos: linux
goarch: amd64
pkg: benchmark-example/generate
cpu: Intel(R) Core(TM) i7-4710HQ CPU @ 2.50GHz
BenchmarkGenerateE3-8              26528             52792 ns/op           16376 B/op         11 allocs/op
BenchmarkGenerateE4-8               2836            489851 ns/op          386297 B/op         20 allocs/op
BenchmarkGenerateE5-8                230           5124790 ns/op         4654344 B/op         30 allocs/op
BenchmarkGenerateE6-8                 30          42110827 ns/op        45188403 B/op         40 allocs/op
PASS
ok      benchmark-example/generate      6.302s
```

可以看出输入 N 变为原来的 10 倍，每次调用时间也约为原来的十倍，所以可以看出此函数的时间复杂度为 *O(N)*。

## 3. 注意事项

### 3.1 RestTimer

若在 benchmark  之前，需要较为耗时的准备工作，则需要忽略这部分的耗时：

 ```go
 func BenchmarkFib(b *testing.B) {
 	for n := 0; n < b.N; n++ {
 		fib(30)
 	}
 }
 
 func BenchmarkFib2(b *testing.B) {
 	time.Sleep(time.Second * 3)
 	b.ResetTimer()
 	for n := 0; n < b.N; n++ {
 		fib(30)
 	}
 }
 
 func BenchmarkFibWithPreparation(b *testing.B) {
 	time.Sleep(time.Second * 3)
 	for n := 0; n < b.N; n++ {
 		fib(30)
 	}
 }
 ```

```sh
$ go test -bench 'Fib' -benchmem ./fib
goos: linux
goarch: amd64
pkg: benchmark-example/fib
cpu: Intel(R) Core(TM) i7-4710HQ CPU @ 2.50GHz
BenchmarkFib-8                               202           5734520 ns/op               0 B/op          0 allocs/op
BenchmarkFib2-8                              205           5850143 ns/op               0 B/op          0 allocs/op
BenchmarkFibWithPreparation-8                  1        3011438467 ns/op              96 B/op          2 allocs/op
PASS
ok      benchmark-example/fib   15.389s
```

### 3.2 StopTimer & StartTimer

若每次调用函数前后需要准备工作和清理工作，则可以使用 `StopTimer` 暂停计时 和 `StartTimer` 开始计时。

```go
func BenchmarkFib3(b *testing.B) {
	for n := 0; n < b.N; n++ {
		b.StopTimer()
		time.Sleep(time.Millisecond * 10)
		b.StartTimer()
		fib(30)
	}
}

func BenchmarkFib4(b *testing.B) {
	for n := 0; n < b.N; n++ {
		time.Sleep(time.Millisecond * 10)
		fib(30)
	}
}
```

```sh
go test -bench 'Fib[3,4]' -benchmem ./fib
goos: linux
goarch: amd64
pkg: benchmark-example/fib
cpu: Intel(R) Core(TM) i7-4710HQ CPU @ 2.50GHz
BenchmarkFib3-8              133           9265549 ns/op               0 B/op          0 allocs/op
BenchmarkFib4-8               60          18395808 ns/op               1 B/op          0 allocs/op
PASS
ok      benchmark-example/fib   5.678s
```

## Reference

1.  [Go 语言高性能编程](https://geektutu.com/post/high-performance-go.html)

