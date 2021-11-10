# Golang Daily lib -- go-homedir

## 简介

`go-homedir`顾名思义用来获取用户的主目录。实际上使用`os/user`也可以获取这个信息

```go
package main

import (
	"fmt"
	"log"
	"os/user"
)

func main() {
	u, err := user.Current()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%#v", u)
}

```

```
&user.User{Uid:"1000", Gid:"1000", Username:"username", Name:"username", HomeDir:"/home/username"}
```

那为什么需要`go-homedir`？

在Darwin系统上,标准库`os/user`的使用需要cgo。所以，任何使用`os/user`的代码都不能交叉编译。但是，大多数情况下，使用`os/user`的目的是为了获取主目录。因此，`go-homedir`出现了

## 快速使用 quick start

```
go get -u github.com/mitchellh/go-homedir v1.1.0
```

```go
package main

import (
	"fmt"
	"github.com/mitchellh/go-homedir"
	"log"
)

func main() {
	dir, err := homedir.Dir()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Home dir:", dir)

	dir = "~/dir"
	expandedDir, err := homedir.Expand(dir)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Expand of %s is :%s\n", dir, expandedDir)
}

```

`go-homedir`有两个功能：

- `Dir`：获取用户主目录
- `Expand`:将路径中的第一个`~`扩展成用户主目录

## 高级用法

由于`Dir`的调用可能涉及一些系统调用和外部执行命令，多次调用耗费性能。所以`go-homedir`提供了缓存功能，默认情况下，缓存是开启的，可以使用`DisableCache`设置`false`来关闭

```go
package main

import (
	"fmt"
	"github.com/mitchellh/go-homedir"
)

func main() {
	homedir.DisableCache = false

	dir, _ := homedir.Dir()
	fmt.Println("Home:", dir)
}

```

使用缓存时，如过程序中修改了主目录，再次调用`Dir`返回的是之前已经缓存的目录。若需获取新的主目录，可以先调用`Reset`清除缓存

## 实现

`go-homedir`源码只有一个文件homedir.go，大概看下`Dir`的实现，去掉缓存相关的Code

```go
func Dir() (string, error) {
  var result string
  var err error
  if runtime.GOOS == "windows" {
    result, err = dirWindows()
  } else {
    // Unix-like system, so just assume Unix
    result, err = dirUnix()
  }

  if err != nil {
    return "", err
  }
  return result, nil
}
```

判断当前系统是`windows`还是`unix-like`,分别调用不同的方法获取homedir

先看windows的

```go
func dirWindows() (string, error) {
	// First prefer the HOME environmental variable
	if home := os.Getenv("HOME"); home != "" {
		return home, nil
	}

	// Prefer standard environment variable USERPROFILE
	if home := os.Getenv("USERPROFILE"); home != "" {
		return home, nil
	}

	drive := os.Getenv("HOMEDRIVE")
	path := os.Getenv("HOMEPATH")
	home := drive + path
	if drive == "" || path == "" {
		return "", errors.New("HOMEDRIVE, HOMEPATH, or USERPROFILE are blank")
	}

	return home, nil
}
```

流程如下：

- 读取环境环境变量`HOME`,不为空则返回
- 读取环境变量`USERPROFILE`，不为空则返回
- 读取环境变量`HOMEDRIVE`,`HOMEPATH`,均不为空则拼接后返回

Unix-like

```go
func dirUnix() (string, error) {
	homeEnv := "HOME"
	if runtime.GOOS == "plan9" {
		// On plan9, env vars are lowercase.
		homeEnv = "home"
	}

	// First prefer the HOME environmental variable
	if home := os.Getenv(homeEnv); home != "" {
		return home, nil
	}

	var stdout bytes.Buffer

	// If that fails, try OS specific commands
	if runtime.GOOS == "darwin" {
		cmd := exec.Command("sh", "-c", `dscl -q . -read /Users/"$(whoami)" NFSHomeDirectory | sed 's/^[^ ]*: //'`)
		cmd.Stdout = &stdout
		if err := cmd.Run(); err == nil {
			result := strings.TrimSpace(stdout.String())
			if result != "" {
				return result, nil
			}
		}
	} else {
		cmd := exec.Command("getent", "passwd", strconv.Itoa(os.Getuid()))
		cmd.Stdout = &stdout
		if err := cmd.Run(); err != nil {
			// If the error is ErrNotFound, we ignore it. Otherwise, return it.
			if err != exec.ErrNotFound {
				return "", err
			}
		} else {
			if passwd := strings.TrimSpace(stdout.String()); passwd != "" {
				// username:password:uid:gid:gecos:home:shell
				passwdParts := strings.SplitN(passwd, ":", 7)
				if len(passwdParts) > 5 {
					return passwdParts[5], nil
				}
			}
		}
	}

	// If all else fails, try the shell
	stdout.Reset()
	cmd := exec.Command("sh", "-c", "cd && pwd")
	cmd.Stdout = &stdout
	if err := cmd.Run(); err != nil {
		return "", err
	}

	result := strings.TrimSpace(stdout.String())
	if result == "" {
		return "", errors.New("blank output when reading home directory")
	}

	return result, nil
}
```

流程如下：

- 读取环境变量`HOME`（`plan9`系统上为`home`）,不为空则返回
- 使用`getnet`命令查看系统的数据库中的相关记录，我们直到`passwd`文件中存储了用户信息，包括用户的主目录。使用`getnet`查看`passwd`中当前用户的记录，然后找到主目录返回
- 若上个步骤失败，`cd`不加参数可以直接切换到用户主目录，而`pwd`可显示当前目录，结合两者可以返回用户主目录

## 参考

1. [go-homedir](https://github.com/mitchellh/go-homedir) github repo 
2. [go-homedir](https://darjun.github.io/2020/01/14/godailylib/go-homedir/) darjun blog

