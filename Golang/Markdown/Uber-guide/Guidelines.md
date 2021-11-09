## Guidelines

### 指向interface的指针

你几乎(almost)不需要指向接口的指针。应该将接口作为值进行传递，在这样传递的过程中，实质上传递的底层数据仍可为指针

接口实质上在底层用两个字段表示：

1. 一个指向某些特定类型信息的指针。可以视作 “type”
2. 数据指针。如果存储的数据是指针，则直接存储。若存储的数据是值，则存储指向此值的指针

若希望接口方法修改基础数据，则必须使用指针传递（将对象指针赋值给接口变量）

```go
// Setter interface
type Setter interface{
	Set(i int64)
}

// S1 struct
type S1 struct {
	a int64
}
// S2 struct
type S2 struct {
	b int64
}
// Set implements Setter interface
// 此处的s为值类型，将为值的一个拷贝，无法修改底层数据
func (s S1 ) Set(i int64) {
	s.a=i
}

// Set implements Setter interface
// 此处s为指针类型，可以修改底层数据结构
func (s *S2) Set(i int64) {
	s.b=i
}

// PTI is a test function ,will be called in main
/*
	总结： 在接口类型上调用方法
		1. 指针方法可以通过指针调用
		2. 值方法可以通过值调用
		3. 值方法可以通过指针指针调用，指针会被自动解引用
		4. 指针方法不能被值调用，存储在接口中的值无地址
*/
func PTI(){
	var s1 Setter = S1{}
	var s2 Setter = &S2{}
	var s3 Setter = &S1{}
	fmt.Printf("S1:%T,%v S2:%T,%v,S3:%T,%v\n",s1,s1,s2,s2,s3,s3)
	s1.Set(10) // S1的Set方法Receiver为值类型，无法修改底层数据
	s2.Set(20) // S2的Set方法Receiver为指针类型，可以修改底层数据
	s3.Set(30) // S1的Set方法Receiver为值类型，无法修改底层数据
	fmt.Printf("S1:%T,%v S2:%T,%v,S3:%T,%v\n",s1,s1,s2,s2,s3,s3)
}
```

### Interface 合理性验证

- Bad

  ```go
  // 如果Handler没有实现http.Handler,会在运行时报错
  type Handler struct {
    // ...
  }
  func (h *Handler) ServeHTTP(
    w http.ResponseWriter,
    r *http.Request,
  ) {
    ...
  }
  ```

- Good

  ```go
  type Handler struct {
    // ...
  }
  // 用于触发编译期的接口的合理性检查机制
  // 如果Handler没有实现http.Handler,会在编译期报错
  var _ http.Handler = (*Handler)(nil)
  func (h *Handler) ServeHTTP(
    w http.ResponseWriter,
    r *http.Request,
  ) {
    // ...
  }
  ```

  如果 `*Handler` 与 `http.Handler` 的接口不匹配,
  那么语句 `var _ http.Handler = (*Handler)(nil)` 将无法编译通过.

赋值的右边应该是断言类型的零值

* 指针，切片和map，其零值为nil
* 结构类型，其为空结构

```go
type LogHandler struct {
  h   http.Handler
  log *zap.Logger
}
var _ http.Handler = LogHandler{}
func (h LogHandler) ServeHTTP(
  w http.ResponseWriter,
  r *http.Request,
) {
  // ...
}
```

###  接收器(Receiver)与接口

接口调用方法时：

1. 值方法可以通过值调用
2. 指针方法可以通过指针或addressable value调用
3. 值方法可以通过指针调用，指针会自动解引用
4. 指针方法不可以通过值调用，存储在接口中的值没有地址

```go
type Sre struct {
	data string
}

func (s Sre) Read() string {
	return s.data
}

func (s *Sre) Write(str string) {
	s.data = str
}

func RM() {
	s := Sre{"A"}
	fmt.Println("Before s:", s.Read())
	s.Write("AB") // s is addressable value, like (&s).Write
	fmt.Println("After s:", s.Read())

	sVals := map[int]Sre{1: {"A"}}
	fmt.Println(sVals[1].Read())
	//fmt.Println(&sVals[1]) // cannot take address of sVals[1]
	//sVals[1].Write("AB") // 此处的sVals[1]是不可寻址(addressable value)的,编译不通过
}
```

* 接口的匹配（实现）
  * 类型实现了接口的所有方法，叫做匹配
  * 匹配方法由两种：
    1. 类型的值方法匹配接口
       给接口变量为值或者指针均可
    2. 类型的指针方法匹配接口
       只能将指针对象赋给接口变量

### 零值Mutex是有效的

`sync.Mutex`和`sync.RWMutex`是有效的，所以指向mutex的指针是不必要的

* Bad

  ```go
  mu := new(sync.Mutex)
  mu.Lock()
  ```

* Good

  ```go
  var mu sync.Mutex
  mu.Lock()
  ```

若使用结构体指针，mutex应为结构体的非指针字段。即使结构体不被导出，也不要将mutex嵌入至结构体中

* Bad

  ```go
  type SMap struct {
      sync.Mutex
      data map[string]string
  }
  
  func NewSMap() *SMap {
      return &SMap{
          data: make(map[string]string)
      }
  }
  
  func (m *SMap) Get(k string) string {
      m.Lock()
      defer m.Unlock()
      return m.data[k]
  }
  ```

  Mutex字段，Lock和Unlock方法是SMap导出的API中不可以说明的部分

* Good

  ```go
  type SMap struct {
      mu sync.Mutex 
      data map[string]string
  }
  
  func NewSMap() *SMap {
      return &SMap{
          data: make(map[string]string)
      }
  }
  
  func (m *SMap) Get(k string) string {
      m.mu.Lock()
      defer m.mu.Unlock()
      return m.data[k]
  }
  ```

  mu及其方法是SMap的实现细节，对调用者不可见

### 在边界处拷贝 Slice 和 Map

slice和map包含指向底层数据的指针，因此在复制它们时要特别注意

### 接收Slice和Map

当map和slice作为函数参数传入时，若存储了对其的引用，用户可以对其进行修改

* Bad

   ```go
   func (d *Driver) SetTrips(trips []Trip) {
     d.trips = trips
   }
   
   trips := ...
   d1.SetTrips(trips)
   
   // 你是要修改 d1.trips 吗？
   trips[0] = ...
   ```

* Good

  ```go
  func (d *Driver) SetTrips(trips []Trip) {
    d.trips = make([]Trip, len(trips))
    copy(d.trips, trips)
  }
  
  trips := ...
  d1.SetTrips(trips)
  
  // 这里我们修改 trips[0]，但不会影响到 d1.trips
  trips[0] = ...
  ```

  示例：

  ```go
  type Csm struct {
  	data []int
  }
  
  func (c *Csm) SetData(data []int) {
  	c.data = data // 将data的引用复制给c.data,针对data的改动将会影响c.data
  }
  
  func (c *Csm) SetDataWithCopy(data []int) {
  	c.data = make([]int, len(data)) // 创建新的slice
  	copy(c.data, data)              // 复制slice，针对data的修改不会影响c.data
  }
  
  func CSM() {
  	/* Bad */
  	c := Csm{}
  	data := []int{1, 2, 3}
  	c.SetData(data)
  	fmt.Printf("Before: data:%v c:%v\n", data, c)
  	data[0] = 4 // 此处对data的修改将会影响c.data
  	fmt.Println("Change data[0] to 4")
  	fmt.Printf("After: data:%v,c:%v \n", data, c)
  	/* Good */
  	c1 := Csm{}
  	data1 := []int{1, 2, 3}
  	c1.SetDataWithCopy(data1)
  	fmt.Printf("Before: data:%v c:%v\n", data1, c1)
  	data1[0] = 4 // 此处对data的修改将会影响c.data
  	fmt.Println("Change data[0] to 4")
  	fmt.Printf("After: data:%v,c:%v \n", data1, c1)
  }
  ```

  

### 返回slice或map

请注意用户对暴露内部状态的修改

* Bad
  
  ```go
  type Rsm struct {
  	mu     sync.Mutex
  	status map[string]int
  }
  
  func (r *Rsm) GetStatus() map[string]int {
  	r.mu.Lock()
  	defer r.mu.Unlock()
  	return r.status
  }
  ```
  
  
  
* Good

  ```go
  type Rsm struct {
  	mu     sync.Mutex
  	status map[string]int
  }
  func (r *Rsm) GetStatusWithCopy() map[string]int {
  	r.mu.Lock()
  	defer r.mu.Unlock()
  	status := make(map[string]int, len(r.status))
  	for k, v := range r.status {
  		status[k] = v
  	}
  	return status
  }
  ```
### 使用defer释放资源

  使用defer释放资源，诸如文件和锁

* Bad

  ```go
  p.Lock()
  if p.count < 10 {
    p.Unlock()
    return p.count
  }
  
  p.count++
  newCount := p.count
  p.Unlock()
  
  return newCount
  
  // 当有多个 return 分支时，很容易遗忘 unlock
  ```

* Good

  ```go
  p.Lock()
  defer p.Unlock()
  
  if p.count < 10 {
    return p.count
  }
  
  p.count++
  return p.count
  
  // 更可读
  ```

  defer 的开销非常小，只有在您可以证明函数执行时间处于纳秒级的程度时，才应避免这样做。使用 defer 提升可读性是值得的，因为使用它们的成本微不足道。尤其适用于那些不仅仅是简单内存访问的较大的方法，在这些方法中其他计算的资源消耗远超过 `defer`。

### Channel的size要么是1，要么是无缓冲的

channel 通常 size 应为 1 或是无缓冲的。默认情况下，channel 是无缓冲的，其 size 为零。任何其他尺寸都必须经过严格的审查。我们需要考虑如何确定大小，考虑是什么阻止了 channel 在高负载下和阻塞写时的写入，以及当这种情况发生时系统逻辑有哪些变化。(翻译解释：按照原文意思是需要界定通道边界，竞态条件，以及逻辑上下文梳理)

* Bad

  ```go
  // 应该足以满足任何情况！
  c := make(chan int, 64)
  ```

  

* Good

  ```go
  // 大小：1
  c := make(chan int, 1) // 或者
  // 无缓冲 channel，大小为 0
  c := make(chan int)
  ```

  

### 枚举从1开始

在 Go 中引入枚举的标准方法是声明一个自定义类型和一个使用了 iota 的 const 组。由于变量的默认值为 0，因此通常应以非零值开头枚举。

* Bad

  ```go
  type Operation int
  
  const (
    Add Operation = iota
    Subtract
    Multiply
  )
  
  // Add=0, Subtract=1, Multiply=2
  ```

  

* Good

  ```go
  type Operation int
  
  const (
    Add Operation = iota + 1
    Subtract
    Multiply
  )
  
  // Add=1, Subtract=2, Multiply=3
  ```

  

### 使用time来处理时间

时间处理很复杂。关于时间的错误假设通常包括以下几点。

1. 一天有 24 小时
2. 一小时有 60 分钟
3. 一周有七天
4. 一年 365 天
5. [还有更多](https://infiniteundo.com/post/25326999628/falsehoods-programmers-believe-about-time)

例如，*1* 表示在一个时间点上加上 24 小时并不总是产生一个新的日历日。

因此，在处理时间时始终使用 [`"time"`] 包，因为它有助于以更安全、更准确的方式处理这些不正确的假设。

#### time.Time表达瞬时时间

在处理时间的瞬间时使用 [`time.Time`]，在比较、添加或减去时间时使用 `time.Time` 中的方法。

* Bad

  ```go
  func isActive(now, start, stop int) bool {
    return start <= now && now < stop
  }
  ```

  

* Good

  ```go
  func isActive(now, start, stop time.Time) bool {
    return (start.Before(now) || start.Equal(now)) && now.Before(stop)
  }
  ```

#### 使用time.Duration表达时间段

* Bad

  ```go
  func poll(delay int) {
    for {
      // ...
      time.Sleep(time.Duration(delay) * time.Millisecond)
    }
  }
  poll(10) // 是几秒钟还是几毫秒?
  ```

  

* Good

  ```go
  func poll(delay time.Duration) {
    for {
      // ...
      time.Sleep(delay)
    }
  }
  poll(10*time.Second)
  ```

  在一个时间瞬间加上 24 小时，我们用于添加时间的方法取决于意图。如果我们想要下一个日历日(当前天的下一天)的同一个时间点，我们应该使用 [`Time.AddDate`]。但是，如果我们想保证某一时刻比前一时刻晚 24 小时，我们应该使用 [`Time.Add`]

  ```go
  // 加一个日历日
  newDay := t.AddDate(0 /* years */, 0 /* months */, 1 /* days */)
  // 加24小时
  maybeNewDay := t.Add(24 * time.Hour)
  ```

  

#### 对外部系统使用time.Time 和 time.Duration

尽可能在与外部系统的交互中使用time.Duratio和time.Time,例如：

* Command-line 标志：[flag]通过[time.ParseDuration] 支持time.Duration
* JSON：[encoding/json]通过其[UnmarshalJSON]方法支持将time.Time编码为[RFC 3339 ]字符串（RFC 3339为一种Internet时间格式）
* SQL：[database/sql] 支持将DATETIME或TIMESTAMP列转换为time.Time
* YAML: [`gopkg.in/yaml.v2`] 支持将 `time.Time` 作为 [RFC 3339] 字符串，并通过 [`time.ParseDuration`] 支持 `time.Duration`

当不能在交互中使用time.Duration时，请使用int或float64，并在字段中包含单位

例如：encoding/json不支持time.Duration，因此应该将单位包含在字段名中

* Bad

  ```go
  // {"interval":2}
  type Config struct {
      Interval int `json:"interval"`
  }
  ```

  

* Good

  ```go
  // {"intervalMillis":200}
  type Config struct {
      IntervalMillis int `json:"intervalMillis"`
  }
  ```

当在这些交互中不能使用time.Time，除非达成一致，否则使用string和[RFC 3399]中定义的格式时间戳。默认情况下，[time.UnmarshalText]使用此格式，并可通过[time.RFC3399]在time.Format和time.Parse中使用

尽管这在实践中并不成问题，但请记住，`"time"` 包不支持解析闰秒时间戳（[8728]），也不在计算中考虑闰秒（[15190]）。如果您比较两个时间瞬间，则差异将不包括这两个瞬间之间可能发生的闰秒。

### 错误类型

Go中有多种声明错误(Error)的选项：

* [errors.New]用于简单静态字符串的错误
* [fmt.Errorf]用于格式化的错误字符串
* 实现Error()方法的自定义类型
* 用["pkg/errors".Wrap]的Wrapped errors

返回错误时，请考虑以下因素以确定最佳选择：

* 无需额外信息的简单错误，[errors.New]足够了
* 需要检测并处理错误，则应使用自定义类型并实现Error()方法
* 需要传播下游函数返回的错误,查看本文后面有关错误包装 [section on error wrapping](#错误包装 (Error-Wrapping)) 部分的内容
* 否则，fmt.Errorf 就可以了

如果需要检测错误并且使用了error.New，请使用错误变量

* Bad

  ```go
  // pcakage foo
  func Open() error {
      return errors.New("could not open")
  }
  
  // package bar
  func Use() {
      if err := foo.Open();err != nil {
          if err.Error() == "could not open" {
              // handle
          }else {
              panic("unknown error")
          }
      }
  }
  ```

  

* Good

  ```go
  // package foo 
  var ErrCouldNotOpen = errors.New("could not open")
  
  func Open() error {
      return ErrCouldNotOpen
  }
  // package bar 
  func Use() {
      if err := foo.Open();err != nil {
          if errors.Is(err,foo.ErrCouldNotOpen) {
              // handle 
          }else {
              panic("unknown error")
          }
      }
  }
  ```

  

如果需要客户检测的错误，并且想要向其中添加更多的信息，应使用自定义类型

* Bad

  ```go
  func open(file string) error {
      return fmt.Errorf("file %q not found",file)
  }
  
  func use() {
      if err:=open("testFile.txt");err != nil {
          if strings.Contains(err.Error(),"not found"){
              // handle
          }else {
              panic("unknown error")
          }
      }
  }
  ```

  

* Good

  ```go
  type errNotFound struct {
      file string
  }
  // 实现Error()方法
  func (e errNotFound) Error() string {
      return fmt.Sprintf("file %q not found",e.file)
  }
  
  func open(file string) error {
      return errNotFound{file:file}
  }
  
  func use() {
      if err := open("testFile.txt");err != nil {
          // 使用类型断言
          if _,ok := err.(errNotFound);ok {
              // handle
          }else {
              panic("unknown error")
          }
      }
  }
  ```

  

直接导出自定义错误类型要小心，因为它们成为了程序包公共API的一部分，最好公开匹配器功能以检查错误

```go
// package foo
type errNotFound struct {
    file string
}
// 实现Error()方法
func (e errNotFound) Error() string {
    return fmt.Sprintf("file %q not found",e.file)
}

func open(file string) error {
    return errNotFound{file:file}
}

func IsNotFoundErr(err error) bool {
    _,ok := err.(ErrNotFound)
    return ok
} 

// package bar
func use() {
    if err := open("testFile.txt");err != nil {
        // 使用类型断言
        if foo.IsNotFoundErr(err) {
            // handle
        }else {
            panic("unknown error")
        }
    }
}
```

### 错误包装(Error Wrapping)

一个(函数/方法)调用失败是，有三种主要的错误传播方式：

* 如果没有要添加的其他上下文，并且想要维护原始错误类型，则返回原始错误
* 添加上下文，使用["pkg/errirs".Wrap]以便错误消息提供更多上下文，["pkg/errors".Cause]可用于提取原始错误
* 若调用者不需要检测或处理的特定错误，使用[fmt.Errorf]

建议在可能的地方添加上下文，以使获得诸如“调用服务 foo：连接被拒绝”之类的更加有用的错误，而不是“连接被拒绝”的模糊错误

在将上下文添加到返回的错误时，请避免使用"failed to "之类的短语以保持上下文简洁，这些短语会陈述明显的内容，并随着错误在堆栈中的渗透而逐渐堆积

* Bad

  ```go
  s, err := store.New()
  if err != nil {
      return fmt.Errorf(
          "failed to create new store: %v", err)
  }
  ```

  ```
  failed to x: failed to y: failed to create new store: the error
  ```

  

* Good

  ```go
  s, err := store.New()
  if err != nil {
      return fmt.Errorf(
          "new store: %v", err)
  }
  ```

  ```
  x: y: new store: the error
  ```

  但是一旦将错误发送到另一个系统，就应该明确消息是错误信息（例如使用err标记，或在日志中以Failed为前缀）

  另请参见 [Don't just check errors, handle them gracefully]. 不要只是检查错误，要优雅地处理错误

  [`"pkg/errors".Cause`]: https://godoc.org/github.com/pkg/errors#Cause
  [Don't just check errors, handle them gracefully]: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

### 处理类型断言失败

[type assertion]的单个返回值形式针对不正确的类型将产生panic，因此请始终使用 comma ok 的惯用法

* Bad

  ```go
  t := i.(string)
  ```

  

* Good

  ```go
  t,ok := i.(string)
  if !ok {
      // 优雅的处理错误
  }
  ```

### 不要 Panic

在生产环境中运行的代码必须避免出现 panic，panic是[cascading failuers] 级联失败的主要根源。如果发生错误，该函数必须返回错误，并允许调用方法决定如何处理

* Bad

  ```go
  func run(args []string){
      if len(args) == 0 {
          panic("an argument is required")
      }
      // ...
  }
  func main() {
      run(os.Args[1:])
  }
  ```

  

* Good

  ```go
  func run(args []string) error {
      if len(args) == 0 {
          return errors.New("an argument is required")
      }
      // ...
      return nil
  }
  func main() {
      // 将错误交由调用者处理
      if err := run(os.Args[1:]);err != nil {
          fmt.Println(os.Stderr,err)
          os.Exit(1)
      }
  }
  ```

panic/recover不是错误处理策略，仅当发生不可恢复的事情（例如：nil引用）时，程序才必须panic。程序初始化是一个例外：程序启动时应使程序中止的不良情况可能会引起panic

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

即使在测试代码中，也优先使用`t.Fatal`或者`t.FailNow`而不是 panic 来确保失败被标记。

* Bad

  ```go
  // func TestFoo(t *testing.T)
  
  f, err := ioutil.TempFile("", "test")
  if err != nil {
    panic("failed to set up test")
  }
  ```

  

* Good

  ```go
  // func TestFoo(t *testing.T)
  
  f, err := ioutil.TempFile("", "test")
  if err != nil {
    t.Fatal("failed to set up test")
  }
  ```

### 使用 go.uber.org/atomic



