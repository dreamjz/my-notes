#  Golang daily lib -- cast

## Introduction 简介

[cast](https://github.com/spf13/cast)是一个小巧使用的类型转换库，用于将一个类型转换成另一个类型，最初是用在[hugo](https://github.com/gohugoio/hugo)中的

## Quick start 快速开始

在go module中导入：

```
go get -u github.com/spf13/cast v1.4.1
```

```go
package main

import (
	"fmt"
	"github.com/spf13/cast"
)

func main() {
	// ToString
	fmt.Println(cast.ToString("dreamjz"))          // dreamjz
	fmt.Println(cast.ToString(8))                  // 8
	fmt.Println(cast.ToString(8.31))               // 8.31
	fmt.Println(cast.ToString([]byte("one time"))) // one time
	fmt.Println(cast.ToString(nil))                // ""

	var foo interface{} = "one more time"
	fmt.Println(cast.ToString(foo)) // one more time

	// To int
	fmt.Println(cast.ToInt(8))     // 8
	fmt.Println(cast.ToInt(8.31))  // 8.31
	fmt.Println(cast.ToInt("8"))   // 8
	fmt.Println(cast.ToInt(true))  // 1
	fmt.Println(cast.ToInt(false)) // 0

	var eight interface{} = 8
	fmt.Println(cast.ToInt(eight)) // 8
	fmt.Println(cast.ToInt(nil))   // 0
}

```

`cast`实现了多种常见类型之间的相互转换，返回最符合直觉的结果。例如：

- `nil`转`string`的结果为`""`,而不是`"nil"`
- `true`转为`string`的结果为`true`,`true`转`int`的结果为`1`
- `interface{}`转为其他类型，要看其存储的具体值

这些类型包括所有的基本类型(整形、浮点型、布尔值和字符串)、空接口、`nil`,时间（`time.Time`）、时间间隔(`time.Duration`)以及它们的切片类型，还有`map[string]Type`（Type为前面提到的类型）:

```
byte     bool      float32    float64    string  
int8     int16     int32      int64      int
uint8    uint16    uint32     uint64     uint
interface{}   time.Time  time.Duration   nil
```

## 高级转换

`cast`提供了两组函数：

- `ToType`（Type为任何支持的类型），将参数转换为`Type`类型。如果无法转换，返回`Type`类型的零值或`nil`
- `ToTypeE`以E结尾,返回转换后的值和一个`error`。这组函数可以区分参数中实际存储了零值还是转换失败

实现的代码大部分类似，`ToType`在内部调用`ToTypeE`函数，返回结果并忽略错误。`ToType`函数的实现在文件`cast.go`中，而`ToTpyeE`在文件`caste.go`中

```go
// cast/cast.go
func ToBool(i interface{}) bool {
  v, _ := ToBoolE(i)
  return v
}

// ToDuration casts an interface to a time.Duration type.
func ToDuration(i interface{}) time.Duration {
  v, _ := ToDurationE(i)
  return v
}
```

`ToTypeE`函数接收任意类型参数(`interface{}`),之后使用类型断言根据具体类型来执行不同的转换，若无法转换则返回错误

```go
// cast/caste.go
func ToBoolE(i interface{}) (bool, error) {
  i = indirect(i)

  switch b := i.(type) {
  case bool:
    return b, nil
  case nil:
    return false, nil
  case int:
    if i.(int) != 0 {
      return true, nil
    }
    return false, nil
  case string:
    return strconv.ParseBool(i.(string))
  default:
    return false, fmt.Errorf("unable to cast %#v of type %T to bool", i, i)
  }
}
```

首先调用`indirect`函数将参数中可能的指针去除。若类型本身不是指针，那么直接返回，否则返回指针指向的值。循环直到返回一个非指针的值：

```go
// cast/caste.go
func indirect(a interface{}) interface{} {
  if a == nil {
    return nil
  }
  if t := reflect.TypeOf(a); t.Kind() != reflect.Ptr {
    // Avoid creating a reflect.Value if it's not a pointer.
    return a
  }
  v := reflect.ValueOf(a)
  for v.Kind() == reflect.Ptr && !v.IsNil() {
    v = v.Elem()
  }
  return v.Interface()
}
```

所以下面的输出均为 8 ：

```go
func main(){
    // pointer
	p := new(int)
	*p = 8
	fmt.Println(cast.ToInt(p)) // 8

	pp := &p
	fmt.Println(cast.ToInt(pp)) // 8
}
```

## 时间类型转换

时间类型转换的代码如下：

```go
func ToTimeE(i interface{}) (tim time.Time, err error) {
  i = indirect(i)

  switch v := i.(type) {
  case time.Time:
    return v, nil
  case string:
    return StringToDate(v)
  case int:
    return time.Unix(int64(v), 0), nil
  case int64:
    return time.Unix(v, 0), nil
  case int32:
    return time.Unix(int64(v), 0), nil
  case uint:
    return time.Unix(int64(v), 0), nil
  case uint64:
    return time.Unix(int64(v), 0), nil
  case uint32:
    return time.Unix(int64(v), 0), nil
  default:
    return time.Time{}, fmt.Errorf("unable to cast %#v of type %T to Time", i, i)
  }
}
```

根据传入的类型执行不同的处理：

- 若为`time.Time`直接返回
- 若为整型，将参数作为时间戳(自UTC时间`1970.01.01 00:00:00`到现在的秒数)调用`time.Unix`生成时间。`Unix`接收两个参数，第一个参数指定为秒，第二个参数指定纳秒
- 如果是字符串，调用`StringToDate`函数依次尝试以下时间格式调用`time.Parse`解析该字符串。如果某个格式解析成功，则返回获得的`time.Time`。否则解析失败返回错误
- 任何其他类型均无法转换成`time.Time`

字符串转换为时间：

```go
// cast/caste.go
func StringToDate(s string) (time.Time, error) {
  return parseDateWith(s, []string{
    time.RFC3339,
    "2006-01-02T15:04:05", // iso8601 without timezone
    time.RFC1123Z,
    time.RFC1123,
    time.RFC822Z,
    time.RFC822,
    time.RFC850,
    time.ANSIC,
    time.UnixDate,
    time.RubyDate,
    "2006-01-02 15:04:05.999999999 -0700 MST", // Time.String()
    "2006-01-02",
    "02 Jan 2006",
    "2006-01-02T15:04:05-0700", // RFC3339 without timezone hh:mm colon
    "2006-01-02 15:04:05 -07:00",
    "2006-01-02 15:04:05 -0700",
    "2006-01-02 15:04:05Z07:00", // RFC3339 without T
    "2006-01-02 15:04:05Z0700",  // RFC3339 without T or timezone hh:mm colon
    "2006-01-02 15:04:05",
    time.Kitchen,
    time.Stamp,
    time.StampMilli,
    time.StampMicro,
    time.StampNano,
  })
}

func parseDateWith(s string, dates []string) (d time.Time, e error) {
  for _, dateType := range dates {
    if d, e = time.Parse(dateType, s); e == nil {
      return
    }
  }
  return d, fmt.Errorf("unable to parse date: %s", s)
}
```

`time.Duration`类型转换如下：

```go
// cast/caste.go
func ToDurationE(i interface{}) (d time.Duration, err error) {
  i = indirect(i)

  switch s := i.(type) {
  case time.Duration:
    return s, nil
  case int, int64, int32, int16, int8, uint, uint64, uint32, uint16, uint8:
    d = time.Duration(ToInt64(s))
    return
  case float32, float64:
    d = time.Duration(ToFloat64(s))
    return
  case string:
    if strings.ContainsAny(s, "nsuµmh") {
      d, err = time.ParseDuration(s)
    } else {
      d, err = time.ParseDuration(s + "ns")
    }
    return
  default:
    err = fmt.Errorf("unable to cast %#v of type %T to Duration", i, i)
    return
  }
}
```

根据传入的类型进行不同的处理：

- 若为`time.Duration`类型，直接返回
- 若为整型或浮点型，将其数值强制转换为`time.Duration`类型，单位默认为`ns`
- 若为字符串，分为两种情况：
  1. 字符串中有时间单位：`nsuµmh`,直接调用`time.ParseDuration`
  2. 否则拼接`ns`后调用`time.ParseDuration`
- 其他类型解析失败

示例：

```go
package main

import (
	"fmt"
	"github.com/spf13/cast"
	"time"
)

func main() {
	now := time.Now()
	timestamp := 1579615973
	timeStr := "2021-10-26 17:29:00"

	fmt.Println(cast.ToTime(now))       // 2021-10-26 17:35:55.35905014 +0800 CST m=+0.000115363
	fmt.Println(cast.ToTime(timestamp)) // 2020-01-21 22:12:53 +0800 CST
	fmt.Println(cast.ToTime(timeStr))   // 2021-10-26 17:29:00 +0000 UTC

	d, _ := time.ParseDuration("1m30s")
	ns := 30000
	strWithUnit := "130s"
	strWithoutUnit := "130"

	fmt.Println(cast.ToDuration(d))              // 1m30s
	fmt.Println(cast.ToDuration(ns))             // 30µs
	fmt.Println(cast.ToDuration(strWithUnit))    // 2m10s
	fmt.Println(cast.ToDuration(strWithoutUnit)) // 130ns
}

```

## 切片类型转换

实际上，这些函数的实现逻辑基本类似。使用类型断言判断类型。若为目标类型直接返回。否则进行响应的转换

我们主要分析两个实现：`ToIntSliceE`和`ToStringSliceE`。`ToBoolSliceE/ToDurationSliceE`与`ToIntSlice`基本相同

首先是`ToIntSliceE`：

```go
func ToIntSliceE(i interface{}) ([]int, error) {
  if i == nil {
    return []int{}, fmt.Errorf("unable to cast %#v of type %T to []int", i, i)
  }

  switch v := i.(type) {
  case []int:
    return v, nil
  }

  kind := reflect.TypeOf(i).Kind()
  switch kind {
  case reflect.Slice, reflect.Array:
    s := reflect.ValueOf(i)
    a := make([]int, s.Len())
    for j := 0; j < s.Len(); j++ {
      val, err := ToIntE(s.Index(j).Interface())
      if err != nil {
        return []int{}, fmt.Errorf("unable to cast %#v of type %T to []int", i, i)
      }
      a[j] = val
    }
    return a, nil
  default:
    return []int{}, fmt.Errorf("unable to cast %#v of type %T to []int", i, i)
  }
}
```

根据传入参数的类型：

- 若为`nil`,直接返回错误
- 若为`[]int`,不用转换，直接返回
- 若为**切片**或**数组**，新建[]int,将每个元素转换为`int`类型后放入切片并返回
- 其他类型不可转换

`ToStringSliceE`：

```go
func ToStringSliceE(i interface{}) ([]string, error) {
  var a []string

  switch v := i.(type) {
  case []interface{}:
    for _, u := range v {
      a = append(a, ToString(u))
    }
    return a, nil
  case []string:
    return v, nil
  case string:
    return strings.Fields(v), nil
  case interface{}:
    str, err := ToStringE(v)
    if err != nil {
      return a, fmt.Errorf("unable to cast %#v of type %T to []string", i, i)
    }
    return []string{str}, nil
  default:
    return a, fmt.Errorf("unable to cast %#v of type %T to []string", i, i)
  }
}
```

根据传入的参数类型：

- 若为`[]interface{}`，将每个元素转为`string`并返回结果切片
- 若为`[]string`,无需转换，直接返回
- 若为`interface{}`，将参数转为`string`,返回只包含这个值的切片
- 若为`string`,调用`strings.Fields`函数按空白符将参数拆分，返回拆分的字符串切片
- 其他情况，不能转换

示例：

```go
func main() {
	sliceOfInt := []int{1, 2, 7}
	arrayOfInt := [3]int{8, 12}

	// ToIntSlice
	fmt.Println(cast.ToIntSlice(sliceOfInt)) // [1,3,7]
	fmt.Println(cast.ToIntSlice(arrayOfInt)) // [8,12,0]

	sliceOfInterface := []interface{}{1, 2.0, "kesa"}
	sliceOfString := []string{"a", "b", "cd"}
	stringFields := " abc def hij"
	any := interface{}(37)
	// ToStringSlice
	fmt.Println(cast.ToStringSlice(sliceOfInterface)) // [1 2 kesa]
	fmt.Println(cast.ToStringSlice(sliceOfString))    // [a b cd]
	fmt.Println(cast.ToStringSlice(stringFields))     // [abc def hij]
	fmt.Println(cast.ToStringSlice(any))              // 37

}
```

## map[string]Type 类型转换

`cast`能将传入的参数转换为`map[stirng]Type`类型，`Type`其支持的所有类型

实现基本相同，下面分析下`ToStringMapString`:

```go
func ToStringMapStringE(i interface{}) (map[string]string, error) {
  var m = map[string]string{}

  switch v := i.(type) {
  case map[string]string:
    return v, nil
  case map[string]interface{}:
    for k, val := range v {
      m[ToString(k)] = ToString(val)
    }
    return m, nil
  case map[interface{}]string:
    for k, val := range v {
      m[ToString(k)] = ToString(val)
    }
    return m, nil
  case map[interface{}]interface{}:
    for k, val := range v {
      m[ToString(k)] = ToString(val)
    }
    return m, nil
  case string:
    err := jsonStringToObject(v, &m)
    return m, err
  default:
    return m, fmt.Errorf("unable to cast %#v of type %T to map[string]string", i, i)
  }
}
```

根据传入的类型：

- 若为`map[string]string`则直接返回
- 若为`map[string]interface{}`，将每个value转换为`string`存入新的map并返回
- 若为`map[interface{}]string`，将每个key转换为`string`存入新的map并返回
- 若为`map[interface{}]interface{}`,将key，value均转换为`string`存入新的map并返回
- 若为`string`，将其视作JSON解析到`map[string]string`并返回
- 其他情况则返回错误

示例：

```go
func main() {
	m1 := map[string]string{
		"name":   "kesa",
		"gender": "male",
	}

	m2 := map[string]interface{}{
		"name":   "kesa",
		"gender": "male",
	}

	m3 := map[interface{}]string{
		"name": "miao",
		"age":  "10",
	}

	m4 := map[interface{}]interface{}{
		"name": "miao",
		"age":  25,
	}

	jsonStr := `{"name":"pp","age": 222}`

	// ToStringMapString
	fmt.Println(cast.ToStringMapString(m1))      // map[gender:male name:kesa]
	fmt.Println(cast.ToStringMapString(m2))      // map[gender:male name:kesa]
	fmt.Println(cast.ToStringMapString(m3))      // map[age:10 name:miao]
	fmt.Println(cast.ToStringMapString(m4))      // map[age:25 name:miao]
	fmt.Println(cast.ToStringMapString(jsonStr)) // map[age: name:pp]

	m5 := make(map[string]interface{})
	json.Unmarshal([]byte(jsonStr), &m5)
	fmt.Println(m5) // map[age:222 name:pp]
}
```

注意上述的`jsonStr`中含有`Number`类型，是不能被转换成golang的`string`,所以`ToStringMapString`转换的结果中`age`为空字符串(其解析调用了`json.Unmarshal`)

## Conclusion 总结

`cast`库能在几乎所有常见类型之间转换，使用非常方便。代码量也很小，建议阅读源码

## Reference 参考

1. [cast](https://github.com/spf13/cast) GitHub-repo

2. [cast](https://darjun.github.io/2020/01/20/godailylib/cast/) darjun/blog

3. [decoding-json-numbers-into-strings-in-go](http://igorsobreira.com/2015/04/11/decoding-json-numbers-into-strings-in-go.html) igorsobreira

4. [Decoding JSON int into string](https://stackoverflow.com/questions/24480835/decoding-json-int-into-string) stackoverflow





