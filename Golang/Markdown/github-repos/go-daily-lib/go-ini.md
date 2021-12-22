# Golang daily lib -- go-ini

## 简介

ini是windows上常用的配置文件格式。MySQL的Windows版本就是使用ini格式存储配置的。`go-ini`是Golang中用于操作ini文件的三方库

### ini文件简介

.ini 文件是Initialization File的缩写，即初始化文件 ，是windows的系统配置文件所采用的存储格式

#### 文件格式

ini文件由节、键、值组成,注释使用`;`

```ini
; commnet line
[section]
key1=value1
key2=value2
```

## 快速开始 quick start

```
go get -u gopkg.in/ini.v1 v1.63.2
```



```ini
; config.ini
app_name = awesome web

; possible values: DEBUG, INFO, WARNING, ERROR, FATAL
log_level = DEBUG

[mysql]
ip = 127.0.0.1
port = 3306
user = u
password = 123456
database = awesome

[redis]
ip = 127.0.0.1
port = 6381

```



```go
package main

import (
	"fmt"
	"gopkg.in/ini.v1"
	"log"
)

func main() {
	config, err := ini.Load("../resources/config.ini")
	if err != nil {
		log.Fatal("read config error ", err)
	}
	fmt.Println("App name:", config.Section("").Key("app_name").String())
	fmt.Println("Log level:", config.Section("").Key("log_level").String())
	fmt.Println("Mysql ip:", config.Section("mysql").Key("ip").String())
	mysqlPort, err := config.Section("mysql").Key("port").Int()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(mysqlPort)
	fmt.Println("Redis ip:", config.Section("redis").Key("ip").String())
	redisPort, err := config.Section("redis").Key("port").Int()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(redisPort)
}

```



```sh
$ go run ./main.go 
App name: awesome web
Log level: DEBUG
Mysql ip: 127.0.0.1
3306
Redis ip: 127.0.0.1
6381
```

在ini文件中，每个键值对占用一行，中间使用`=`分隔。`;`开头为注释，文件内容以`section`组织，`section`以`[section_name]`开始，下一个`section`结束。所有的section前的内容属于默认section

使用`go-ini`读取配置文件步骤如下：

- 首先调用`init.load`加载文件，获取配置对象`config`(类型`ini/File`)
- 之后调用`Section`方法获取section，默认section名为"",也可使用`ini.DefaultSection`
- 调用`Key`方法获取key(类型`ini/Key`)对象
- 根据value的类型调用不同的方法获取值，如`String`,`Int` 等

```
config.Section("section_name").Key("key_name").[Value(),String(),Int()...]
```

配置文件中存储的均为字符串，故`String()`仅返回值；若类型为`int/uint/float64`时，转换可能失败，故`Int()/Uint()/Float64()`返回值和error

## MustType 便捷方法

若每次取值需要进行错误判断，那么Code将会变得繁琐。为此，`go-ini`也提供对应的`MustType`方法，这种方法仅返回一个值。同时可接收变长参数，若无法进行类型转换，取参数列表中的第一个返回

```ini
; config_2.ini
[redis]
port=port_6379
```



```go
package main

import (
	"fmt"
	"gopkg.in/ini.v1"
)

func main() {
	config, err := ini.Load("../resources/config_2.ini")
	if err != nil {
		fmt.Println("read config error :", err)
	}
	fmt.Println("Before call MustType")
	port, err := config.Section("redis").Key("port").Int()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("port:", port)
	fmt.Println("Call MustType")
	port = config.Section("redis").Key("port").MustInt(6379)
	fmt.Println("port:", port)
	port, err = config.Section("redis").Key("port").Int()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("After call MustType")
	fmt.Println("port:", port)
}

```



```sh
$ go run ./main.go 
Before call MustType
strconv.ParseInt: parsing "port_6379": invalid syntax
port: 0
Call MustType
port: 6379
After call MustType
port: 6379
```

第一次调用`Int`方法返回错误，在调用`MustInt`将参数设置为6379之后，再次调用`Int`返回6379

```go
// gopkg.in/ini.v1/key.go
func (k *Key) MustInt(defaultVal ...int) int {
  val, err := k.Int()
  if len(defaultVal) > 0 && err != nil {
    k.value = strconv.FormatInt(int64(defaultVal[0]), 10)
    return defaultVal[0]
  }
  return val
}
```

## Section 操作

### 获取信息

在加载配置之后，可以通过`Sections`获取所有sectionm,`SectionStrings`方法获取所有section name.

```go
package main

import (
	"fmt"
	"gopkg.in/ini.v1"
	"log"
)

func main() {
	config, err := ini.Load("../resources/config.ini")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Sections:%#v\n\n", config.Sections())
	fmt.Printf("Section names:%v\n\n", config.SectionStrings())

	newSection := config.Section("new_section")
	fmt.Printf("New section:%#v\n\n", newSection)
	fmt.Printf("Section names:%v\n\n", config.SectionStrings())
}

```



```sh
$ go run ./main.go
Sections:[]*ini.Section{(*ini.Section)(0xc00011a620), (*ini.Section)(0xc00011a770), (*ini.Section)(0xc00011aa10)}

Section names:[DEFAULT mysql redis]

New section:&ini.Section{f:(*ini.File)(0xc00012c000), Comment:"", name:"new_section", keys:map[string]*ini.Key{}, keyList:[]string{}, keysHash:map[string]string{}, isRawSection:false, rawBody:""}

Section names:[DEFAULT mysql redis new_section]
```

调用`Section(name)`获取以name为名的section，若此section不存在，则自动创建一个section返回.之后调用`SectionStrings`新的section也会被返回

也可以手动创建一个新的分区

```go
err := cfg.NewSection("new")
```

### 子分区

在配置文件中，可以使用占位符`%(name)s`表示用之前已定义的Key`name`的值来替换，`s`表示值为字符串占用类型

```ini
NAME = ini 
VERSION = v1
IMPORT_PATH = gopkg.in/%(NAME)s.%(VERSION)s

[package]
CLONE_URL = https://%(IMPORT_PATH)s

[package.sub]
```

上述文件设置`IMPORT_PATH`时使用前面定义的`NAME`和`VERSION`。在`package`中设置`CLONE_URL`时，使用了默认section中定义的`IMPORT_PATH`

可以在section名中使用`.`来表示子section。若在`package.sub`中未找到所需的key就会到`package`中寻找

```go
package main

import (
	"fmt"
	"gopkg.in/ini.v1"
	"log"
)

func main() {
	config, err := ini.Load("../resources/sub_section.ini")
	if err != nil {
		log.Fatal("Read config error:", err)
	}
	fmt.Println("CLONE_URL:", config.Section("package.sub").Key("CLONE_URL").String())
}

```

```sh
$ go run ./main.go 
CLONE_URL: https://gopkg.in/ini.v1
```

在`package.sub`中没有`CLONE_URL`，返回`package`中的值

## 保存配置

有时需要将配置写入到文件中。写入配置有两种方式：

- `SaveTo`:写入到文件中
- `WriteTo`:写入到`io.Writer`中

```go
err = cfg.SaveTo("my.ini")
err = cfg.SaveToIndent("my.ini","\t")

cfg.WriteTo(writer)
cfg.WriteToIndent(writer,"\t")
```

```go
package main

import (
	"fmt"
	"gopkg.in/ini.v1"
	"os"
)

func main() {
	config := ini.Empty()

	defaultSection := config.Section("")
	defaultSection.NewKey("app_name", "save config")
	defaultSection.NewKey("log_level", "DEBUG")

	mysqlSec, err := config.NewSection("mysql")
	if err != nil {
		fmt.Println("create mysql section error:", err)
	}
	mysqlSec.NewKey("ip", "127.0.0.1")
	mysqlSec.NewKey("port", "3306")

	redisSec, err := config.NewSection("reids")
	if err != nil {
		fmt.Println("create redis section error :", err)
	}
	redisSec.NewKey("ip", "127.0.0.1")
	redisSec.NewKey("port", "6379")

	err = config.SaveTo("../resources/saved-config.ini")
	if err != nil {
		fmt.Println("save config error :", err)
	}

	err = config.SaveToIndent("../resources/saved-config-pretty.ini", " ")
	if err != nil {
		fmt.Println("save config error:", err)
	}

	config.WriteTo(os.Stdout)
	fmt.Println()
	config.WriteToIndent(os.Stdout, " ")

}

```

saved-config.ini:

```ini
app_name  = save config
log_level = DEBUG

[mysql]
ip   = 127.0.0.1
port = 3306

[reids]
ip   = 127.0.0.1
port = 6379
```

saved-config-pretty.ini:

```ini
app_name  = save config
log_level = DEBUG

[mysql]
 ip   = 127.0.0.1
 port = 3306

[reids]
 ip   = 127.0.0.1
 port = 6379
```

`[SaveTo/WriteTo]Indent`方法会对Section下的key-Value添加缩进

## Section和struct映射

定义结构变量，加载完配置文件后，调用`MapTo`将配置项赋值到结构变量的字段中

```go
package main

import (
	"fmt"
	"gopkg.in/ini.v1"
	"log"
)

type Config struct {
	AppName  string `ini:"app_name"`
	LogLevel string `ini:"log_level"`

	MySql MysqlConfig `ini:"mysql"'`
	Redis RedisConfig `ini:"redis"`
}

type MysqlConfig struct {
	IP       string `ini:"ip"`
	Port     int    `ini:"port"`
	User     string `ini:"user"`
	Password string `ini:"password"`
	Database string `ini:"database"`
}

type RedisConfig struct {
	IP   string `ini:"ip"`
	Port int    `ini:"port"`
}

func main() {
	config, err := ini.Load("../resources/config.ini")
	if err != nil {
		log.Fatal("load config error:", err)
	}
	c := Config{}
	config.MapTo(&c)
	fmt.Printf("%#v", c)
}

```

`MapTo`内部使用了反射，所以结构字段必须是导出的。若字段名和键名不同，则需要在结构标签中指定对应的键名。

可以使用`ini.MapTo(&c,"my.ini")`将加载和映射合并

```go
err := ini.MapTo(&c,"my.ini")
```

也可以仅映射一个分区：

```go
mysqlConfig := MysqlConfig{}
err := config.Section("mysql").MapTo(&mysqlConfig)
```

还可以通过结构体生成配置：

```go
	config2 := ini.Empty()
	c1 := Config{
		AppName:  "map_to_struct",
		LogLevel: "DEBUG",
		MySql: MysqlConfig{
			IP:   "127.0.0.1",
			Port: 3306,
			User: "root",
		},
		Redis: RedisConfig{
			IP: "127.0.0.1",
		},
	}

	err = ini.ReflectFrom(config2, &c1)
	if err != nil {
		log.Fatal("load from struct error")
	}
	err = config2.SaveTo("../resources/load_from_Struct.ini")
	if err != nil {
		log.Fatal("save config error", err)
	}
```

## 其他

`go-ini`还有其他的高级特性，可以参考[官方文档](https://ini.unknwon.io/)

## 参考

1. [go-ini](https://github.com/go-ini/ini) gitHub repo
2. [go-ini](https://ini.unknwon.io/) godoc
3. [go-ini](https://darjun.github.io/2020/01/15/godailylib/go-ini/) darjun blog

