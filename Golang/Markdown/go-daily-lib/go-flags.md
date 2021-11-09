# Golang Daily lib go-flags

## 简介

`flag`库是用于解析命令行选项的。但是`flag`库有几个缺点：

- 不显式支持短选项
- 选项变量的定义比较繁琐，每个选项均需要调用对应的`Type`或`TypeVar`函数
- 默认只支持有限的数据类型，当前只有基本类型`bool/int/uint/string`和`time.Duration`

为了解决这些问题，出现了不少第三方解析命令行选项的库，今天的主角`go-flags`是其中一个。

`go-flags`提供比标准库`flag`更多的选项。它利用结构标签(struct tag)和反射提供了一个方便、简洁的借口。其除了基本的功能，还提供了丰富的特性：

- 支持短选项(-v)和长选项(-verbose)
- 支持短选项合写，如`-aux`
- 同一个选项可以设置多个值
- 支持所有的基础类型和map类型，甚至函数
- 支持命名空间和选项组

## 快速开始 Quick Start

由于是第三方库，使用前需要安装

```sh
 $ go get github.com/jessevdk/go-flags
```

或者使用go moudel file

```go
module quick-start

require github.com/jessevdk/go-flags latest
```

```sh
$ go mod download
```

```go
package main

import (
	"fmt"

	"github.com/jessevdk/go-flags"
)

type Option struct {
	Verbose []bool `short:"v" long:"verbose" description:"Show verbose debug message"`
}

func main() {
	var opt Option
	flags.Parse(&opt)

	fmt.Println(opt.Verbose)
}

```

使用`go-flags`的一般步骤：

- 定义选项结构，在结构标签中设置选项信息。通过`short`和`long`设置短、长选项名，`description`设置帮助信息。命令行传参时，短选项前加`-`,长选项前加`--`
- 声明选项变量
- 调用`go-flags`的解析方法解析

短选项：

```sh
$ go run ./main.go -v
[true]
```

长选项：

```sh
$ go run ./main.go --verbose
[true]
```

由于`Verbose`字段为切片类型，每次遇到`-v`,`–-verbose`都会追加一个true到切片中

多个短选项：

```sh
$ go run ./main.go -v -v -v 
[true true true]
```

多个长选项：

```sh
$ go run ./main.go --verbose --verbose
[true true]
```

短选项+长选项：

```sh
go run ./main.go --verbose -v
[true true]
```

短选项合写：

```sh
$ go run ./main.go -vvv        
[true true true]
```

## 基本特性

### 支持丰富的数据类型

`go-flags`相比标准库`flag`支持更丰富的数据类型

- 所有的基本类型(包括有符号整数`int/int8/int16/int32/int64`,无符号整数`uint/uint8/uint16/uint32/uint64`,浮点数`float32/float64`,布尔类型`bool`和字符串`string`)和它们的切片
- `map`类型；仅支持key为`string`,value为基础类型的map
- 函数类型

若字段是基本类型的切片，基本解析流程与对应的基本类型是一样的。切片类型选项的不同之处在于，遇到相同的选项时，值会被追加到切片之中。而非切片类型的选项，后出现的值会覆盖先出现的值

```go
package main

import (
	"fmt"

	"github.com/jessevdk/go-flags"
)

type Option struct {
	IntFlag        int            `short:"i" long:"int" description:"int flag value"`
	IntSlice       []int          `long:"intSlice" description:"int slice flag value"`
	BoolFlag       bool           `long:"bool" description:"bool flag value"`
	BoolSlice      []bool         `long:"boolSlice" description:"bool slice flag value"`
	FloatFlag      float64        `long:"float" description:"float flag value"`
	FloatSlice     []float64      `long:"floatSlice" description:"float slice flag value"`
	StringFlag     string         `short:"s" long:"string" description:"string flag value"`
	StringSlice    []string       `long:"stringSlice" description:"string slice flag value"`
	PtrStringSlice []*string      `short:"p" long:"prtStrSlice" description:"pointer of string slice flag value"`
	Call           func(string)   `long:"call" description:"callback"`
	IntMap         map[string]int `long:"intMap" description:"a map from string to int"`
}

func main() {
	var opt Option
	opt.Call = func(value string) {
		fmt.Println("in callback:", value)
	}
	_, err := flags.Parse(&opt)
	if err != nil {
		fmt.Println("Parse error:", err)
		return
	}

	fmt.Println("int flag:", opt.IntFlag)
	fmt.Println("int slice:", opt.IntSlice)
	fmt.Println("bool flag:", opt.BoolFlag)
	fmt.Println("bool slice:", opt.BoolSlice)
	fmt.Println("float flag:", opt.FloatFlag)
	fmt.Println("float slice:", opt.FloatSlice)
	fmt.Println("string flag:", opt.StringFlag)
	fmt.Println("string slice:", opt.StringSlice)
	fmt.Println("pointer of string slice:", opt.PtrStringSlice)
	fmt.Println("int map:", opt.IntMap)
}
```

基本类型及其切片比较简单，值得留意的是基本类型指针的切片，上述例子中的`PtrStringSlice`，类型为`[]*stirng`。由于结构中存储的是字符串指针，`go-flags`在解析过程中遇到该选项会自动创建字符串，将指针追加到切片中

```sh
$ go run ./main.go -p test1 -p test2
...
pointer of string slice:
        0: test1
        1: test2
...
```

另外，我们可以在选项中定义函数类型。该函数的唯一要求是有一个字符串类型的参数。解析中每次遇到该选项就会以选项值为参数调用此函数。上述例子中，`Call`函数简单打印了传入的参数

```sh
$ go run ./main.go --call test1 --call test2
...
in callback: test1
in callback: test2
...
```

`go-flags`还支持map类型。虽然限制键必须是`string`类型，值必须是基本类型，也可以实现比较灵活的配置。`map`类型的选项值格式为`key:value`，可以设置多个

```sh
$ go run ./main.go --intMap A:1 --intMap B:2
...
int map: map[A:1 B:2]
...
```

### 常用设置

`go-flags`提供了非常多的设置选项，具体参见[文档](https://pkg.go.dev/github.com/jessevdk/go-flags)。此处重点介绍`required`和`default`

- `required`非空时，表示对应的选项必须设置，否则解析时返回`ErrRequired`
- `default`用于设置默认。若已经设置默认值，`required`设置将会失效

```go
package main

import (
	"fmt"
	"log"

	"github.com/jessevdk/go-flags"
)

type Option struct {
	Required string `short:"r" long:"required" required:"true"`
	Default  string `short:"d" long:"default" default:"default"`
}

func main() {
	var opt Option
	_, err := flags.Parse(&opt)
	if err != nil {
		log.Fatal("Parse error:", err)
	}

	fmt.Println("Required:", opt.Required)
	fmt.Println("Default:", opt.Default)
}

```

不传入`required`选项则会报错，不传入`default`选项则会取默认值

```sh
$ go run ./main.go -d s     
the required flag `-r, --required' was not specified
2021/10/08 11:35:27 Parse error:the required flag `-r, --required' was not specified
exit status 1

$ go run ./main.go -r s    
Required: s
Default: default

```

## 高级特性

### 选项分组

```go
package main

import (
	"fmt"
	"github.com/jessevdk/go-flags"
	"os"
)

type Option struct {
	Basic GroupBasicOption `description:"basic group" group:"basic"`
	Slice GroupSliceOption `description:"slice group" group:"slice"`
}

type GroupBasicOption struct {
	IntFlag    int     `short:"i" long:"intFlag" description:"int flag"`
	BoolFlag   bool    `short:"b" long:"boolFlag" description:"bool flag"`
	FloatFlag  float64 `short:"f" long:"floatFlag" description:"float flag"`
	StringFlag string  `short:"s" long:"stringFlag" description:"string flag"`
}

type GroupSliceOption struct {
	IntSliceFlag    []int     `long:"intSlice" description:"int slice flag"`
	BoolSliceFlag   []bool    `long:"boolSlice" description:"bool slice flag"`
	FloatSliceFlag  []float64 `long:"floatSlice" description:"float slice flag"`
	StringSliceFlag []string  `long:"stringSlice" description:"string slice flag"`
}

func main() {
	var opt Option
	p := flags.NewParser(&opt, flags.Default)
	_, err := p.ParseArgs(os.Args[1:])
	if err != nil {
		return
	}
	basicGroup := p.Command.Group.Find("basic")
	for _, option := range basicGroup.Options() {
		fmt.Printf("name:%s,value:%v\n", option.LongNameWithNamespace(), option.Value())
	}
	sliceGroup := p.Command.Group.Find("slice")
	for _, option := range sliceGroup.Options() {
		fmt.Printf("name:%s,value:%v\n", option.LongNameWithNamespace(), option.Value())
	}
}

```

上述的例子中将基本类型和切片类型选项拆分到两个结构体中，这样可以使得代码看起来更清晰自然，特别是在代码量很大的情况下。

```sh
$ go run ./main.go -h                                                                          !7456
Usage:
  main [OPTIONS]

basic:
  -i, --intFlag=     int flag
  -b, --boolFlag     bool flag
  -f, --floatFlag=   float flag
  -s, --stringFlag=  string flag

slice:
      --intSlice=    int slice flag
      --boolSlice    bool slice flag
      --floatSlice=  float slice flag
      --stringSlice= string slice flag

Help Options:
  -h, --help         Show this help message

```

在帮助信息中，也是按照我们的设定分组显示了，便于查看

### 子命令

`go-flags`支持子命令。我们经常使用的 Go和Git命令行中就存在大量的子命令。例如：`go version`,`go build`,`go run`,`git status`,`git commit`其中的`version`,`build`等就为子命令。

```go
package main

import (
	"errors"
	"fmt"
	"github.com/jessevdk/go-flags"
	"strconv"
	"strings"
)

var InvalidOperation = errors.New("invalid operation")

type MathCommand struct {
	Op     string `long:"op" description:"operation to execute"`
	Args   []string
	Result int64
}

func (mc *MathCommand) Execute(args []string) error {
	op := mc.Op
	if op != "+" && op != "-" && op != "x" && op != "/" {
		return InvalidOperation
	}
	// make([]T,len[,cap])
	nums := make([]int64, 0, len(args))
	for _, arg := range args {
		num, err := strconv.ParseInt(arg, 10, 64)
		if err != nil {
			return err
		}
		nums = append(nums, num)
	}
	mc.Result = Calculate(nums, op)
	mc.Args = args
	return nil
}

func Calculate(nums []int64, op string) int64 {
	if len(nums) == 0 {
		return 0
	}
	result := nums[0]
	for i := 1; i < len(nums); i++ {
		switch op {
		case "+":
			result += nums[i]
		case "-":
			result -= nums[i]
		case "x":
			result *= nums[i]
		case "/":
			result /= nums[i]
		}
	}
	return result
}

type Option struct {
	Math MathCommand `command:"math"`
}

func main() {
	var opt Option
	_, err := flags.Parse(&opt)
	if err != nil {
		return
	}
	fmt.Printf("The result of %s is %d ", strings.Join(opt.Math.Args, opt.Math.Op), opt.Math.Result)
}
```

子命令必须实现`go-flags`定义的`Commander`接口：

```go
type Commander interface {
    Execute(args []string) error
}
```

解析命令行时，如果遇到不是以`-`或`--`开头的参数,`go-flags`会尝试将其解释为子命令。子命令的名字通过在结构标签中使用`command`指定。子命令后面的参数都将作为子命令的参数，子命令也可以有选项

```sh
$ go run ./main.go math --op + 1 2 3 4                                   
The result of 1+2+3+4 is 10
$  go run ./main.go math --op - 1 2 3 4
The result of 1-2-3-4 is -8
$ go run ./main.go math --op x 1 2 3 4
The result of 1x2x3x4 is 24
$ go run ./main.go math --op / 16 4 2
The result of 16/4/2 is 2
```

## 其他

`go-flags`还有其他有意思的特性，例如支持windows选项格式(`/v`,`/verbose`),从环境变量中读取默认值，从ini文件中读取默认设置等。

## 参考文档

1. [go-flags](https://github.com/jessevdk/go-flags) github repository
2. [go-flags](https://godoc.org/github.com/jessevdk/go-flags) lib godoc
3. [go-flags](https://darjun.github.io/2020/01/10/godailylib/go-flags/) darjun blog

