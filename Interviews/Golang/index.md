---
index: 'Index'
date: '2022-04-02'
publish: false
---

|      |      |      |
| ---- | ---- | ---- |
|      |      |      |

## 1. Goalng 安全读写共享变量方式

- sync.Mutex 加锁
- channel
- atomic

## 2. Channel 有无缓冲的区别

- 无缓冲：`make(chan T)`， 发送和接收是**同步**的
  - 发送阻塞，直到数据被接收
  - 接收阻塞，直到读取到数据
- 有缓冲：`make(chan T, buf_size)`，发送和接收**不同步**
  - 缓冲区**满**时，发送阻塞
  - 缓冲区**空**时，接收阻塞

## 3. GMP 模型

### GMP 流程

![img](image/a4vWtvRWGQ.jpeg!large)

1. 创建一个 G
2. 加入队列：
   1. 优先加入 P 的**本地队列**；
   2. 本地队列已满则加入**全局队列**；
3. M 和 P **一对一**绑定，M 从 P 的**本地队列**获取 G 运行：
   1. 若本地队列为**空**，则从**全局队列**获取；
   2. 若全局队列为**空**，则从其他的 MP 组合中**偷取**一半数量的 G 来运行；
4. M 执行 G 若发生**阻塞**，则当前的 M 和 P 会解绑 (detach)，然后**创建** 或 **唤醒** 一个 M 与 P 绑定。

### 自旋

当 MP 组合无法获取 G 执行时，M 将进入**自旋**状态。

### 休眠

当 M 一段时间内没有获取 P 与之绑定时，M 将进入**休眠**状态。

##  4. 常用并发控制

### channel

使用无缓冲通道，其发送和接收是同步的，需要发送方和接收方都准备好才能完成发送和接收的操作。

```go
func main() {
    ch := make(chan struct{})
    go func() {
        fmt.Println("start working")
        time.Sleep(time.Second * 1)
        ch <- struct{}{}
    }
   
    <-ch
    fmt.Println("finished")
}
```

当 main goroutine 运行到 `<-ch` 接收 channel 的值时，若 channel 中没有数据，则会阻塞直到有值。

### sync.WaitGroup

`WaitGroup` 主要有三个方法：

- `func (wg *WaitGroup) Add(delta int)`：给 counter 增加 delta；
- `func (wg *WaitGroup) Done()`：给 counter 增加 -1，等价于 `Add(-1)`；
- `func (wg *WaitGroup) Wait()`：阻塞，直到 counter 为 0；

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
		defer wg.Done()
		mainTask()
	}()
	wg.Wait()
	fmt.Println("Main goroutine done")
}

func mainTask() {
	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go task(i, &wg)
	}
	wg.Wait()
	fmt.Println()
	fmt.Println("main task done")
}

func task(n int, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("%d, ", n)
}
```

```
4, 2, 3, 0, 1, 
main task done
Main goroutine done
```

Main 协程会等待 `mainTask` 的完成，而执行 `mainTask` 的协程会等待 `task` 的协程完成。

### context

context 适用于多个 goroutine 的管理。

```go
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go watch(ctx, "Watcher-1")
	go watch(ctx, "Watcher-2")
	go watch(ctx, "Watcher-3")

	time.Sleep(time.Second * 2)
	fmt.Println("Main: stop watching")
	cancel()
	time.Sleep(time.Second * 3)

}

func watch(ctx context.Context, name string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name, "done")
			return
		default:
			fmt.Println(name, "watching...")
			time.Sleep(time.Second * 1)
		}
	}
}
```

```
Watcher-1 watching...
Watcher-2 watching...
Watcher-3 watching...
Watcher-3 watching...
Watcher-2 watching...
Watcher-1 watching...
Watcher-2 watching...
Watcher-3 watching...
Watcher-1 watching...
Main: stop watching
Watcher-2 done
Watcher-3 done
Watcher-1 done
```

当执行 `cancel` 函数后，所有基于此 Context 及其衍生 Context 都将收到通知。

##  5. Slice 为 nil 和 empty Slice

Slice 的值为 nil，表示没有为此变量分配内存。

Slice 是空的，表示其底层数组没有元素，但是已经分配的内存。

## 6. 进程 线程 协程

### 进程

进程是系统进行**资源分配**和调度的基本单位，每个进程有自己独立的内存空间。

进程通信需要**进程间通讯**来进行。

进程上下文切换的开销较大（栈、寄存器、虚拟内存、文件句柄等）。

### 线程

线程是**程序执行**的最小单位，属于**内核态**。

线程通信通过共享内存（程序计数器、寄存器、栈等）。

线程上下文开销较小。

线程占用内存为几 MiB。

### 协程

协程是**用户态**的轻量级线程。

协程的调度和上下文切换由用户控制，不涉及内核态和用户态的切换，开销很小。

协程占用内存为几 KiB。

## 7. 数据竞争 (Data Race) 如何解决

数据竞争可以通过 `sync.Mutex` 加锁或 `channel` 解决。

`go run/build -race` 可以进行数据竞争的分析。

## 8. Channel

Channel 用于 go 协程间的通讯。

Channel 发送和接收都是原子性的，保证了并发安全。

### 实现原理

Channel 是一个队列，遵循先进先出原则，负责协程之间的通信。

#### 数据结构

通过 var 声明或 make 创建的 channel 变量是一个存储在函数栈上的指针，占用 8 个字节，指向堆上的 Channel 结构体。

![hchan](image/hchan.png)

```go
type hchan struct {
 closed   uint32   // channel是否关闭的标志
 elemtype *_type   // channel中的元素类型
 
 // channel分为无缓冲和有缓冲两种。
 // 对于有缓冲的channel存储数据，使用了 ring buffer（环形缓冲区) 来缓存写入的数据，本质是循环数组
 // 为啥是循环数组？普通数组不行吗，普通数组容量固定更适合指定的空间，弹出元素时，普通数组需要全部都前移
 // 当下标超过数组容量后会回到第一个位置，所以需要有两个字段记录当前读和写的下标位置
 buf      unsafe.Pointer // 指向底层循环数组的指针（环形缓冲区）
 qcount   uint           // 循环数组中的元素数量
 dataqsiz uint           // 循环数组的长度
 elemsize uint16                 // 元素的大小
 sendx    uint           // 下一次写下标的位置
 recvx    uint           // 下一次读下标的位置
  
 // 尝试读取channel或向channel写入数据而被阻塞的goroutine
 recvq    waitq  // 读等待队列
 sendq    waitq  // 写等待队列

 lock mutex //互斥锁，保证读写channel时不存在并发竞争问题
}
```

Channel 的主要组成部分：

- 用于保存 goroutine 之间传递数据的循环数组：buf；
- 用于记录循环数组当前发送和接收数据的下标：sendx 和 recvx；
- 用于保存该 chan 发送和接收数据被阻塞的 goroutine 队列：sendq 和  recvq；
- 保证 channel 写入和读取数据时线程安全的锁：lock；

### 特点

Channel 有两种类型：无缓冲和有缓冲；

Channel 有3种模式：写操作模式（单向）、读操作模式（单向）、读写操作模式（双向）

|      | 写操作模式       | 读操作模式       | 读写操作模式   |
| ---- | ---------------- | ---------------- | -------------- |
| 创建 | make(chan<- int) | make(<-chan int) | make(chan int) |

Channel 有3种状态：未初始化、正常、关闭

| 操作 | 未初始化         | 关闭                               | 正常           |
| ---- | ---------------- | ---------------------------------- | -------------- |
| 关闭 | panic            | panic                              | 正常关闭       |
| 发送 | 永远阻塞导致死锁 | panic                              | 阻塞或发送成功 |
| 接收 | 永远阻塞导致死锁 | 缓冲区为空则为零值，否则可以继续读 | 阻塞或成功接收 |

- channel 不能多次关闭，会导致 panic
- 多个协程监听同一个 channel，channel 上的数据**可能随机被一个goroutine 取走进行消费**
- 多个协程监听 同一个 channel，channel 被关闭，**所有协程都能收到退出信号**

#### 如何实现线程安全

channel 底层实现中，hchan 结构中使用 sync.Mutex 锁保证数据的读写安全。在对循环数组 buf 中的数据进行入队和出队操作时，必须获取互斥锁，才能操作 channel 数据。

#### 如何控制并发顺序

**多个 goroutine并发执行，每个 goroutine 抢到处理器的时间点不一致，无法保证 goroutine 的执行顺序**

可以使用 channel 传递信息，从而控制并发执行顺序

```go
func main() {
	var wg sync.WaitGroup
	ch1 := make(chan struct{}, 1)
	ch2 := make(chan struct{}, 1)
	ch3 := make(chan struct{}, 1)
	wg.Add(3)
	start := time.Now()
	ch1 <- struct{}{}
	go doTask("goroutine1", ch1, ch2, &wg)
	go doTask("goroutine2", ch2, ch3, &wg)
	go doTask("goroutine3", ch3, ch1, &wg)
	wg.Wait()
	end := time.Now()
	fmt.Printf("Duration: %s", end.Sub(start))
}

func doTask(name string, inCh chan struct{}, outCh chan struct{}, wg *sync.WaitGroup) {
	defer wg.Done()
	time.Sleep(1 * time.Second)
	select {
	case <-inCh:
		fmt.Println(name)
		outCh <- struct{}{}
	}
}
```

#### 死锁场景

出现死锁：

- 单个协程永久阻塞；
- 多个协程，由于竞争资源或因通信造成阻塞；

场景：

- 无缓存 channel 只写不读；
- 无缓存 channel 协程阻塞导致子协程无法执行；
- 有缓存 channel 写入超过缓冲区数量；
- 空读；
- 多个协程互相等待；

1. 无缓存 channel 只写不读
   ```go
   func main() {
   	ch := make(chan struct{})
   	ch <- struct{}{}
   }
   ```

2. 无缓存 channel 读在写之后

   ```go
   func main() {
   	ch := make(chan struct{})
   	ch <- struct{}{}
   	go func() {
   		<-ch
   		fmt.Println("Read")
   	}()
   }
   ```

3. 有缓存的 channel 写入超过缓冲区数量
   ```go
   func main() {
   	ch := make(chan int, 3)
   	ch <- 0
   	ch <- 1
   	ch <- 2
   	ch <- 3 // block
   }
   ```

4. 空读
   ```go
   func main() {
   	ch := make(chan int)
   	<-ch
   }
   ```

5. 相互等待

   ```go
   func main() {
   	var wg sync.WaitGroup
   	ch1 := make(chan int)
   	ch2 := make(chan int)
   	wg.Add(2)
   
   	go func() {
   		defer wg.Done()
   		select {
   		case <-ch1:
   			ch2 <- 0
   		}
   	}()
   
   	go func() {
   		defer wg.Done()
   		select {
   		case <-ch2:
   			ch1 <- 0
   		}
   	}()
   
   	wg.Wait()
   }
   ```


#### channel 的关闭

   通道关闭原则

   - 不要在数据接收方或有多个发送者的情况下关闭通道，即让一个唯一的通道发送者关闭通道。

   优雅的关闭通道

1. M 个接收者和一个发送者，发送者通过关闭用来传输数据的通道来传递发送结束信号；
2. 一个接收者和 N 个发送者，接收者关闭额外的信号通道来通知发送者关闭通道；注意此时发送通道并未关闭，之后未使用的通道将会被回收；
3. M 个接收者和一个发送者，采用中间调解者（缓冲为 1 的 channel），发送者或接收者中的任何一个都可以向中间调解者发送信号来关闭通道；

## 9. GC

三色标记法：

- 白色：不确定对象
- 灰色：存活对象，子对象待处理
- 黑色：存活对象

写屏障 (Write Barrier)：

当对象新增或更新时，将其着色为灰色。

流程：

1. 标记准备 (Mark Setup, 需STW)， 打开写屏障
2. 使用三色标记 (Marking，并发)
3. 标记结束 (Mark Termination, 需 STW)，关闭写屏障
3. 清理 (Sweeping，并发)

## 10. GC 触发条件

- 主动触发，调用 `rumtime.GC`
- 被动触发：
  - 用系统监控，当超过两分钟没有产生任何 GC 时，强制触发 GC
  - 使用步调（Pacing）算法，其核心思想是控制内存增长的比例

## 11. 栈空间管理

Golang 分配内存有两个地方：

- 堆：动态分配内存；
- 栈：采用**连续栈**以减少内存碎片的产生，运行时会自动执行栈扩容；

## 12. Golang 锁

- 互斥锁 `sync.Mutex`
- 读写锁 `sync.RWMutext`
- `sync.Map`

## 13. defer

1. 函数中的多个 defer 按照 后进先出(LIFO) 的顺序执行。

2. return 不是原子操作，会被拆分为：

   - 创建一个临时变量保存返回值 （若为有名的返回值则不会）， 给返回值赋值 (rval)
   - 调用 defer
   - 返回给调用者 (ret)

   ```go
   func main() {
   	fmt.Println(increase(1))
       fmt.Println(increase2(1))
   }
   
   func increase(n int) (ret int) {
   	defer func() {
   		ret++
   	}()
   
   	return n
   }
   
   func increase2(n int) int {
   	defer func() {
   		n++
   	}()
   	return n
   }
   ```

   - 首先给返回值赋值, ret = n (`increase`); 创建临时变量并赋值 n (`increase2`)
   - 调用 defer , ret++; n++
   - 返回ret，为 2 ; 返回 1，因为 defer 中的计算未影响返回值

3. defer 底层使用**链表**实现。

## 14. select

golang 的 select 实现了和 select, poll, epoll 相似的功能：监听多个描述符的读写等事件，一旦某个描述符就绪，则将发生的事情通知给关心的程序去处理。

select有个重点，就是为了监听事件执行的公平性，引入了pollorder，lockorder来确保公平性，同时case的执行也是随机性。会有很多考题考这个随机性。
而且selelct这种写法，配合for，可以将channel的阻塞式写法，变成非阻塞式（其实就是for循环，不停select，看看有没有事件就绪）。

## 15. Go 中原子操作和 CAS

CAS (Compare and Swap)，为原子操作的一种，将内存中的值和指定数据进行比较，当数值一样时将内存中的数据替换为新的值。

Go中的CAS操作是借用了CPU提供的原子性指令来实现。CAS操作修改共享变量时候不需要对共享变量加锁，而是通过类似乐观锁的方式进行检查，**本质还是不断的占用CPU 资源换取加锁带来的开销**（比如上下文切换开销）

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

var (
	counter int32          //计数器
	wg      sync.WaitGroup //信号量
)

func main() {
	threadNum := 5
	wg.Add(threadNum)
	for i := 0; i < threadNum; i++ {
		go incCounter(i)
	}
	wg.Wait()
}

func incCounter(index int) {
	defer wg.Done()

	spinNum := 0
	for {
		// 原子操作
		old := counter
		ok := atomic.CompareAndSwapInt32(&counter, old, old+1)
		if ok {
			break
		} else {
			spinNum++
		}
	}
	fmt.Printf("thread,%d,spinnum,%d\n", index, spinNum)
}
```

## 16. 内存逃逸

内存逃逸的五种情况:

1. 发送指针的指针或值包含了指针到`channel` 中，由于在编译阶段无法确定其作用域与传递的路径，所以一般都会逃逸到堆上分配。
2. slices 中的值是指针的指针或包含指针字段。一个例子是类似`[]*string` 的类型。这总是导致 slice 的逃逸。即使切片的底层存储数组仍可能位于堆栈上，数据的引用也会转移到堆中。
3. slice 由于 append 操作超出其容量，因此会导致 slice 重新分配。这种情况下，由于在编译时 slice 的初始大小的已知情况下，将会在栈上分配。如果 slice 的底层存储必须基于仅在运行时数据进行扩展，则它将分配在堆上。
4. 调用接口类型的方法。接口类型的方法调用是动态调度,实际使用的具体实现只能在运行时确定。考虑一个接口类型为 io.Reader 的变量 r。对 r.Read(b) 的调用将导致 r 的值和字节片b的后续转义并因此分配到堆上。
5. 尽管能够符合分配到栈的场景，但是其大小不能够在在编译时候确定的情况，也会分配到堆上.

有效的避免上述的五种逃逸的情况,就可以避免内存逃逸.

## 17. Go 对象的内存分配

Go的内存分配原则:

Go在程序启动的时候，会先向操作系统申请一块内存（注意这时还只是一段虚拟的地址空间，并不会真正地分配内存），切成小块后自己进行管理。

## 18. 栈和堆

栈和堆只是虚拟内存上2块不同功能的内存区域：

- 栈在高地址，从高地址向低地址增长。
- 堆在低地址，从低地址向高地址增长。

栈和堆相比优势：

- 栈的内存管理简单，分配比堆上快。
- 栈的内存不需要回收，而堆需要，无论是主动free，还是被动的垃圾回收，这都需要花费额外的CPU。
- 栈上的内存有更好的局部性，堆上内存访问就不那么友好了，CPU访问的2块数据可能在不同的页上，CPU访问数据的时间可能就上去了。

## 19. 堆内存分配

堆内存管理中主要是三部分

1. 分配内存块

2. 回收内存块, 
3. 组织内存块。

## 20. defer 中的变量

```go
func main() {
	a := 1
	defer fmt.Println("A1:", a)
	a++
	defer func() {
		fmt.Println("A2:", a)
	}()
	a++
}
```

```
A2: 3
A1: 1
```

defer 调用的函数的参数值在**定义**时确定，而函数**内部**的值需要在**执行**时确定。

## 21. new 和 make

- make 只能用来分配及初始化类型为 slice、map、chan 的数据。new 可以分配任意类型的数据；

- new 分配返回的是指针，即类型 *Type。make 返回引用，即 Type；

  ```go
  func main() {
  	s1 := new([]int)
  	s2 := make([]int, 0)
      // S1: *[]int, S2: []int
  	fmt.Printf("S1: %T, S2: %T\n", s1, s2)
  }
  ```

- new 分配的空间被清零。make 分配空间后，会进行初始化；

## 22. G0

G0 用于：

- 寻找其他普通的 G 来执行
- 创建 G
- 垃圾回收相关
- 等等

