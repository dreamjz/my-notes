# Golang daily lib -- cobra

## 简介

cobra是一个命令行程序库，可以用来编写命令行程序。同时也提供了一个脚手架，用于生成基于cobra的应用程序框架。非常多知名的开源项目使用了cobra库构建命令行，如Kubernetes,hugo,etcd等。

关于作者[spf13](http://github.com/spf13)，这里多说两句。spf13 开源不少项目，而且他的开源项目质量都比较高。 相信使用过 vim 的都知道[spf13-vim](https://github.com/spf13/spf13-vim)，号称 vim 终极配置。 可以一键配置，对于我这样的懒人来说绝对是福音。他的[viper](https://github.com/spf13/viper)是一个完整的配置解决方案。 完美支持 JSON/TOML/YAML/HCL/envfile/Java properties 配置文件等格式，还有一些比较实用的特性，如配置热更新、多查找目录、配置保存等。 还有非常火的静态网站生成器[hugo](https://gohugo.io/)也是他的作品。

## 快速开始 quick start

```
go get -u github.com/spf13/cobra v1.2.1
```

下面的例子将模拟`git version`命令，输出的结果通过调用`os/exec`调用外部的`git version`

```go
// main.go
package main

import "quick-start/cmd"

func main() {
	cmd.Execute()
}

```

```go
// cmd/root.go
package cmd

import (
	"errors"

	"github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
	Use:   "sim-git",
	Short: "Sim-git is a simulation of git",
	Long: `Sim-git is a simulation of git.
Git is a free and open source distributed version control system-designed to 
handle everything from small to very large projects with speed and efficiency.`,
	Run: func(cmd *cobra.Command, args []string) {
		Error(cmd, args, errors.New("unrecognized command"))
	},
}

func Execute() {
	rootCmd.Execute()
}

```

```go
// cmd/version.go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
	"os"
)

var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "version subcommand show git version info",
	Run: func(cmd *cobra.Command, args []string) {
		output, err := ExecuteCommand("git", "version", args...)
		if err != nil {
			Error(cmd, args, err)
		}
		fmt.Fprintf(os.Stdout, output)
	},
}

func init() {
	rootCmd.AddCommand(versionCmd)
}

```

```go
// cmd/helper.go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
	"os"
	"os/exec"
)

func ExecuteCommand(name string, subName string, args ...string) (string, error) {
	args = append([]string{subName}, args...)
	cmd := exec.Command(name, args...)
	bytes, err := cmd.CombinedOutput()
	return string(bytes), err
}

func Error(cmd *cobra.Command, args []string, err error) {
	fmt.Fprintf(os.Stderr, "execute %s args:%v error:%v\n", cmd.Name(), args, err)
	os.Exit(1)
}

```

每个cobra程序都有一个根命令，可以给其添加任意多个子命令。我们在`version.go`的init函数中讲子命令添加到根命令中。

cobra将自动生成帮助信息：

```sh
go run ./main.go -h
Sim-git is a simulation of git.
                        Git is a free and open source distributed version control system-designed to 
                        handle everything from small to very large projects with speed and efficiency.

Usage:
  sim-git [flags]
  sim-git [command]

Available Commands:
  completion  generate the autocompletion script for the specified shell
  help        Help about any command
  version     version subcommand show git version info

Flags:
  -h, --help   help for sim-git

Use "sim-git [command] --help" for more information about a command.
quick-start|main⚡ ⇒ go run ./main.go version -h
version subcommand show git version info

Usage:
  sim-git version [flags]

Flags:
  -h, --help   help for version

```

显式单个子命令的帮助信息：

```sh
go run ./main.go version -h
version subcommand show git version info

Usage:
  sim-git version [flags]

Flags:
  -h, --help   help for version

```

调用子命令：

```sh
go run ./main.go version   
git version 2.33.0

go run ./main.go clone  
Error: unknown command "clone" for "sim-git"
Run 'sim-git --help' for usage.

```

使用cobra构建命令行项目时，推荐使用以下文件结构：

```
.
├── cmd
│   ├── helper.go
│   ├── root.go
│   └── version.go
├── go.mod
├── go.sum
└── main.go
```

## 特性 Feature

cobra 提供非常丰富的功能：

- 轻松支持子命令，如： app server, app fetch 等
- 完全兼容 POSItifyX[^1] 选项（包括短、长选项s）
- 嵌套子命令
- 全局、本地层级选项。可以在多处设置选项，按照一定的顺序取用
- 使用脚手架轻松生成程序框架和命令

首先明确三个概念：

1. 命令（Command）： 需要执行的操作
2. 参数（Arg）:      命令的参数
3. 选项（Flag）：     命令的选项可以调整命令的行为

例如，server 是一（子）命令，—-port 是选项

```sh
hugo server --port=1313
```

clone 是（子）命令，URL是参数，—-brae  是选项

```sh
git clone URL --bare
```

## 命令 Command

在cobra中，命令和子命令都是用`Command`结构表示的。`Command`有非常多的字段，用来定制命令的行为。常用的有`Use/Short/Long/Run`

`Use`指定使用信息，即命令如何被使用，格式为`command arg1 [arg2 ... ]`

`Short/Long`指定命令的帮助信息

`Run`为实际执行操作的函数

定义新的子命令只需创建`cobra.Command`变量，设置相关字段，添加至根命令即可。例如添加status命令：

```go
package cmd

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"
)

var statusCmd = &cobra.Command{
	Use:   "status",
	Short: "show status of the git repository",
	Run: func(cmd *cobra.Command, args []string) {
		output, err := ExecuteCommand("git", "status", args...)
		if err != nil {
			Error(cmd, args, err)
		}
		fmt.Fprint(os.Stdout, output)
	},
}

func init() {
	rootCmd.AddCommand(statusCmd)
}

```

```sh
go run ./main.go status
位于分支 main
您的分支与上游分支 'origin/main' 一致。

要提交的变更：
  （使用 "git restore --staged <文件>..." 以取消暂存）
        新文件：   cmd/status.go

尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
        修改：     cmd/root.go
        修改：     cmd/status.go


```

## 选项 Flag

cobra 中选项分为*持久(persistent)选项*，定义它的命令及其子命令均可使用，通过给根命令添加一个选项定义全局选项。另一种是*本地选项*，仅能在定义它的命令中使用

cobra 使用pflag解析命令行选项.pflag 使用和`flag`包类似，存储flag的命令需提前声明：

```go
var Verbose bool 
var Source string
```

设置持久(persistent)选项：

```go
rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```

设置本地(local)选项

```go
localCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
```

### 简单计算器

下面的例子实现了一个简单计算器，支持加减乘除的功能，可以设置是否忽略非数字参数，除数为0是否报错。显然前一个应作为全局选项，后一个作为局部选项

```go
// cmd/root.go
package cmd

import (
	"errors"

	"github.com/spf13/cobra"
)

type ErrorHandling int

const (
	ContinueOnParseError ErrorHandling = iota
	ExitOnParseError
	PanicOnParseError
	ReturnOnDividedByZero
	PanicOnDividedByZero
)

type Operation int

const (
	Add Operation = iota
	Subtract
	Multiply
	Divide
)

var UnrecognizedCommand = errors.New("unrecognized command")

var parseHandling int

var rootCmd = &cobra.Command{
	Use:   "Calculator",
	Short: "Simple calculator in cobra",
	Run: func(cmd *cobra.Command, args []string) {
		Error(cmd, args, UnrecognizedCommand)
	},
}

func init() {
	rootCmd.PersistentFlags().IntVarP(&parseHandling, "parse-error", "p", int(ContinueOnParseError), "define how command behaves if the parse fails")
}

func Execute() {
	rootCmd.Execute()
}

```

```go
// cmd/divide.go
package cmd

import (
	"fmt"
	"strings"

	"github.com/spf13/cobra"
)

var divideByZeroHandling int

var divideCmd = &cobra.Command{
	Use:   "divide",
	Short: "do division",
	Run: func(cmd *cobra.Command, args []string) {
		values := ConvertArgsToFloat64Slice(args, ErrorHandling(parseHandling))
		result := calculate(values, Divide)
		fmt.Printf("%s = %.2f\n", strings.Join(args, "/"), result)
	},
}

func init() {
	divideCmd.Flags().IntVarP(&divideByZeroHandling, "divided-by-zero", "d", int(PanicOnDividedByZero), "define divide command behaves if divided by zero")
	rootCmd.AddCommand(divideCmd)
}

```

```go
// cmd/helper.go
package cmd

import (
	"errors"
	"fmt"
	"os"
	"strconv"

	"github.com/spf13/cobra"
)

var DividedByZero = errors.New("divided by zero")

func Error(cmd *cobra.Command, args []string, err error) {
	fmt.Fprintf(os.Stderr, "execute %s args:%v error:%s", cmd.Name(), args, err)
	os.Exit(1)
}

func ConvertArgsToFloat64Slice(args []string, errorHandling ErrorHandling) []float64 {
	result := make([]float64, 0, len(args))
	for _, arg := range args {
		val, err := strconv.ParseFloat(arg, 64)
		if err != nil {
			switch errorHandling {
			case ExitOnParseError:
				fmt.Fprintf(os.Stderr, "invalid number: %s \n", arg)
				os.Exit(1)
			case PanicOnParseError:
				panic(err)
			}
		}
		result = append(result, val)
	}
	return result
}

func calculate(values []float64, operation Operation) float64 {
	var result float64
	if len(values) == 0 {
		return result
	}
	result = values[0]
	for i := 1; i < len(values); i++ {
		switch operation {
		case Add:
			result += values[i]
		case Subtract:
			result -= values[i]
		case Multiply:
			result *= values[i]
		case Divide:
			if values[i] == 0 {
				switch ErrorHandling(divideByZeroHandling) {
				case ReturnOnDividedByZero:
					return result
				case PanicOnDividedByZero:
					panic(DividedByZero)
				}
			}
			result /= values[i]
		}
	}
	return result
}

```

## 脚手架 Scaffold

cobra程序得到项目框架比较固定，可以使用脚手架工具生成

使用`cobra init`创建一个cobra应用程序:

```sh
$ cobra init scaffold 
```

```
scaffold
├── cmd
│   └── root.go
├── LICENSE
└── main.go
```

在`root.go`中，工具额外生成了一些代码

在根命令中添加了配置文件选项

```go
func init() {
  cobra.OnInitialize(initConfig)

  rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.scaffold.yaml)")
  rootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}
```

在初始化完成的回调中，如果发现选项为空，则默认使用目录下的`.scaffold.yaml`

```go
func initConfig() {
  if cfgFile != "" {
    viper.SetConfigFile(cfgFile)
  } else {
    home, err := homedir.Dir()
    if err != nil {
      fmt.Println(err)
      os.Exit(1)
    }

    viper.AddConfigPath(home)
    viper.SetConfigName(".scaffold")
  }

  viper.AutomaticEnv()

  if err := viper.ReadInConfig(); err == nil {
    fmt.Println("Using config file:", viper.ConfigFileUsed())
  }
}
```

使用`cobra add command`添加命令

```sh
$ cobra add date
```

修改生成的date.go

```go
// cmd/date.go
/*
Copyright © 2021 NAME HERE <EMAIL ADDRESS>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
package cmd

import (
	"fmt"
	"strings"
	"time"

	"github.com/spf13/cobra"
)

var (
	year  int
	month int
)

// dateCmd represents the date command
var dateCmd = &cobra.Command{
	Use:   "date",
	Short: "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	Run: func(cmd *cobra.Command, args []string) {
		//if year < 1000 || year > 9999 {
		//	fmt.Fprintf(os.Stderr, "invalid year, should in [1000,9999], actual:%d\n", year)
		//	os.Exit(1)
		//}
		//if month < 1 || month > 12 {
		//	fmt.Fprintf(os.Stderr, "invalid month, should in [1,12], actual:%d\n", month)
		//	os.Exit(1)
		//}
		showCalendar()
	},
}

func showCalendar() {
	now := time.Now()

	showYear := year
	if showYear == 0 {
		// 默认使用今年
		showYear = int(now.Year())
	}
	showMonth := time.Month(month)
	if showMonth == 0 {
		showMonth = now.Month()
	}

	showTime := time.Date(showYear, showMonth, 1, 0, 0, 0, 0, now.Location())
	weekdays := []string{"Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"}
	for _, weekday := range weekdays {
		fmt.Printf("%5s", weekday)
	}
	fmt.Println()
	for {
		startWd := showTime.Weekday()
		fmt.Printf("%s", strings.Repeat(" ", int(startWd)*5))

		for ; startWd <= time.Saturday; startWd++ {
			fmt.Printf("%5d", showTime.Day())
			showTime = showTime.Add(time.Hour * 24)
			if showTime.Month() != showMonth {
				return
			}
		}
		fmt.Println()
	}

}

func init() {
	rootCmd.AddCommand(dateCmd)

	dateCmd.PersistentFlags().IntVarP(&year, "year", "y", 0, "year to show should in [1000,9999]")
	dateCmd.PersistentFlags().IntVarP(&month, "month", "m", 0, "month to show should in [1,12]")
	// Here you will define your flags and configuration settings.

	// Cobra supports Persistent Flags which will work for this command
	// and all subcommands, e.g.:
	// dateCmd.PersistentFlags().String("foo", "", "A help for foo")

	// Cobra supports local flags which will only run when this command
	// is called directly, e.g.:
	// dateCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}

```

```sh
$ go run ./main.go date      
  Sun  Mon  Tue  Wed  Thu  Fri  Sat
                             1    2
    3    4    5    6    7    8    9
   10   11   12   13   14   15   16
   17   18   19   20   21   22   23
   24   25   26   27   28   29   30
   31                      
```

## 其他

cobra 提供了非常丰富的特性和定制化接口，例如：

- 设置钩子函数，在命令执行前、后执行某些操作；
- 生成 Markdown/ReStructed Text/Man Page 格式的文档
- 等等

## 参考

1. [cobra](https://github.com/spf13/cobra) gitHub repo
2. [cobra](https://darjun.github.io/2020/01/17/godailylib/cobra/) darjun blog



[^1]: [**可移植操作系统接口**](https://zh.wikipedia.org/wiki/%E5%8F%AF%E7%A7%BB%E6%A4%8D%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%8E%A5%E5%8F%A3)（英语：Portable Operating System Interface，缩写为**POSIX**）是[IEEE](https://zh.wikipedia.org/wiki/IEEE)为要在各种[UNIX](https://zh.wikipedia.org/wiki/UNIX)[操作系统](https://zh.wikipedia.org/wiki/操作系统)上运行软件，而定义[API](https://zh.wikipedia.org/wiki/API)的一系列互相关联的标准的总称，其正式称呼为IEEE Std 1003，而国际标准名称为[ISO](https://zh.wikipedia.org/wiki/ISO)/[IEC](https://zh.wikipedia.org/wiki/IEC) 9945。



































