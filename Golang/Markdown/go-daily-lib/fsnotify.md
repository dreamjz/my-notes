#  Golang daily lib -- fsnotify

## Introduction 简介

在上一篇介绍[viper](./viper)的文章中，viper内部使用了`fsnotify`库实现监听文件修改并自动重新加载

`fsnotify`是一个基于Go的跨平台文件系统通知库（Cross-platform file system notifications for Go）

## Quick start 快速开始

在go module中导入：

```
go get -u github.com/fsnotify/fsnotify latest
```

```go
package main

import (
	"github.com/fsnotify/fsnotify"
	"log"
)

func main() {
	watcher, err := fsnotify.NewWatcher()
	if err != nil {
		log.Fatal("create new watcher failed:", err)
	}
	defer watcher.Close()

	done := make(chan bool)
	go func() {
		for {
			select {
			case event, ok := <-watcher.Events:
				if !ok {
					return
				}
				log.Printf("%s : %s\n", event.Name, event.Op)
			case err, ok := <-watcher.Errors:
				if !ok {
					return
				}
				log.Println("error:", err)
			}
		}
	}()

	err = watcher.Add("../resources")
	if err != nil {
		log.Fatal("add watch path failed:", err)
	}
	<-done
}
```

启动程序后在`../resources`目录下执行：

```sh
touch test_fs.txt
echo 'test' >> test_fs.txt
mv test_fs.txt fs.txt
rm fs.txt
```

程序输出：

```
go run ./main.go
2021/10/26 10:21:11 ../resources/test_fs.txt : CREATE
2021/10/26 10:21:11 ../resources/test_fs.txt : CHMOD
2021/10/26 10:21:11 ../resources/test_fs.txt : WRITE
2021/10/26 10:21:11 ../resources/test_fs.txt : RENAME
2021/10/26 10:21:11 ../resources/fs.txt : CREATE
2021/10/26 10:21:11 ../resources/fs.txt : REMOVE
```

值得注意的是：

- 重命名触发了两个事件：原文件的`RENAME`和新文件的`CREATE`
- `touch`创建文件触发了两个事件:`CREATE`和`CHMOD`

`fsnotify`的使用比较简单：

- 调用`NewWatcher`创建监听器
- 调用监听器的`Add`方法添加监听路径
- 若目录/文件由事件产生，从监听器的`Events`通道取出事件；出现错误，从`Errors`通道取出错误
- 由于`fsnotify`使用了操作系统接口，监听器中保存了系统资源的句柄，使用之后需要关闭

## Event 事件

上文示例中出现的事件是`fsnotify.Event`类型：

```go
// fsnotify.go
// Event represents a single file system notification.
type Event struct {
	Name string // Relative path to the file or directory.
	Op   Op     // File operation that triggered the event.
}
```

其中，`Name`为相关的文件/目录，`Op`为触发事件的文件操作

`Op`共有五种取值：

```go
// fsnotify.go
// Op describes a set of file operations.
type Op uint32

// These are the generalized file operations that can trigger a notification.
const (
	Create Op = 1 << iota
	Write
	Remove
	Rename
	Chmod
)
```

可以看到`Op`按照位存储的，可以通过`&`来判断事件:

```go
if event.Op & fsnotify.Write == fsnotify.Write{
    fmt.Printlm("Op is write ")
}
```

但是使用时无需判断，在`Op`的`String`方法中已经处理:

```go
func (op Op) String() string {
	// Use a buffer for efficient string concatenation
	var buffer bytes.Buffer

	if op&Create == Create {
		buffer.WriteString("|CREATE")
	}
	if op&Remove == Remove {
		buffer.WriteString("|REMOVE")
	}
	if op&Write == Write {
		buffer.WriteString("|WRITE")
	}
	if op&Rename == Rename {
		buffer.WriteString("|RENAME")
	}
	if op&Chmod == Chmod {
		buffer.WriteString("|CHMOD")
	}
	if buffer.Len() == 0 {
		return ""
	}
	return buffer.String()[1:] // Strip leading pipe
}
```

## Application 应用

`fsnotify`的应用比较广泛，在[godoc](https://pkg.go.dev/github.com/fsnotify/fsnotify@v1.5.1)页面上可以看到其被超过3000个项目引用

这里看下`viper.WatchConfig`是是如何使用`fsnotify` 的:

```go
// viper.go
func WatchConfig() { v.WatchConfig() }

func (v *Viper) WatchConfig() {
	initWG := sync.WaitGroup{}
	initWG.Add(1)
	go func() {
		watcher, err := newWatcher()
		if err != nil {
			log.Fatal(err)
		}
		defer watcher.Close()
		// we have to watch the entire directory to pick up renames/atomic saves in a cross-platform way
		filename, err := v.getConfigFile()
		if err != nil {
			log.Printf("error: %v\n", err)
			initWG.Done()
			return
		}

		configFile := filepath.Clean(filename)
		configDir, _ := filepath.Split(configFile)
		realConfigFile, _ := filepath.EvalSymlinks(filename)

		eventsWG := sync.WaitGroup{}
		eventsWG.Add(1)
		go func() {
			for {
				select {
				case event, ok := <-watcher.Events:
					if !ok { // 'Events' channel is closed
						eventsWG.Done()
						return
					}
					currentConfigFile, _ := filepath.EvalSymlinks(filename)
					// we only care about the config file with the following cases:
					// 1 - if the config file was modified or created
					// 2 - if the real path to the config file changed (eg: k8s ConfigMap replacement)
					const writeOrCreateMask = fsnotify.Write | fsnotify.Create
					if (filepath.Clean(event.Name) == configFile &&
						event.Op&writeOrCreateMask != 0) ||
						(currentConfigFile != "" && currentConfigFile != realConfigFile) {
						realConfigFile = currentConfigFile
						err := v.ReadInConfig()
						if err != nil {
							log.Printf("error reading config file: %v\n", err)
						}
						if v.onConfigChange != nil {
							v.onConfigChange(event)
						}
					} else if filepath.Clean(event.Name) == configFile &&
						event.Op&fsnotify.Remove&fsnotify.Remove != 0 {
						eventsWG.Done()
						return
					}

				case err, ok := <-watcher.Errors:
					if ok { // 'Errors' channel is not closed
						log.Printf("watcher error: %v\n", err)
					}
					eventsWG.Done()
					return
				}
			}
		}()
		watcher.Add(configDir)
		initWG.Done()   // done initializing the watch in this go routine, so the parent routine can move on...
		eventsWG.Wait() // now, wait for event loop to end in this go-routine...
	}()
	initWG.Wait() // make sure that the go routine above fully ended before returning
}
```

使用流程类似：

- 使用`NewWatcher`创建监听器
- `v.getConfigFile`方法获取配置文件路径，并提取文件名，目录，链接指向路径（若配置文件是一个link）
- 使用`watcher.Add`监听配置目录，并启用新的goroutine处理事件

`WatchConfig`不能阻塞主goroutine,故创建监听器是在新的goroutine中进行；方法中由两个`sync.WaitGroup`变量，`initWG`保证监听器的初始化,`eventsWG`在events通道关闭，配置删除或遇到错误时退出循环

之后就是核心代码：

- 事件触发后，判断是否是关于配置文件的事件且仅处理`Create`和`Write`事件
- 调用`v.ReadInConfig`加载配置(配置文件的link发生改变也会触发重新加载)
- 若注册事件回调函数，则以事件为参数调用回调参数

## Conclusion 总结

`fsnotify`的接口非常简单直接，所有系统相关的复杂性都被封装起来了。这也是我们平时设计模块和接口时可以参考的案例

## Reference 参考

1. [fsnotify](https://pkg.go.dev/github.com/fsnotify/fsnotify@v1.5.1) godocs
2. [fsnotify](https://github.com/fsnotify/fsnotify) github-repo
3. [fsnotify](https://darjun.github.io/2020/01/19/godailylib/fsnotify/) darjun/blog
