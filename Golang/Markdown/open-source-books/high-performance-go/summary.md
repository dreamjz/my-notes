---
title: '总结'

date: '2022-04-12'

publish: false
---

## Benchmark

1. 进行性能测试时，尽可能保持测试环境的稳定
2. 实现 benchmark 测试
   • 位于 `_test.go` 文件中
   • 函数名以 `Benchmark` 开头
   • 参数为 `b *testing.B`
   • `b.ResetTimer()` 可重置定时器
   • `b.StopTimer()` 暂停计时
   • `b.StartTimer()` 开始计时
3. 执行 benchmark 测试
   • `go test -bench .` 执行当前测试
   • `b.N` 决定用例需要执行的次数
   • `-bench` 可传入正则，匹配用例
   • `-cpu` 可改变 CPU 核数
   • `-benchtime` 可指定执行时间或具体次数
   • `-count` 可设置 benchmark 轮数
   • `-benchmem` 可查看内存分配量和分配次数

## pprof

### 性能分析类型

- CPU 性能分析，runtime 每隔 10 ms 中断一次，记录此时正在运行的 goroutines 的堆栈信息
- 内存性能分析，记录堆内存分配时的堆栈信息，忽略栈内存分配信息，默认每 1000 次采样 1 次
- 阻塞性能分析，GO 中独有的，记录一个协程等待一个共享资源花费的时间
- 锁性能分析，记录因为锁竞争导致的等待或延时

### 生成 Profile

#### runtime/pprof

进行 CPU分析，需在 `main` 函数中添加如下：

```go
func main() {
    pprof.StartCPUProfile(w)
    defer pprof.StopCPUProfile
    
    // rest of the program
}
```

进行 内存 分析，需在 `main` 函数添加如下：

```go
func main() {    
    // rest of the program
    
    if err := pprof.WriteHeapProfile(w); err != nil {
        // ...
    }
}
```

#### net/http/pprof

可以在 Web 服务中开启分析。若没有启动的 Web 服务，则需要自行启动：

```go
func main() {
    go func(){
        log.Fatalln(http.ListenAndServe(":6060"), nil)
    }
    // ...
}
```

可以在 `/debug/pprof` 路径下看到所有类型的分析结果。

#### github.com/pkg/profile

可以引入三方库 [pkg/profile](https://github.com/pkg/profile) 进行分析；

#### benchmark 

进行 benchmark 时，可以生成 profile:

进行 benchmark 测试时可以生成 profile 文件：

- `-cpuprofile FILE`  ；
- `-memprofile FILE` ；
- `-memprofilerate N`   ，将记录速率改为原来的 $\frac {1} {N}$；
- `-blockprofile FILE` ；

### 分析 Profile

使用工具 `go tool pprof` 对生成的结果进行分析：

- `go tool pprof -http=:6060 profile_name` ：在浏览器页面查看；
- `go tool pprof profile_name`：命令行模式查看；

