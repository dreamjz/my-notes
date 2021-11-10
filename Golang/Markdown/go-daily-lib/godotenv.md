# Golang daily lib —- godotenv

## 简介

[twelve-factor](https://12factor.net/)应用提倡将配置存储在环境变量中，任何从开发环境切换到生产环境时需要修改的东西从代码抽取到环境变量中。但是在实际开发中，如果同一台机器运行多个项目，环境变量容易冲突。[godotenv](https://github.com/joho/godotenv)库从`.env`文件中读取配置，然后存储到程序的环境变量中，在代码中读取非常方便。`godotenv`源于Ruby开源项目[dotenv](https://github.com/bkeepers/dotenv)

## 快速开始

在go module 中导入：

```
go get -u github.com/joho/godotenv latest
```

```go
// quick-start/main.go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/joho/godotenv"
)

func main() {
	err := godotenv.Load()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("name:", os.Getenv("name"))
	fmt.Println("id:", os.Getenv("id"))
}

```

在当前目录创建`.env`文件：

```
name = dj
id = 110001
```

```sh
$ go run ./main.go
name: kesa
id: 110001
```

默认情况下，`godotenv`读取项目根目录下的`.env`文件，文件中使用`key=value`的格式，每行一个键值对；调用`godotenv.Load`即可加载，直接使用`os.Getenv("key")`即可读取

## 高级特性

### 自动加载

导入`github.com/joho/godotenv/autoload`,配置会自动读取(懒才是第一生产力 ヾ(≧▽≦*)o )

```go
// auto-load/main.go
package main

import (
	"fmt"
	"os"

	_ "github.com/joho/godotenv/autoload"
)

func main() {
	fmt.Println("name:", os.Getenv("name"))
	fmt.Println("id:", os.Getenv("id"))
}
```

在导入的包名之前加`_`会执行包的`init`函数，使用`_`导入包只是为了使用包副作用(side-effect)，不会导入包的其他内容

查看`github.com/joho/godotenv/autoload`:

```go
// github.com/joho/godotenv/autoload/autoload.go
package autoload

/*
	You can just read the .env file on import just by doing

		import _ "github.com/joho/godotenv/autoload"

	And bob's your mother's brother
*/

import "github.com/joho/godotenv"

func init() {
	godotenv.Load()
}

```

n(\*≧▽≦\*)n

### 加载自定义文件

默认情况下加载`.env`文件，也可以加载任意名称的文件(不必`.env`结尾)

```go
// godotenv/custom-env-file/main.go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/joho/godotenv"
	_ "github.com/joho/godotenv/autoload"
)

func main() {
	err := godotenv.Load("common", "dev.env", "production.env")
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("App name:", os.Getenv("app_name"))
	fmt.Println("Version::", os.Getenv("version"))
	fmt.Println("Database:", os.Getenv("database"))

}
```

创建配置文件

`common`:

```
app_name = godotenv_note
version = 0.0.1
```

`dev.env`:

```
database = sqlite
```

`production.env`:

```
database = mysql
```

```sh
$ go run ./main.go
App name: godotenv_note
Version:: 0.0.1
Database: sqlite
```

`Load`读取配置时，若多个文件中出现相同的Key，先出现的优先读取，后续的Key不会生效

### 注释

`.env`文件可以添加注释，以`#`开始行尾结束

```
# app name
app_name = godotenv_note
# app version
version = 0.0.1
```

### YAML

文件可以使用YAML格式:

```yaml
# godotenv/custom-env-file/app_common.yml
# app name
app_name: godotenv_note
# app version
version: 0.0.1
```

```go
// godotenv/yaml-env/main.go
package main

import (
	"fmt"
	"os"

	_ "github.com/joho/godotenv/autoload"
)

func main() {
	fmt.Println("App name:", os.Getenv("app_name"))
	fmt.Println("Version::", os.Getenv("version"))
}

```

```sh
$ go run ./main.go 
App name: godotenv_note
Version:: 0.0.1
```

### 不存入环境变量

`godotenv`可以不将配置存入环境变量，使用`godotenv.Read`返回`map[string]string`可直接使用：

```go
// godotenv/read-env-directly/main.go
package main

import (
	"fmt"
	"log"

	"github.com/joho/godotenv"
	_ "github.com/joho/godotenv/autoload"
)

func main() {
	myEnv, err := godotenv.Read()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("App name:", myEnv["app_name"])
	fmt.Println("version:", myEnv["version"])
}

```

```sh
$ go run ./main.go 
App name: godotenv_note
version: 0.0.1
```

### 从`string`和`io.Reader`中读取

`godotenv`可以直接从`string`中读取：

```go
// godotenv/read-string/main.go
package main

import (
	"fmt"
	"log"

	"github.com/joho/godotenv"
)

func main() {
	content := `
app_name: godotenv_note@str
version: 0.0.1
`
	strEnv, err := godotenv.Unmarshal(content)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("App name:", strEnv["app_name"])
	fmt.Println("version:", strEnv["version"])
}
```

`godotenv`可以从实现了`io.Reader`接口（如`os.File`,`net.Conn`,`bytes.Buffer`等）的数据源中读取

```go
// read-from-reader/main.go
package main

import (
	"bytes"
	"fmt"
	"github.com/joho/godotenv"
	_ "github.com/joho/godotenv/autoload"
	"log"
	"os"
)

func main() {
	file, _ := os.OpenFile(".env", os.O_RDONLY, 0666)
	myEnv, err := godotenv.Parse(file)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("app name:", myEnv["app_name"])
	fmt.Println("version:", myEnv["version"])

	buf := &bytes.Buffer{}
	buf.WriteString("app_name: godotenv_note@buffer")
	buf.WriteString("\n")
	buf.WriteString("version: 0.0.1")
	bufEnv, err := godotenv.Parse(buf)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("app name:", bufEnv["app_name"])
	fmt.Println("version:", bufEnv["version"])
}

```

注意从字符串读取使用`Unmarshal`,从`io.Reader`读取使用`Parse`

### 生成`.env`文件

可以通过程序生成一个`.env`文件

```go
// 
package main

import (
	"bytes"
	"github.com/joho/godotenv"
	"log"
)

func main() {
	// parse env
	buf := &bytes.Buffer{}
	buf.WriteString("app_name: app@env_generate\n")
	buf.WriteString("version: 0.0.1")
	bufEnv, _ := godotenv.Parse(buf)
	// save env
	err := godotenv.Write(bufEnv, "./.env")
	if err != nil {
		log.Fatal(err)
	}
}
```

查看文件:

```sh
$ cat .env
app_name="app@env_generate"
version="0.0.1"
```

也可以生成字符串，这里使用和解析字符串`Unmarshal`相对的`Marshal`:

```go
// marshal-str/main.go
package main

import (
	"bytes"
	"fmt"
	"github.com/joho/godotenv"
	"log"
)

func main() {
	buf := &bytes.Buffer{}
	buf.WriteString("app: @buf\n")
	buf.WriteString("ver: 0.1")
	bufEnv, _ := godotenv.Parse(buf)

	content, err := godotenv.Marshal(bufEnv)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("String env: ", content)
}
```

### 多环境配置

在生产上一般会根据环境加载不同的配置文件

```go
package main

import (
	"fmt"
	"github.com/joho/godotenv"
	"log"
	"os"
)

func main() {
	appEnv := os.Getenv("APP_ENV")
	if appEnv == "" {
		appEnv = "dev"
	}
	err := godotenv.Load(".env." + appEnv)
	if err != nil {
		log.Fatal(err)
	}
	// read basic env
	err = godotenv.Load()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("app:", os.Getenv("app"))
	fmt.Println("version:", os.Getenv("version"))
	fmt.Println("database:", os.Getenv("database"))
}

```

上例先读取环境变量`APP_ENV`来获取对应的配置`.env.`+env,最后读取默认的`env`文件

由于只有先读取到的值才生效，所以在`.env`配置中配置基础信息和默认值，在根据开发/测试/环境配置相关的值，写到对应的`.env.dev/.env.test/.env.production`

`.env`文件：

```
app: multi_env
version: 0.0.1
```

`.env.dev`文件：

```
database: sqlite3
```

`.env.prd`文件：

```
database: mysql
```

运行程序：

```sh
# 默认dev环境
$ go run ./main.go     
app: multi_env
version: 0.0.1
database: sqlite3
# prd环境
$ APP_ENV=prd go run ./main.go    
app: multi_env
version: 0.0.1
database: mysql
```

## 命令行

`godotenv`提供了命令行程序：

```sh
$ go get github.com/joho/godotenv/cmd/godotenv
```

```
$ godotenv -f ./.env COMMAND ARGS
```

程序`godotenv`会被安装在`\$GOPAHT/bin`中，将其加入`$PATH`即可全局调用

`-f`指定配置文件(默认为`.env`),示例如下：

```go
// godotenv-cli
package main

import (
	"fmt"
	"os"
)

func main() {
	fmt.Println("app", os.Getenv("app"))
	fmt.Println("ver", os.Getenv("ver"))
}
```

```sh
$ godotenv -f ./.env go run main.go
app @cli
ver 0.1
```

## 源码

`godotenv`读取的是文件的内容，但是可以调用`os.Getenv`访问

```go
func Load(filenames ...string) (err error) {
	filenames = filenamesOrDefault(filenames)

	for _, filename := range filenames {
		err = loadFile(filename, false)
		if err != nil {
			return // return early on a spazout
		}
	}
	return
}

func loadFile(filename string, overload bool) error {
	envMap, err := readFile(filename)
	if err != nil {
		return err
	}

	currentEnv := map[string]bool{}
	rawEnv := os.Environ()
	for _, rawEnvLine := range rawEnv {
		key := strings.Split(rawEnvLine, "=")[0]
		currentEnv[key] = true
	}

	for key, value := range envMap {
		if !currentEnv[key] || overload {
			os.Setenv(key, value)
		}
	}

	return nil
}
```

可以看到`godotenv`将配置通过`os.Setenv`设置到环境变量中了

## 总结

本文介绍了`godotenv`的基本和高级用法，在多环境开发中，可以考虑使用`godotenv`来进行环境切换

## 参考

1. [godotenv](https://github.com/joho/godotenv) github repo
2. [godotenv](https://pkg.go.dev/github.com/joho/godotenv) godocs
3. [Go 每日一库之 godotenv](https://darjun.github.io/2020/02/12/godailylib/godotenv/) darjun blog
4. [Go Modules Reference](https://golang.org/ref/mod) go module



