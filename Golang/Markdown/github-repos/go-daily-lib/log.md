# Golang daily lib -- log

## Introduction 简介

在日常开发中，日志功能是必不可少的，虽然可以使用`fmt`库输出一些信息，但灵活性不够；golang提供了标准库`log`，可以实现基本的日志功能

## Quick Start 快速开始

 ```go
 func main() {
 	user := User{
 		ID:       1,
 		Username: "Kesa",
 	}
 
 	log.Printf("User: %s login,ID:%d", user.Username, user.ID)
 	log.Fatalf("Warning hacker %s detected", user.Username)
 
 	// do not execute
 	log.Panicf("User:%s[ID:%d] login failed", user.Username, user.ID)
 
 }
 ```

```sh
$  go run ./main.go
2021/10/27 11:59:15 User: Kesa login,ID:1
2021/10/27 11:59:15 Warning hacker Kesa detected
exit status 1
```

`log`提供了三组函数:

- `Print/Printf/Println`:正常输出日志
- `Panic/Panicf/Panicln`：输出日志后以拼装好的字符串作为参数调用`panic`
- `Fatal/Fatalf/Fatalln`:输出日志后，调用`os.Exit(1)`退出程序

`log`默认输出至标准错误(`stderr`)，每条日志之前会加上日期和时间；若日志信息不是以换行符结尾的则会加上换行符，即每条日志均会在新行中打印

虽然日志均会在新行打印，但是`Print`和`Println`是有区别的,看下面的示例：

```go
log.Print("A", 1, 2, "B")
log.Println("A", 1, 2, "B")
```

```sh
2021/10/27 14:17:38 A1 2B
2021/10/27 14:17:38 A 1 2 B
```

`Print`只会在两个均不为`string`型的参数间添加空格，如`A1 2B`;而`Println`会在所有的参数之间加空格,如`A 1 2 B`

## 定制

### 前缀

`log.SetPrefix`会为每条日志文本前添加一个前缀

```go
type User struct {
	ID       int
	Username string
}

func main() {
	user := User{
		ID:       1,
		Username: "Kesa",
	}

	// set prefix
	log.SetPrefix("[Login]")

	log.Printf("User: %s login,ID:%d", user.Username, user.ID)
}
```

```
[Login]2021/10/27 14:25:08 User: Kesa login,ID:1
[Login]2021/10/27 14:25:08 Warning hacker Kesa detected
```

### 选项

设置选项可以在每条输出的文本前添加额外信息，如时间，文件名等

`log`提供了6个选项：

```go
// src/log/log.go
const (
	Ldate         = 1 << iota     // the date in the local time zone: 2009/01/23
	Ltime                         // the time in the local time zone: 01:23:23
	Lmicroseconds                 // microsecond resolution: 01:23:23.123123.  assumes Ltime.
	Llongfile                     // full file name and line number: /a/b/c/d.go:23
	Lshortfile                    // final file name element and line number: d.go:23. overrides Llongfile
	LUTC                          // if Ldate or Ltime is set, use UTC rather than the local time zone
	Lmsgprefix                    // move the "prefix" from the beginning of the line to before the message
	LstdFlags     = Ldate | Ltime // initial values for the standard logger
)
```

- `Ldate`：输出当地时区的日期，如:`2021/10/27`
- `Ltime`:输出当地时区的时间,如：`14:31:21`
- `Lmicroseconds`:输出时间精确到微秒，设置此选项就无需设置`Ltime`,如`11:45:45.123123`
- `Llongfile`:输出长文件名和行号，含包名，如：`/a/b/c/d.go:23`
- `Lshortfile`:输出短文件名和行号，不含包名，如：`d.go:23`
- `LUTC`：若在`Ldate`和`Ltime`之后设置，将输出UTC时间
- `Lmsgprefix`:将设置的日志前缀从行首移动至日志信息的开头
- `LstdFlags`：logger的默认设置，设置`Ldate`和`Ltime`

使用`log.SetFlags`设置选项，可以设置多个：

```go
func main() {
	user := User{
		ID:       1,
		Username: "Kesa",
	}

	// set prefix
	log.SetPrefix("[Login]")

	// set flag
	log.SetFlags(log.Llongfile | log.Ldate | log.Ltime | log.Lmsgprefix)

	log.Printf("User: %s login,ID:%d", user.Username, user.ID)
}
```

```sh
2021/10/27 16:14:17 main.go:22: [Login]User: Kesa login,ID:1
```

可以看到前缀`[login]`被移动到日志信息之前了

调用`log.Flags()`可以查看当前的设置,由于设置返回的值为整型，不够直观，可以编写函数转成字符串，在上述例子中添加以下代码：

```go
func main(){
    // ...
    fmt.Println(LogFlagsToString(log.Flags()))
    // ...
}
func LogFlagsToString(flags int) string {
	var buffer bytes.Buffer

	if flags&log.Ldate == log.Ldate {
		buffer.WriteString("|Ldate")
	}
	if flags&log.Ltime == log.Ltime {
		buffer.WriteString("|Ltime")
	}
	if flags&log.Lmicroseconds == log.Lmicroseconds {
		buffer.WriteString("|Lmicroseconds")
	}
	if flags&log.Llongfile == log.Llongfile {
		buffer.WriteString("|Llongfile")
	}
	if flags&log.Lshortfile == log.Lshortfile {
		buffer.WriteString("|Lshortfile")
	}
	if flags&log.LUTC == log.LUTC {
		buffer.WriteString("|Ldate")
	}
	if flags&log.Lmsgprefix == log.Lmsgprefix {
		buffer.WriteString("|Lmsgprefix")
	}
	if flags&log.LstdFlags == log.LstdFlags {
		buffer.WriteString("|LstdFlags")
	}

	if buffer.Len() <= 0 {
		return ""
	}
	return buffer.String()[1:]
}
```

### 自定义

实际上，`log`库定义了一个默认的Logger `std`,意为标准日志，直接调用的`log`库的函数实际上内部会调用`std`的方法

```go
// src/log/log.go
var std = New(os.Stderr, "", LstdFlags)
...
func Printf(format string, v ...interface{}) {
  std.Output(2, fmt.Sprintf(format, v...))
}

func Fatalf(format string, v ...interface{}) {
  std.Output(2, fmt.Sprintf(format, v...))
  os.Exit(1)
}

func Panicf(format string, v ...interface{}) {
  s := fmt.Sprintf(format, v...)
  std.Output(2, s)
  panic(s)
}
```

也可以自定义Logger:

```go
// create-logger/main.go
type User struct {
	ID       int
	Username string
}

func main() {
	u := User{
		ID:       1,
		Username: "Kesa",
	}
	buf := &bytes.Buffer{}
	logger := log.New(buf, "[login]", log.LstdFlags|log.Lmsgprefix)
	logger.Printf("User %s(ID:%d) login ", u.Username, u.ID)
	fmt.Print(buf.String())
}
```

```sh
2021/10/27 17:21:55 [login]User Kesa(ID:1) login 
```

`log.New`接受三个参数：

- `io.Writer`:日志信息会写入其中
- `prefix`:日志前缀
- `flag`:日志选项

上面的示例中将日志写入`bytes.Buffer`中，之后将`buf`打印到标准输出

`log.New`的第一个参数可以使用`io.MultiWriter`实现多目的输出，下例将日志同时输出至标准输出、buffer和文件中

```go
type User struct {
	ID       int
	Username string
}

func main() {
	u := User{
		ID:       1,
		Username: "Kesa",
	}

	writer1 := &bytes.Buffer{}
	writer2 := os.Stdout
	writer3, err := os.OpenFile("./multi.log", os.O_WRONLY|os.O_CREATE, 0755)
	if err != nil {
		log.Fatal("create file failed:", err)
	}
	logger := log.New(io.MultiWriter(writer1, writer2, writer3), "[Multi]", log.LstdFlags|log.Lmsgprefix)
	logger.Printf("%s login,ID:%d", u.Username, u.ID)
}
```

```sh
go run ./main.go 
2021/10/27 20:49:35 [Multi] Kesa login,ID:1
Buf:2021/10/27 20:49:35 [Multi] Kesa login,ID:1
```

同时生成文件`multi.log`:

```
2021/10/27 20:52:32 [Multi]Kesa login,ID:1
```

#### 实现

`log`库的核心是`Output`方法：

```go
// src/log/log.go
func (l *Logger) Output(calldepth int, s string) error {
  now := time.Now() // get this early.
  var file string
  var line int
  l.mu.Lock()
  defer l.mu.Unlock()
  if l.flag&(Lshortfile|Llongfile) != 0 {
    // Release lock while getting caller info - it's expensive.
    l.mu.Unlock()
    var ok bool
    _, file, line, ok = runtime.Caller(calldepth)
    if !ok {
      file = "???"
      line = 0
    }
    l.mu.Lock()
  }
  l.buf = l.buf[:0]
  l.formatHeader(&l.buf, now, file, line)
  l.buf = append(l.buf, s...)
  if len(s) == 0 || s[len(s)-1] != '\n' {
    l.buf = append(l.buf, '\n')
  }
  _, err := l.out.Write(l.buf)
  return err
}
```

如果设置了`Lshortfile`或`Llongfile`，`Output`方法中会调用`runtime.Caller`获取文件名和行号；`runtime.Caller`的参数`calldepth`表示获取调用栈向上多少层信息，当前层为0

一般的调用路径为：

- 程序中使用`log.Printf`之类的函数
- `log.Printf`内调用`std.Output`
- `Output`方法中需要获取调用`log.Printf`的文件和行号,这里`calldepth`为2
  - `calldepth`为0表示`Output`调用`runtime.Caller`的那一行信息
  - `calldepth`为1表示`log.Printf`调用`Output`的那一行信息
  - `calldepth`为2表示调用`log.Printf`的那一行信息
- 调用`formatHeader`处理前缀和选项
- 将生成的字节流写入`Writer`之中

值得注意的是此处有两个优化技巧：

- 由于`runtime.Caller`调用比较耗时，先释放锁，避免等待时间过长
- 为避免频繁的内存分配，`logger`保存了一个`[]byte`类型的`buf`，可重复使用，前缀和日志内容先写入到`buf`中，之后统一写入`Writer`，减少IO操作

## Conclusion

标准库的`log`比较小巧，可以简单使用；但若其不满足功能，也有很多优秀的开源log库可供选择，如：zap,logrus,zerolog等

## Reference 参考

1. [log](https://golang.org/pkg/log) godoc
2. [log](https://darjun.github.io/2020/02/07/godailylib/log) darjun/blog
3. [log.Print() ? log.Println() ? What the difference ?](https://groups.google.com/g/golang-nuts/c/SOibvyF6Tjo) google groups
4. [log.Print behaves like log.Println](https://github.com/golang/go/issues/2062) [golang](https://github.com/golang)/**[go](https://github.com/golang/go)** issue

