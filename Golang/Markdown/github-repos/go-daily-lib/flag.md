# Golang Daily lib flag

## 简介

`flag`用于解析命令行选项。例如在Unix系统中`la -al`列出当前所有的文件与目录信息，其中的`-al`就是命令行选项。

命令行选项在实际开发中很常用，特别实在写工具的时候。

- 指定配置文件的路径，如`redis-server ./redis.conf`以当前目录下的配置文件`redis.conf`启动Redis服务器
- 自定义某些参数，如`python -m SimpleHTTPServer 8080`启动一个HTTP服务器，监听8080端口。若不指定，则默认监听8000端口

## 快速使用 Quick Start

先来看看`flag`库的基本使用

```go
package main

import (
	"flag"
	"fmt"
)

var (
	intFlag int
	boolFlag bool
	stringFlag string
)

func init(){
	flag.IntVar(&intFlag,"intFlag",0,"int flag value")
	flag.BoolVar(&boolFlag,"boolFlag",false,"bool flag value")
	flag.StringVar(&stringFlag,"stringFlag","default","string flag value")
}

func main(){
	flag.Parse()

	fmt.Println("int flag:",intFlag)
	fmt.Println("bool flag:",boolFlag)
	fmt.Println("string flag:",stringFlag)
}
```

可以先编译程序再运行（ENV：Manjaro linux）

```sh
 $ go build -o main ./main.go 
 $ ./main -intFlag 100 -boolFlag -stringFlag string
```

Output:

```
int flag: 100
bool flag: true
string flag: string
```

若不设置某些选项，相应的变量会取默认值

```sh
$ ./main -intFlag 100
```

Output:

```
int flag: 100
bool flag: false
string flag: default
```

也可以直接使用`go run`,此命令会先编译程序生成可执行文件，然后执行该文件，将命令行中的其他选项传给这个程序

```sh
$  go run ./main.go -boolFlag
```

Output:

```
int flag: 0
bool flag: true
string flag: default
```

可以使用`-h`显示选项信息

```sh
$ go run ./main.go -h
Usage of /tmp/go-build3218362730/b001/exe/main:
  -boolFlag
        bool flag value
  -intFlag int
        int flag value
  -stringFlag string
        string flag value (default "default")
```

总结一下，使用`flag`库的一般步骤：

- 定义一些全局变量存储选项的值，如上述的`intFlag/boolFlag/stringFlag`
- 在`init`方法中使用`flag.TypeVar`方法定义选项，这里`Type`可以为基本类型`int/uint/float64/bool`，还可以是时间间隔`time.Duration`。定义时传入变量的地址、选项名、默认值和帮助信息
- 在`main`方法中调用`flag.Parse`从`os.Args[1:]`中解析选项。因为`os.Args[0]`为可执行路径，会被剔除

注意：

`flag.Parse`必须在所有选项均定义之后调用，且`flag.Parse`调用之后不能再定义选项。在上述的例子中，将选项的定义放在`init`函数中，`init`会在`main`方法之前执行，选项定义在`flag.Parse`之前就全部做好了

## 选项格式

`flag`库支持三种命令行选项格式

- -flag
- -flag=x
- -flag x

`-`和`--`都可以使用，它们的作用是一样的。有些库使用`-`表示短选项，`--`表示长选项。相对而言，`flag`使用起来较为简单

`-flag`形式仅支持bool类型的选项，出现即为`true`,不出现即为默认值。`-falg x`形式不支持bool类型的选项。因为这种形式的bool选项在Unix系统中会出现意想不到的行为。例如：

```sh
$ cmd -x *
```

`*`为shell通配符。若由名字为0，false的文件，bool选项`-x`将会取`false`，反之取`true`,同时这个选项消耗了一个参数。若要显式设置一个bool选项为`false`，只能使用`-flag=false`的形式。

遇到第一个非选项参数(即非`-`或`--`开头的)或终止符`--`,解析停止。例如：

```sh
$ go run ./main.go noflag -intFlag 100
```

Output:

```
int flag: 0
bool flag: false
string flag: default
```

可以看到解析遇到`noflag`就停止了，后续的选项`-intFlag`未被解析，所有的选项将取默认值

```sh
$ go run ./main.go -intFlag 100 -- -boolFlag=true
```

Output:

```
int flag: 100
bool flag: false
string flag: default
```

首先解析了选项`-intFlag`,值为100。在遇到`--`后解析终止，后续的`-boolFlag`未被解析，取默认值`false`。

解析终止后若还有命令行参数，`flag`库会存储下来，通过`flag.Args`方法返回这些参数的切片。可以通过`flag.NArg`函数获取未解析的参数数量，`flag.Arg(i)`访问位置`i`（从0开始）上的参数。选项个数也可通过调用`flag.NFlag`函数获取

修改下上面的程序：

```go
func main(){
	flag.Parse()

	fmt.Println("Non-flag command-line arguments:",flag.Args())
	fmt.Println("The number of non-flag command-line arguments:",flag.NArg())
	for i:=0;i<flag.NArg();i++{
		fmt.Printf("%d'th argument remaining after flags have been processed:%s\n",i,flag.Arg(i))
	}
	fmt.Println("The number of flags have been set:",flag.NFlag())

	fmt.Println("int flag:",intFlag)
	fmt.Println("bool flag:",boolFlag)
	fmt.Println("string flag:",stringFlag)
}
```

```sh
$ go run ./main.go -intFlag 100 -- -boolFlag=true -stringFlag SS
```

Output:

```
Non-flag command-line arguments: [-boolFlag=true -stringFlag SS]
The number of non-flag command-line arguments: 3
0'th argument remaining after flags have been processed:-boolFlag=true
1'th argument remaining after flags have been processed:-stringFlag
2'th argument remaining after flags have been processed:SS
The number of flags have been set: 1
int flag: 100
bool flag: false
string flag: default
```

在遇到`--`解析终止后，剩余参数`-boolFlag=true -stringFlag SS`保存在`flag`中，可以通过`Args/NArg/Arg`等函数访问。

整数选项值可以接受1234（十进制）、0664（八进制）和0x1234（十六进制）的形式，并且可以是负数。实际上`flag`在内部使用的是`strconv.ParseInt`函数将字符串解析成`int`,所以理论上，`ParseInt`接受的格式均可

Remark：

bool类型的选项值可以为：

- 取值为`true`:1,t,T,true,TRUE,True
- 取值为`false`:0,f,F,false,FALSE,False

## 另一种定义选项的方式

上面我们介绍了使用`flag.TypeVar`定义选项，这种方式需要我们先定义变量。还有一种方式，调用`flag.Type`（其中`Type`可为`Int/Uint/Bool/Float64/String/Duration`等）会自动分配变量，返回该变量的地址，用法和前一种方法类似。

```go
package main

import (
	"flag"
	"fmt"
)

var (
	intFlag    *int
	boolFlag   *bool
	stringFlag *string
)

func init() {
	intFlag = flag.Int("intFlag", 0, "int flag value")
	boolFlag = flag.Bool("boolFlag", false, "boolean flag value")
	stringFlag = flag.String("stringFlag", "default", "string flag value")
}

func main() {
	flag.Parse()

	fmt.Println("Int flag :", *intFlag)
	fmt.Println("Bool flag", *boolFlag)
	fmt.Println("String flag", *stringFlag)
}

```

```sh
$ go run ./main.go -intFlag 100 -boolFlag -stringFlag TEST
```

```
Int flag : 100
Bool flag true
String flag TEST
```

除了使用时需要解引用，其他的方式基本相同

## 高级用法

### 定义短选项

`flag`库没有显式支持短选项，但是可以通过给某个相同的变量设置不同的选项来实现。即两个选项共享同一个变量，由于初始化顺序不确定，故必须保证两者拥有相同的默认值，否则不传该选项时，行为是不确定的。

```go
package main

import (
	"flag"
	"fmt"
)

var (
	logLevel string
)

const (
	defaultLevel = "debug"
	usage        = "set log level value"
)

func init() {
	flag.StringVar(&logLevel, "logLevel", defaultLevel, usage)
	flag.StringVar(&logLevel, "l", defaultLevel, usage+"(shorthand)")
}

func main() {
	flag.Parse()

	fmt.Println("Log level:", logLevel)
}
```

```sh
$ go run ./main.go
```

```
Log level: debug
```

```sh
$ go run ./main.go -l info
```

```
Log level: info
```

```sh
$ go run ./main.go -logLevel info
```

```
Log level: info
```

### 解析时间间隔

除了能使用基本类型作为选项，`flag`库还支持`time.Duration`类型，即时间间隔。其支持的格式较多，如"300ms","-1.5h","2h45m"等。时间单位可以是`ns/us/ms/s/m/h/d`等。实际上`flag`内部会调用`time.ParseDuration`，具体可参照`time`包文档。

```go
package main

import (
	"flag"
	"fmt"
	"time"
)

var (
	period time.Duration
)

func init() {
	flag.DurationVar(&period, "period", 1*time.Second, "set sleep period")
}

func main() {
	flag.Parse()

	fmt.Printf("Sleeping for %v\n", period)
	time.Sleep(period)
	fmt.Println("Wake up ...")
}

```

```sh
$ go run ./main.go -period 5000ms
Sleeping for 5s
Wake up ...
```

### 自定义选项

除了使用`flag`库提供的选项类型，还可以自定义类型。我们来分析下面的案例：

```go
package main

import (
	"errors"
	"flag"
	"fmt"
	"strings"
	"time"
)

var FlagHasBeenSet error = errors.New("flag has been set")

type interval []time.Duration

func (i interval) String() string {
	return fmt.Sprint(([]time.Duration)(i))
}

func (i *interval) Set(value string) error {
	if len(*i) > 0 {
		return FlagHasBeenSet
	}
	for _, v := range strings.Split(value, ",") {
		d, err := time.ParseDuration(v)
		if err != nil {
			return err
		}
		*i = append(*i, d)
	}
	return nil
}

var intervalFlag interval

func init() {
	flag.Var(&intervalFlag, "deltaT", "comma-seperated list of intervals to use between events ")
}

func main() {
	flag.Parse()

	fmt.Println(intervalFlag)
}

```

首先定义类型`interval`

新类型必须实现`flag.Value`接口

```go
// src/flag/flag.go
type Value interface {
  String() string
  Set(string) error
}
```

其中`String`方法格式化该类型的值，`flag.Parse`函数在执行时遇到自定义类型的选项会将选项值作为参数调用该类型变量的`Set`方法。示例中将`,`分隔的时间间隔解析至切片中。

自定义类型选项的定义必须使用`flag.Var`方法

```sh
$ go run ./main.go -deltaT 30ms,1m,1h,1us  
[30ms 1m0s 1h0m0s 1µs]
```

## 解析程序中的字符串

有些时候选项并不是通过命令行传递的。例如，从配置表中读取或程序生成的。此时可以使用`flag.FlagSet`结构的相关方法来解析这些选项。

实际上，我们之前调用的`flag`库的方法，都会间接调用`FlagSet`结构的方法。`flag`库中定义了一个`FlagSet`类型的全局变量`CommandLine`专门用于解析命令行选项。调用的`flag`库函数只是为了提供便利，其内部都是调用了`CommandLine`的相应方法。

```go
// src/flag/flag.go
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)

func Parse() {
  CommandLine.Parse(os.Args[1:])
}

func IntVar(p *int, name string, value int, usage string) {
  CommandLine.Var(newIntValue(value, p), name, usage)
}

func Int(name string, value int, usage string) *int {
  return CommandLine.Int(name, value, usage)
}

func NFlag() int { return len(CommandLine.actual) }

func Arg(i int) string {
  return CommandLine.Arg(i)
}

func NArg() int { return len(CommandLine.args) }
```

同样的，我们也可以自己创建`FlagSet`类型变量来解析选项

```go
package main

import (
	"flag"
	"fmt"
)

func main() {
	args := []string{"-intFlag", "12", "-stringFlag", "test"}

	var intFlag int
	var boolFlag bool
	var stringFlag string

	fs := flag.NewFlagSet("MyFlagSet", flag.ContinueOnError)
	fs.IntVar(&intFlag, "intFlag", 0, "set int flag value")
	fs.BoolVar(&boolFlag, "boolFlag", false, "set bool flag value")
	fs.StringVar(&stringFlag, "stringFlag", "default", "set string flag value")

	fs.Parse(args)

	fmt.Println("int flag:", intFlag)
	fmt.Println("bool flag:", boolFlag)
	fmt.Println("string flag:", stringFlag)
}
```

`NewFlagSet`函数有两个参数，第一个是程序名称，输出帮助或出错时会显示该信息；第二个是解析出错是如何处理，有一下选项：

- `ContinueOnError`: 发生错误后继续解析
- `ExitOnError`    : 出错时调用`os.Exit(2)`退出程序
- `PanicOnError`   : 出错时产生panic

```go
// src/flag/flag.go
func (f *FlagSet) Parse(arguments []string) error {
  f.parsed = true
  f.args = arguments
  for {
    seen, err := f.parseOne()
    if seen {
      continue
    }
    if err == nil {
      break
    }
    switch f.errorHandling {
    case ContinueOnError:
      return err
    case ExitOnError:
      os.Exit(2)
    case PanicOnError:
      panic(err)
    }
  }
  return nil
}
```

与直接使用`flag`库的函数不同，`FlagSet`的`Parse`方法需要显式传入string切片作为参数。因为`flag.Parse`在内部使用了`CommandLine.Parse(os.Arg[1:])`.

## 参考

1. [flag](https://github.com/darjun/go-daily-lib)  github Repo

2. [flag](https://darjun.github.io/2020/01/10/godailylib/flag/)  darjun blog
3. [flag](https://pkg.go.dev/flag)  godoc

