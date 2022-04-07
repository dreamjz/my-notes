---
title: 'Gee Web'
date: '2022-04-02'
categories: 
 - 'golang'
publish: true
---

## 1. Day0 - 设计框架

Golang 中标准库 `net/http` 提供了基础的 Web 功能，即监听端口，映射静态路由，解析静态路由和 HTTP 报文。但是一些 Web 开发中的简单需求并不支持，需要手工实现：

- 动态路由：例如`hello/:name`，`hello/*`这类的规则。
- 鉴权：没有分组/统一鉴权的能力，需要在每个路由映射的handler中实现。
- 模板：没有统一简化的HTML机制。
- …

当我们离开框架，使用标准库，需要频繁手动处理的地方，就是框架的价值所在。

## 2. Day1 - HTTP 基础

- `net/http` 及 `http.Handler` 的简单使用
- `Gee` 框架的雏形

### 2.1 标准库启动 Web 服务

[day1-http-base/base1/main.go]()

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", indexHandler)
	http.HandleFunc("/hello", helloHandler)
	log.Fatal(http.ListenAndServe(":9999", nil))
}

// handler echoes r.URL.Path
func indexHandler(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
}

// handler echoes r.URL.Header
func helloHandler(w http.ResponseWriter, req *http.Request) {
	for k, v := range req.Header {
		fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
	}
}

```

上述代码设置了两个路由 `/` 和 `/hello`，分别绑定 `indexHandler` 和 `helloHandler`，根据不同的 HTTP 请求会调用不同的处理函数。简单测试一下：

```sh
$ curl 'http://localhost:9999/'
URL.Path = "/"
$ curl 'http://localhost:9999/hello'
Header["Accept"] = ["*/*"]
Header["Accept-Encoding"] = ["gzip"]
Header["User-Agent"] = ["curl/7.82.0"]
```

`http.ListenAndServe` 函数用于启动 Web 服务，第一个参数为地址，而第二个参数代表处理所有 HTTP 请求的实例，`nil` 表示使用标准库中的实例处理。第二个参数就是利用 `net/http` 标准库实现 Web 框架的入口。

### 2.2 实现 http.Handler 接口

```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

`ListenAndServe` 函数的第二个参数类型为 `Handler` 接口，只有一个 `ServeHTTP` 方法，也就是说只要传入实现了 `Handler` 接口的实例就可以用于处理所有的 HTTP 请求了。

 ```go
 package main
 
 import (
 	"fmt"
 	"log"
 	"net/http"
 )
 
 type Engine struct{}
 
 func (e *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
 	switch req.URL.Path {
 	case "/":
 		fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
 	case "/hello":
 		for k, v := range req.Header {
 			fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
 		}
 	default:
 		fmt.Fprintf(w, "404 NOT FOUND: %s\n", req.URL)
 	}
 }
 
 func main() {
 	engine := new(Engine)
 	log.Fatal(http.ListenAndServe(":9999", engine))
 }
 ```

- `Engine`: 空结构体， 实现了 `ServeHTTP` 方法。参数 `Request` 包含 HTTP 请求的所有信息，参数 `ResponseWriter`，用于构造请求响应信息；
- 在 `main` 函数中，给 `ListenAndServe` 函数的第二个参数传入自定义的 `engine` 实例，将所有的 HTTP 请求转向了自己的处理逻辑。此时我们拥有了统一的控制入口，我们可以自定义路由映射规则，也可以添加处理逻辑，如日志、异常处理等；

### 2.3 Gee 框架雏形

重新组织上述代码，可以搭建出整个框架雏形，目录结构如下：

```
base3
├── gee
│   ├── gee.go
│   └── go.mod
├── go.mod
└── main.go
```



```
module base3

go 1.16

require gee v0.0.0

replace gee => ./gee
```

- `replace`: 将 gee 指向 `./gee`

main.go()

```go
package main

import (
	"fmt"
	"gee"
	"net/http"
)

func main() {
	r := gee.New()
	r.GET("/", func(w http.ResponseWriter, req *http.Request) {
		fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
	})
	r.GET("/hello", func(w http.ResponseWriter, req *http.Request) {
		for k, v := range req.Header {
			fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
		}
	})
	r.Run(":9999")
}
```

gee.go()

```go
package gee

import (
	"fmt"
	"net/http"
)

// HandlerFunc defines the request handler used by gee
type HandlerFunc func(http.ResponseWriter, *http.Request)

// Engine implement the interface of http.Handler
type Engine struct {
	router map[string]HandlerFunc
}

// New is the constructor of gee.Engine
func New() *Engine {
	return &Engine{router: make(map[string]HandlerFunc)}
}

func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) {
	key := method + "-" + pattern
	engine.router[key] = handler
}

// GET defines the method to add GET request
func (engine *Engine) GET(pattern string, handler HandlerFunc) {
	engine.addRoute("GET", pattern, handler)
}

// POST defines the method to add POST request
func (engine *Engine) POST(pattern string, handler HandlerFunc) {
	engine.addRoute("POST", pattern, handler)
}

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	key := req.Method + "-" + req.URL.Path
	if handler, ok := engine.router[key]; ok {
		handler(w, req)
	} else {
		fmt.Fprintf(w, "404 NOT FOUND: %s\n", req.URL)
	}
}

// Run defines the method to start a http server
func (engine *Engine) Run(addr string) error {
	return http.ListenAndServe(addr, engine)
}
```

- `HandlerFunc` ：用于定义路由映射的处理方法。`Engine` 中维护了一个路由表 `router` ，`key`由请求方法和静态路由地址组成，`value` 是对应的处理方法；
- `Run`：对 `http.ListenAndServer` 的封装；

## 3. Day2 - 上下文

- 将路由 (router) 独立出来；
- 设计上下文 (Context), 封装 `Request` 和 `Response` ，提供对 JSON、HTML 等返回类型的支持；

### 3.1 设计 Context

1. 对于 Web 服务来说，无非是根据请求 `*http.Request` ，构造响应 `http.ResponseWriter`。但是这两个对象提供的接口粒度太细，例如要构造一个完整的响应，需要考虑消息头 (Header) 和消息体 (Body)，而 Header 包含了状态码 `StatusCode` ，消息类型 (ContentType) 等几乎每次请求都需要设置的信息。因此，若不进行有效的封装，用户就需要编写大量重复、繁杂的代码，并且容易出错。所以，针对常用的场景，高效的构造出 HTTP 响应是一个好的框架需要考虑的点。

以下是返回 JSON 数据，封装前后的区别：

- 封装前

  ```go
  obj := map[string]interface{}{
      "name": "dreamjz",
      "password": "1234",
  }
  w.Header().Set("Content-Type", "application/json")
  w.WriteHeader(http.StatusOK)
  encoder := json.NewEncoder(w)
  if err := encoder.Encode(obj); err != nil {
      http.Error(w, err.Error(), 500)
  }
  ```

- 封装后

  ```go
  c.JSON(http.StatusOK, gee.H{
      "username": c.PostForm("username"),
      "password": c.PostForm("password"),
  })
  ```

2. 对于框架来说，还需要支撑额外的功能，例如解析动态路由 `/hello/:name`，`:name` 值的存储，中间件产生的信息等。Context 随着每一个请求的出现而产生，请求的结束而销毁，和当前请求强相关的信息都应由 Context 承载。因此，设计 Context 结构，将扩展性和复杂性保留在内部，对外则简化接口。路由处理函数和中间件的参数统一使用 Context 实例，Context 就会像一次会话百宝箱，可以找到任何东西。

### 3.2 具体实现

#### Context

[]()

```go
package gee

import (
	"encoding/json"
	"fmt"
	"net/http"
)

// H is the alias of map[string]interface{}
type H map[string]interface{}

type Context struct {
	// origin objects
	Writer http.ResponseWriter
	Req    *http.Request
	// request info
	Path   string
	Method string
	// response info
	StatusCode int
}

func newContext(w http.ResponseWriter, req *http.Request) *Context {
	return &Context{
		Writer: w,
		Req:    req,
		Path:   req.URL.Path,
		Method: req.Method,
	}
}

// PostForm returns the form value from request
func (c *Context) PostForm(key string) string {
	return c.Req.FormValue(key)
}

// Query returns the query string from request
func (c *Context) Query(key string) string {
	return c.Req.URL.Query().Get(key)
}

// Status set the http status code for context and response
func (c *Context) Status(code int) {
	c.StatusCode = code
	c.Writer.WriteHeader(code)
}

// SetHeader set the http header for response
func (c *Context) SetHeader(key, value string) {
	c.Writer.Header().Set(key, value)
}

// String set response with string format
func (c *Context) String(code int, format string, values ...interface{}) {
	c.SetHeader("Content-Type", "text-plain")
	c.Status(code)
	c.Writer.Write([]byte(fmt.Sprintf(format, values...)))
}

// JSON set response with JSON format
func (c *Context) JSON(code int, obj interface{}) {
	c.SetHeader("Content-Type", "application/json")
	c.Status(code)
	encoder := json.NewEncoder(c.Writer)
	if err := encoder.Encode(obj); err != nil {
		http.Error(c.Writer, err.Error(), 500)
	}
}

// Data write data to response
func (c *Context) Data(code int, data []byte) {
	c.Status(code)
	c.Writer.Write(data)
}

// HTML set response with HTML format
func (c *Context) HTML(code int, html string) {
	c.SetHeader("Content-Type", "text/html")
	c.Status(code)
	c.Writer.Write([]byte(html))
}
```

- `H`： `map[string]interface{}` 的别名，可以更加简洁的构建 JSON 数据；
- `Context`：目前包含 `http.ResponseWriter` 和 `*http.Request` 以及 Method 和 Path 两个常用属性；
- 提供了访问 `Query` 和 `PostForm` 的方法；
- 提供了快速构造 `String/Data/JSON/HTML` 响应的方法；

#### Router

将路由相关的方法和结构提取出来，放入新的文件 `router.go` 中，方便后续功能的增强，如动态路由支持等；将 `handle` 方法的参数改为 `Context` 类型。

```go
package gee

import (
	"log"
	"net/http"
)

type router struct {
	handlers map[string]HandlerFunc
}

func newRouter() *router {
	return &router{handlers: make(map[string]HandlerFunc)}
}

func (r *router) addRoute(method, pattern string, handler HandlerFunc) {
	log.Printf("Route %4s - %s", method, pattern)
	key := method + "-" + pattern
	r.handlers[key] = handler
}

func (r *router) handle(c *Context) {
	key := c.Method + "-" + c.Path
	if handler, ok := r.handlers[key]; ok {
		handler(c)
	} else {
		c.String(http.StatusNotFound, "404 NOT FOUND: %s\n", c.Path)
	}
}

```



#### Gee



```go
package gee

import "net/http"

// HandlerFunc defines the request handler used by gee
type HandlerFunc func(c *Context)

// Engine implements the interface http.Handler
type Engine struct {
	router *router
}

// New is the constructor of gee.Engine
func New() *Engine {
	return &Engine{router: newRouter()}
}

func (e *Engine) addRoute(method, pattern string, handler HandlerFunc) {
	e.router.addRoute(method, pattern, handler)
}

// GET defines the method to add GET request
func (e *Engine) GET(pattern string, handler HandlerFunc) {
	e.addRoute("GET", pattern, handler)
}

// POST defines the method to add POST request
func (e *Engine) POST(pattern string, handler HandlerFunc) {
	e.addRoute("POST", pattern, handler)
}

func (e *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	c := newContext(w, req)
	e.router.handle(c)
}

// Run defines the method to start a http server
func (e *Engine) Run(addr string) error {
	return http.ListenAndServe(addr, e)
}
```

在调用 `router.handle` 之前先构造了一个 `gee.Context` 对象。

#### Main



```go
package main

import (
	"gee"
	"net/http"
)

func main() {
	r := gee.New()

	r.GET("/", func(c *gee.Context) {
		c.HTML(http.StatusOK, "<h1>Hello Gee</h1>")
	})

	r.GET("/hello", func(c *gee.Context) {
		c.String(http.StatusOK, "Hello %s, you are at %s\n", c.Query("name"), c.Path)
	})

	r.POST("/login", func(c *gee.Context) {
		c.JSON(http.StatusOK, gee.H{
			"username": c.PostForm("username"),
			"password": c.PostForm("password"),
		})
	})

	r.Run(":9999")
}
```

Run and Test ：

```sh
$ curl 'http://localhost:9999/'
<h1>Hello Gee</h1>%           
$ curl 'http://localhost:9999/hello?name=kesa'
Hello kesa, you are at /hello
$ url -X POST -d "username=kesa&password=1234" 'http://localhost:9999/login'
{"password":"1234","username":"kesa"}
```

## 4. Day3 - 前缀树路由

- 使用 Trie 树实现动态路由 (dynamic route)；
- 支持两种模式 `:name` 和 `*filepath`；

### 4.1 Trie 树

之前使用 `map` 结构来存储路由表， 虽然索引比较高效，但是只能支持**静态路由**。而动态路由，即一条路由规则可以匹配某一类型而非一条固定的路由。例如 `hello/:name`，可以匹配 `/hello/kesa`, `/hello/geektutu`。

动态路由实现方式有很多，例如在 `gorouter`中使用的是正则表达式，而 `gin` 中使用的是 Trie 树。

**Trie**，又称**前缀树**或**字典树**，是一种有序树，用于保存关联数组，其中的键通常为字符串[^1]。前缀树的**每一个节点的所有子节点都拥有相同的前缀**。这种结构非常适用于路由匹配，例如定义如下路由规则：

- /:lang/doc
- /:lang/tutorial
- /:lang/intro
- /about
- /p/blog
- /p/related

可以用如下前缀树来表示：

![trie tree](image/trie_router.jpg)

HTTP 请求的路径恰好是 `/` 分隔的多段构成，因此每一段可以作为前缀树的一个节点。我们通过树结构查询，如果中间某一层的节点都不满足条件，则说明没有匹配到路由，查询结束。

#### 实现

// TODO: file URL

```go
package gee

import (
	"fmt"
	"strings"
)

// Trie 树节点
type node struct {
	pattern  string  // 待匹配的路由
	part     string  // 当前节点对应部分
	children []*node // 子节点
	isWild   bool    // 通配标志
}

func (n *node) String() string {
	return fmt.Sprintf("node{pattern= %s, part= %s, isWild=%t", n.pattern, n.part, n.isWild)
}

// 寻找第一个匹配的节点
func (n *node) matchChild(part string) *node {
	// 查找子节点
	for _, child := range n.children {
		if child.part == part || child.isWild {
			return child
		}
	}
	return nil
}

// 寻找所有匹配的节点
func (n *node) matchChildren(part string) *[]*node {
	nodes := make([]*node, 0)
	for _, child := range n.children {
		if child.part == part || child.isWild {
			nodes = append(nodes, child)
		}
	}
	return &nodes
}

// 插入新节点
func (n *node) insert(pattern string, parts []string, height int) {
	// 递归出口
	if len(parts) == height {
		// 记录匹配模式
		n.pattern = pattern
	}

	part := parts[height]
	// 查找匹配节点
	child := n.matchChild(part)
	if child == nil {
		// 未找到则新增节点
		child = &node{part: part, isWild: part[0] == ':' || part[0] == '*'}
		n.children = append(n.children, child)
	}

	// 递归新增节点
	child.insert(pattern, parts, height+1)
}

// 查找匹配节点
func (n *node) search(parts []string, height int) *node {
	// 递归出口
	if len(parts) == height || strings.HasPrefix(n.part, "*") {
		// 匹配模式为空，未找到
		if n.pattern == "" {
			return nil
		}
		return n
	}

	part := parts[height]
	// 查找所有匹配的节点
	children := n.matchChildren(part)

	for _, child := range *children {
		// 递归搜索每个子节点
		result := child.search(parts, height+1)
		if result != nil {
			return result
		}
	}
	return nil
}

// 遍历前缀树，获取所有的路由模式节点
func (n *node) travel(list *[]*node) {
	if n.pattern != "" {
		*list = append(*list, n)
	}
	for _, child := range n.children {
		child.travel(list)
	}
}
```

- `node`：前缀树节点；`pattern` 为匹配的模式，`part` 为当前节点对应的路由部分，`children`子节点，`isWild` 为通配标志，当遇到动态参数 `:` 或 通配符`*` 时为 true。
  例如：路由 `/hello/:name/go`，最后一个节点的 `pattern` 为 `/hello/:name/go`， `part` 为 `go`，`isWild` 为 false；
- `insert`：插入新节点；递归查找 (`matchChild`)每一层节点，若没有则新增节点，直到最后一个节点的高度(`height`)和 `len(parts)` 相同时才设置 `pattern` 并退出递归；
- `search`：查找节点；递归查询每一层节点，若 `len(parts) == height ` 或 遇到通配符，则检查 `pattern` 是否为空，不为空则表示找到匹配的节点；

### 4.2 Router



```go
package gee

import "strings"

// router 结构
type router struct {
	roots    map[string]*node       // HTTP 方法，前缀树根节点映射
	handlers map[string]HandlerFunc // 路由，处理函数映射
}

func newRouter() *router {
	return &router{
		roots:    make(map[string]*node),
		handlers: make(map[string]HandlerFunc),
	}
}

// 解析路由匹配模式，只能有一个通配符
func parsePattern(pattern string) []string {
	vs := strings.Split(pattern, "/")

	parts := make([]string, 0)
	for _, val := range vs {
		parts = append(parts, val)
		if val[0] == '*' {
			break
		}
	}
	return parts
}

// 添加路由
func (r *router) addRoute(method, pattern string, handler HandlerFunc) {
	parts := parsePattern(pattern)

	key := method + "-" + pattern
	if _, ok := r.roots[method]; !ok {
		// 前缀树不存在则创建
		r.roots[method] = &node{}
	}
	r.roots[method].insert(pattern, parts, 0)
	r.handlers[key] = handler
}

// 获取路由
func (r *router) getRoute(method, path string) (*node, map[string]string) {
	// 解析当前请求路径 Path
	searchParts := parsePattern(path)
	params := make(map[string]string)
	root, ok := r.roots[method]

	if !ok {
		return nil, nil
	}

	// 搜索前缀树，查找匹配模式
	n := root.search(searchParts, 0)

	if n != nil {
		// 解析匹配模式
		// 因为匹配模式和请求路径解析后的长度相同
		// 可以推断出动态参数或通配参数
		parts := parsePattern(n.pattern)
		for i, part := range parts {
			if part[0] == ':' {
				params[part[1:]] = searchParts[i]
			}
			if part[0] == '*' && len(part) > 1 {
				// 获取通配符之后的内容
				params[part[1:]] = strings.Join(searchParts[i:], "/")
				// 通配符只能有一个
				break
			}
		}
		return n, params
	}
	return nil, nil
}

// 获取指定方法的所有路由模式
func (r *router) getRoutes(method string) []*node {
	root, ok := r.roots[method]
	if !ok {
		return nil
	}
	nodes := make([]*node, 0)
	root.travel(&nodes)
	return nodes
}

// 请求处理
func (r *router) handle(c *Context) {
	n, params := r.getRoute(c.Method, c.Path)
	if n != nil {
		c.Params = params
		key := c.Method + "-" + n.pattern
		r.handlers[key](c)
	}
}
```

- `router`: 路由结构体；
  - `roots`：HTTP Method 和 前缀树根节点组成的映射；例如：POST 方法将会对应自己的路由前缀树；
  - `handlers`：路由和处理函数组成的映射；
- `parsePattern`：将路由模式 / 请求路径解析成字符串切片；例如：`/hello/:name` 将会解析成 `["hello", ":name"]`；
- `addRoute`：添加路由模式和处理函数；将给定的路由模式添加至对应的前缀树中，并添加相关的处理函数；
- `getRoute`：获取请求路径匹配的路由前缀树节点，并解析出动态路由的参数；

### 4.3 Context

Context 相较于之前，添加了 `Params` 属性和 `Param` 方法，用于存储和获取路由参数；



```go
package gee

import (
	"encoding/json"
	"fmt"
	"net/http"
)

// H is alias of map[string]interface{}
type H map[string]interface{}

type Context struct {
	// origin objects
	Writer http.ResponseWriter
	Req    *http.Request
	// request info
	Path   string
	Method string
	Params map[string]string // 路由参数
	// response info
	StatusCode int
}

// the constructor of gee.Context
func newContext(w http.ResponseWriter, req *http.Request) *Context {
	return &Context{
		Writer: w,
		Req:    req,
		Path:   req.URL.Path,
		Method: req.Method,
	}
}

// Param returns the parameter in URL path
func (c *Context) Param(key string) string {
	val, _ := c.Params[key]
	return val
}

// PostForm returns the parameter in form data
func (c *Context) PostForm(key string) string {
	return c.Req.FormValue(key)
}

// Query returns the parameter in query string
func (c *Context) Query(key string) string {
	return c.Req.URL.Query().Get(key)
}

// Status set the StatusCode of gee.Context
// And write status code to response
func (c *Context) Status(code int) {
	c.StatusCode = code
	// 写入响应 HTTP Status Code
	c.Writer.WriteHeader(code)
}

// SetHeader set response header
func (c *Context) SetHeader(key string, val string) {
	c.Writer.Header().Set(key, val)
}

func (c *Context) String(code int, format string, values ...interface{}) {
	c.SetHeader("Content-Type", "text/plain")
	c.Status(code)
	c.Writer.Write([]byte(fmt.Sprintf(format, values...)))
}

func (c *Context) JSON(code int, obj interface{}) {
	c.SetHeader("Content-Type", "application/json")
	c.Status(code)
	encoder := json.NewEncoder(c.Writer)
	if err := encoder.Encode(obj); err != nil {
		http.Error(c.Writer, err.Error(), 500)
	}
}

func (c *Context) Data(code int, data []byte) {
	c.Status(code)
	c.Writer.Write(data)
}

func (c *Context) HTML(code int, html string) {
	c.SetHeader("Content-Type", "text/html")
	c.Status(code)
	c.Writer.Write([]byte(html))
}
```

### 4.4 Gee



```go
package gee

import (
	"log"
	"net/http"
)

type HandlerFunc func(c *Context)

type Engine struct {
	router *router
}

func New() *Engine {
	return &Engine{router: newRouter()}
}

func (e *Engine) addRoute(method, pattern string, handler HandlerFunc) {
	log.Printf("Route: %4s - %s", method, pattern)
	e.router.addRoute(method, pattern, handler)
}

func (e *Engine) GET(pattern string, handler HandlerFunc) {
	e.addRoute("GET", pattern, handler)
}

func (e *Engine) POST(pattern string, handler HandlerFunc) {
	e.addRoute("POST", pattern, handler)
}

func (e *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	c := newContext(w, req)
	e.router.handle(c)
}
```



## Reference

1. [七天用Go从零实现系列](https://geektutu.com/post/gee.html)



[^1]: [Trie Wikipedia](https://zh.wikipedia.org/wiki/Trie)
