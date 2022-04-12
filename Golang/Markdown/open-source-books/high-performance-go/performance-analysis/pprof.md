---
title: 'pporf'
date: '2022-04-12'
categories:
 - 'golang'
publish: true
---

Benchmark 可以度量某个函数的性能，前提是知道性能的瓶颈。对于未知程序，则需要 `pprof` 工具来进行分析，[pprof](https://github.com/google/pprof) 分为两个部分：

- `runtime/pprof` 包；
- `pprof` 性能分析工具；

## 1. 性能分析类型

### 1.1 CPU

CPU 性能分析 (CPU Profiling) ，启动分析时，运行时 (runtime) 将每隔 10ms 中断一次，记录此时正在运行的协程 (goroutines) 的堆栈信息。

程序运行结束后，可以分析记录的数据找到 Hottest Code Paths。

>Compiler hot paths are code execution paths in the compiler in which most of the execution time is spent, and which are potentially executed very often.
>– [What’s the meaning of “hot codepath”](https://english.stackexchange.com/questions/402436/whats-the-meaning-of-hot-codepath-or-hot-code-path)

### 1.2 内存

内存性能分析 (Memory Profiling) 记录堆内存分配时的堆栈信息，忽略栈内存分配信息。

启用内存分析时，默认每 1000 次采样 1 次，频率可以进行调整。

### 1.3 阻塞

阻塞性能分析 (Block Profiling) 为 Golang 特有，用于记录一个协程等待一个共享资源花费的时间，在判断程序的并发瓶颈很有用。

