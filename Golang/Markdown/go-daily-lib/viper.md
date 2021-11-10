# Golang daily lib -- viper

## 简介

`viper`是一个配置解决方案，拥有丰富的特性：

- 支持 JOSN/TOML/YAML/HCL/envfile/properties 等多种格式配置文件
- 可以设置监听配置文件的修改，修改时自动加载新配置
- 从环境变量、命令行选项和`io.Reader`中读取配置
- 从远程配置系统中读取和监听修改，如 etcd/Consul
- 代码逻辑中显示设置建值

## 快速开始 Quick Start

在 go module 中引入：

```
go get -u github.com/spf13/viper latest
```

配置文件使用TOML格式(语法快速入门可以看[learn X in Y miniutes](https://learnxinyminutes.com/docs/toml/))，config.toml:

```toml
app_name="golang daily lib -- viper"

log_level="DEBUG"

[mysql]
ip="127.0.0.1"
port = 3306
user = "root"
password = 123456
database = "awesome"

[redis]
ip = "127.0.0.1"
port = 7381
```

```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
	"log"
)

func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("toml")
	viper.AddConfigPath("../resources")
	// 设置默认值
	viper.SetDefault("redis.port", 6379)

	err := viper.ReadInConfig()
	if err != nil {
		log.Fatal("read config failed: ", err)
	}

	// print config
	// base config
	fmt.Println("app name", viper.Get("app_name"))
	fmt.Println("log level", viper.Get("log_level"))

	// mysql config
	fmt.Println("mysql ip:", viper.Get("mysql.ip"))
	fmt.Println("mysql port:", viper.Get("mysql.port"))
	fmt.Println("mysql user:", viper.Get("mysql.user"))
	fmt.Println("mysql pass:", viper.Get("mysql.password"))
	fmt.Println("mysql database:", viper.Get("mysql.database"))

	// redis config
	fmt.Println("redis ip:", viper.Get("redis.ip"))
	fmt.Println("redis port:", viper.Get("redis.port"))
}

```

viper使用较为简单：

- SetConfigName: 设置文件名
- SetConfigType: 设置配置类型
- AddConfigPath: 设置配置文件路径
- ReadInConfig:  读取配置

```sh
$ go run ./main.go
app name golang daily lib -- viper
log level DEBUG
mysql ip: 127.0.0.1
mysql port: 3306
mysql user: root
mysql pass: 123456
mysql database: awesome
redis ip: 127.0.0.1
redis port: 7381
```

## 读取

viper 提供了多种形式的读取函数。上述的例子中，`Get`函数返回`interface{}`类型的值，但是使用不太方便

`GetType`系列函数可以返回指定类型的值。Type 可以是 `Bool/Float64/Int/String/Time/Duration/IntSlice/StringSlice`,需要注意*若指定Key不存在或类型不正确，`GetType`将返回类型的零值*

若要判断key是否存在，可以使用`IsSet`函数

使用`GetStringMap`和`GetStringMapString`将以map的形式返回Key下的所有键值对，前者返回`map[string]interface{}`,后者返回`map[string]string`

`AllSettings`以`map[string]interface{}`返回所有值

```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
	"log"
)

func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("toml")
	viper.AddConfigPath("../resources")

	if err := viper.ReadInConfig(); err != nil {
		log.Fatal("read config failed:", err)
	}

	fmt.Println("protocols:", viper.GetStringSlice("server.protocols"))
	fmt.Println("ports:", viper.GetIntSlice("server.ports"))
	fmt.Println("timeout:", viper.GetDuration("server.timeout"))

	fmt.Println("mysql ip:", viper.GetString("mysql.ip"))
	fmt.Println("mysql port", viper.GetInt("mysql.port"))

	if viper.IsSet("redis.port") {
		fmt.Println("redis port is set ")
	} else {
		fmt.Println("redis port is not set")
	}

	fmt.Println("mysql settings: ", viper.GetStringMap("redis"))
	fmt.Println("mysql settings: ", viper.GetStringMapString("mysql"))
	fmt.Println("all settings:", viper.AllSettings())
}

```

在config.toml中添加`protocols`和`ports`配置

```toml
[server]
protocols = ["https","http","ftp"]
ports = [10000,10001,10002]
timeout = "3s"
```

运行程序：

```sh
$ go run ./main.go 
protocols: [https http ftp]
ports: [10000 10001 10002]
timeout: 3s
mysql ip: 127.0.0.1
mysql port 3306
redis port is set 
mysql settings:  map[ip:127.0.0.1 port:7381]
mysql settings:  map[database:awesome ip:127.0.0.1 password:123456 port:3306 user:root]
all settings: map[app_name:golang daily lib -- viper log_level:DEBUG mysql:map[database:awesome ip:127.0.0.1 password:123456 port:3306 user:root] redis:map[ip:127.0.0.1 port:7381] server:map[ports:[10000 10001 10002] protocols:[https http ftp] timeout:3s]]

```

上例中`GetDuration`可以读取`Duration`类型配置，只要是`time.ParseDuration`接受的格式均可，例如：`3s`,`2min`,`1min30s`

## 设置配置

viper 支持在多个地方设置，使用以下顺序依次读取：

- 使用`Set`函数显式设置的
- 命令行选项
- 环境变量
- 配置文件
- 默认值

### `viper.Set`

若Key-Value通过`viper.Set`设置，那么此配置的优先级是最高的

```go
viper.Set("redis.port",5379)
```

将上述代码放入示例程序中，`viper.GetInt("redis.port")`将为5379(优先级在配置文件之前)

### 命令行选项

如果Key没有通过`viper.Set`显式设置,viper 将会从命令行选项中读取；命令行选项使用`pflag`来解析选项，首先在`init`方法中定义选项并调用`viper.BindPFlags`绑定选项到配置中：

```go
// set-kv/main.go
func init() {
	pflag.Int("redis.port", 6379, "redis port")

	// 绑定命令行
	viper.BindPFlags(pflag.CommandLine)
}

func main() {
	pflag.Parse()

	viper.SetConfigName("config")
	viper.SetConfigType("toml")
	viper.AddConfigPath("../resources")

	if err := viper.ReadInConfig(); err != nil {
		log.Fatal("read config failed :", err)
	}

	viper.Set("mysql.port", 3307)

	fmt.Println("Mysql port:", viper.GetInt("mysql.port"))
	fmt.Println("Redis port", viper.GetInt("redis.port"))
}
```

设置`redis.port`命令行参数：

```sh
$ go run ./main.go --redis.port 6667
Mysql port: 3307
Redis port 6667
```

不设置命令行参数,读取配置文件配置：

```sh
$ go run ./main.go                   
Mysql port: 3307
Redis port 7381
```

将配置文件的`redis.port`注释， 将会读取设置的默认值：

```sh
$ go run ./main.go 
Mysql port: 3307
Redis port 6379
```

### 环境变量

若从配置和命令行选项中未获取参数，则将从环境变量中获取；我们可以单个绑定，也可设置自动绑定

在`init`函数中使用`AutomaticEnv`绑定全部环境变量:

```go
func init(){
    // 绑定全部环境变量
    viper.AutomaticEnv()
}
```

```go
func main() {
    fmt.Println("HOME:",viper.GetString("HOME"))
}
```

输出HOME环境变量：

```sh
$ go run ./main.go 
HOME: /home/username
```

单独绑定环境变量`redis_port`:

```go
func init(){
    viper.BindEnv("redis.port","redis_port")
}
```

```go
func main(){
    fmt.Println("redis.port:",viper.GetInt("redis.port"))
}
```

设置环境变量`redis_port`(os:Manjaor shell:zsh,注意此处的环境变量名不可含有`.`,`=`两边不要有空格)：

```sh
$ export redis_port=6666;go run ./main.go  
HOME: /home/kesa
redis.port 6666
```

需要注意的是使用export设置环境变量只对当前process生效，在新的shell process中用export设置环境变量程序是无法读取到的

对于`BindEnv`函数：

- 只传入一个参数：此参数将同时作为键名和环境变量名
- 传入两个参数：  第一个为键名，第二个为环境变量名
- 传入两个以上参数： 将会按第二个参数开始的多个环境变量名顺序读取，直到读取到值为止

```go
func init(){   	           				viper.BindEnv("redis.port","redis_port_1","redis_port_2","redis_port3")
}
func main(){
    fmt.Println("redis.port:",viper.GetInt("redis.port"))
}
```

```shell
// run-main.sh
#!/bin/zsh
export redis_port_1=6000
export redis_port_2=6001
export redis_port_3=6002
go run ./main.go
```

```sh
$ ./run-main.sh
HOME: /home/kesa
redis.port 6000
```

程序读取到`redis_port_1`之后就不会继续读取后续的环境变量

还可以使用`viper.SetEnvPrefix`函数设置环境变量前缀，之后通过`AtuomaticEnv`和一个参数的`BindEnv`绑定的环境变量，在使用`Get*`是变回加上前缀进行查找，若设置的环境变量不存在，viper会转成全大写后再次查找，例如：使用`home`可以查到`HOME`的值

### 配置文件

在上述的步骤中未获取配置，则会从配置文件中的配置查找，关于配置文件的用法参见[quick-start](#快速开始 Quick Start)

###  默认值

关于默认值的配置，可参见[quick-start](#快速开始 Quick Start)

## 读取配置

### 从`io.Reader`中读取

viper 支持从`io.Reader`中读取配置。这种形式比较灵活，来源可以是文件，可以是程序中生成的字符串，或者是从网络连接中读取的字节流

从字符串中读取配置：

```go
package main

import (
	"bytes"
	"fmt"
	"github.com/spf13/viper"
	"log"
)

func main() {
	viper.SetConfigType("yaml")
	ymlConfig := []byte(`
app_name: read-config
log_level: debug
mysql:
 port: 3306
redis:
 port: 6379
`)
	err := viper.ReadConfig(bytes.NewBuffer(ymlConfig))
	if err != nil {
		log.Fatal("read config failed:", err)
	}
	fmt.Println("redis port:", viper.GetInt("redis.port"))
}

```

```sh
$ go run ./main.go
redis port: 6379
```

### Unmarshal

viper 支持将配置Unmarshal到结构体中 

创建yaml配置

```yaml
# /resources/config2.yml
app_name: unmarshal
log_level: debug
mysql:
  port: 3306
  ip: 127.0.0.1
redis:
  port: 6379
```

创建结构体并解析：

```go
type Config struct {
	AppName  string `mapstructure:"app_name"`
	LogLevel string `mapstructure:"log_level"`

	MySql MySqlConfig
	Redis RedisConfig
}

type MySqlConfig struct {
	Port int
	ip   string // will not be parsed
}

type RedisConfig struct {
	Port int
}

func main() {
    // viper.SetConfigName("config")
	viper.SetConfigName("config2")
	viper.SetConfigType("yaml")
	viper.AddConfigPath("../resources")
	if err := viper.ReadInConfig(); err != nil {
		log.Fatal("read config failed:", err)
	}

	var c Config
	viper.Unmarshal(&c)

	fmt.Printf("%#v", c)
}
```

```sh
$ go run ./main.go
main.Config{
	AppName:"unmarshal",
	LogLevel:"debug", 
	MySql:
		main.MySqlConfig{
			Port:3306, 
			ip:""
		},
        Redis:main.RedisConfig{
        	Port:6379
        }
} 
```

结构体的字段必须是导出的才会被赋值，示例中的`MysqlConfig.ip`由于不是导出的所以未赋值

unmarshal功能引用了mapstructure,详情参见项目[github.com/mitchellh/mapstructure](https://github.com/mitchellh/mapstructure) 

值得注意的是`viper.SetConfigName`仅设置配置文件名（忽略扩展名）,`viper.SetConfigType`设置的是配置文件内容格式（用于解析）

若存在同名的配置`config.toml`和`config.yml`,在上例的程序中设置配置文件名为`config`，解析类型为`yaml`，可能遇到解析报错，程序会将`config.toml`当做`yaml`文件解析，需要设置不同的文件名来解决

## 保存配置

viper提供了以下接口将配置保存下来：

- `WriteConfig`: 将当前的 viper 配置写入至预设路径，若未设置路径则报错；此方式将覆盖当前配置
- `SafeWriteConfig`:功能同上，但不覆盖已有配置
- `WriteConfigAs`:保存配置到指定路径，覆盖已有配置
- `SafeWriteConfig`:功能同上，不覆盖已有配置

下例通过程序生成配置文件：

```go
func main() {
	viper.SetConfigName("config3")
	viper.SetConfigType("json")
	viper.AddConfigPath("../resources")

	viper.Set("app_name", "save-config")
	viper.Set("mysql.port", 3306)
	viper.Set("redis.port", 6379)

	if err := viper.SafeWriteConfig(); err != nil {
		log.Fatal("write config fialed :", err)
	}
}
```

执行后生成`config3.json`

```json
{
  "app_name": "save-config",
  "mysql": {
    "port": 3306
  },
  "redis": {
    "port": 6379
  }
}
```

## 监听文件修改

viper 可以监听文件修改，实现配置热加载，无需重启服务即可使配置生效

```go
func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("toml")
	viper.AddConfigPath("../resources")

	err := viper.ReadInConfig()
	if err != nil {
		log.Fatal("read config failed: ", err)
	}

	viper.WatchConfig()

	fmt.Println("Before : redis.port=", viper.GetInt("redis.port"))
	time.Sleep(time.Second * 10)
	fmt.Println("After : redis.port=", viper.GetInt("redis.port"))
}
```

调用`viper.WatchConfig`函数，viper会自动监听配置修改

上例中，先打印redis.port，在Sleep期间修改配置文件，配置修改成功生效

```sh
$ go run ./main.go
Before : redis.port= 7381
After : redis.port= 6379
```

还可为配置修改增加回调函数：

```go
viper.OnConfigChange(func(e fsnotify.Event) {
  fmt.Printf("Config file:%s Op:%s\n", e.Name, e.Op)
})
```

在文件修改时会执行此回调函数,viper 使用[fsnotify](https://github.com/fsnotify/fsnotify)库来实现监听文件修改的功能

## 参考

1. [viper](https://github.com/spf13/viper) GitHub-repo
2. [viper](https://darjun.github.io/2020/01/18/godailylib/viper/) darjun/blog
