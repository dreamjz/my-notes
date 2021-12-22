# Strconv

strconv包提供字符串和简单数据类型之间的类型转换功能，可将简单类型转换成字符串或者将字符串转换为其他简单类型

其函数大致分为几类：

* 字符串转int : Atio()
* int转字符串 : Itoa()
* ParseType类函数将string转换为Type类型：ParseBool(),ParseFloat(),ParseInt(),ParseUnit()
  这些函数有第二个返回参数用以表示是否转换成功
* FormatType类函数将其他类型转换为string ： FormatBool(),FormatFloat(),FormatInt(),FormatUnit()
* AppendType类函数将Type转换成字符串后append到slice中： 
  AppendBool(),AppendFloat(),AppendInt(),AppendUnit()

Tips：查看官方手册可使用`go doc strconv`或者访问https://golang.org/pkg/strconv/

当类型转换错误时，将返回strconv包中自定义的错误：

```go
var ErrRange=errors.New("value out of range")
var ErrSyntax=errors.New("invalid syntax")
```

```go
//例如将A转换成int型
fmt.Print(strconv.Atoi("A"))
//输出：
0 strconv.Atoi: parsing "A": invalid syntax
```

## string和int转换

int转换为string：`func strconv.Itoa(i int) string`

> Atoi is equivalent to ParseInt(s, 10, 0), converted to type int.

Atoi等同于ParseInt(s,10,0),将string转换为int型

```go
var str string=strconv.Itoa(100)
```

string转换为int:`func strconv.Atoi(s string) (int,error)`

> Itoa is equivalent to FormatInt(int64(i), 10).

Itoa等同于FormatInt(int4(i),10)

```go
i,err:=strconv.Atoi("100")
if err != nil {
	log.Fatal("Convert error", err.Error())
}
```

## ParseType类函数

ParseType类函数用于将字符串转换为给定类型的值：ParseBool(),ParseFloat(),ParseInt(),ParseUnit()

因string转换为其他类型可能会失效，故上述函数会返回两种类型参数(string,error)

####  `func strconv.ParseBool(s string) (bool, error)`

> ParseBool returns the boolean value represented by the string. It accepts 1, t, T, TRUE, true, True, 0, f, F, FALSE, false, False. Any other value returns an error.

ParseBool返回字符串表示的布尔值，可接受字符串1,t,T,TRUE,true,True,0,f,F,FALSE,false,False.任何其他的值将会返回错误

#### `func strconv.ParseFloat(s string,bitSize int) float64`

> ParseFloat converts the string s to a floating-point number with the precision specified by bitSize: 32 for float32, or 64 for float64. When bitSize=32, the result still has type float64, but it will be convertible to float32 without changing its value.

ParseFloat将字符串转换为制定精度的float类型，bitSize取32和64分别代表float32和float64.当bitSize为32时，返回结果仍为float64,但是其转换成float32时不会改变其值

#### `func strconv.ParseInt(s string, base int, bitSize int) (i int64, err error)`

> ParseInt interprets a string s in the given base (0, 2 to 36) and bit size (0 to 64) and returns the corresponding value i.
>
> If the base argument is 0, the true base is implied by the string's prefix: 2 for "0b", 8 for "0" or "0o", 16 for "0x", and 10 otherwise. Also, for argument base 0 only, underscore characters are permitted as defined by the Go syntax for integer literals.
>
> The bitSize argument specifies the integer type that the result must fit into. Bit sizes 0, 8, 16, 32, and 64 correspond to int, int8, int16, int32, and int64. If bitSize is below 0 or above 64, an error is returned.

ParseInt根据给定的进制数(0,2-36)和位数(0-64)解析字符串并返回int64型的结果

若base为0，则base取值取决于字符串的前缀："0b"表示2进制，"0"或"0o"表示8进制,"0x"表示16进制，其他则为10进制。当base为0时，可转换使用下划线分隔的整数

biSize指定了返回结果类型，bitSize为0,8,16,32,64分别表示int,int8,int16,int32,int64.如果bitSize小于0或大于64则报错

#### `func strconv.ParseUint(s string, base int, bitSize int) (uint64, error)`

> ParseUint is like ParseInt but for unsigned numbers.

与ParseInt类似但仅适用于无符号数字

### 示例

```go
//strconv_notes.go
package strconvnotes

import (
	"log"
	"strconv"
)

//DoStrconvNotes strconv包笔记
func DoStrconvNotes() {
	//int->string
	intToString()
	//string->int
	stringToInt()
	//ParseType
	parseType()
}

func stringToInt() {
	//int->string
	var i1 = 100
	var str string = strconv.Itoa(i1)
	printConvertLog(i1, str)
}

func intToString() {
	//string->int
	var (
		i   int
		err error
		str = "10"
	)
	i, err = strconv.Atoi(str)
	printConvertLog(str, i)
	hasError(err, "string convert to int")
}

func parseType() {
	//ParseType
	//ParseBool
	var strTrue string = "true"
	b, err := strconv.ParseBool(strTrue)
	hasError(err, "string->bool")
	printConvertLog(strTrue, b)
	//ParseFloat
	var strFloat string = "100.0123"
	//func strconv.ParseFloat(s string, bitSize int) (float64, error)
	//When bitSize=32, the result still has type float64, but it will be convertible to float32 without changing its value.
	f, err := strconv.ParseFloat(strFloat, 64)
	f2, err := strconv.ParseFloat(strFloat, 64)
	hasError(err, "")
	printConvertLog(strFloat, f)
	printConvertLog(strFloat, f2)
	//ParseInt
	//base 10
	var strInt string = "-1000"
	i, err := strconv.ParseInt(strInt, 10, 64)
	hasError(err, "")
	printConvertLog(strInt, i)
	//base 2 and bitSize 32
	i2, err := strconv.ParseInt(strInt, 2, 32)
	hasError(err, "")
	printConvertLog(strInt, i2)
	//转换下划线分隔的整数
	i3, err := strconv.ParseInt("1_2", 0, 64)
	hasError(err, "")
	printConvertLog("1_2", i3)
	//ParseUnit
	var strUnit = "1004"
	//ParseUint is like ParseInt but for unsigned numbers.
	u, err := strconv.ParseUint(strUnit, 10, 64)
	hasError(err, "")
	printConvertLog(strUnit, u)
}

func hasError(err error, message string) {
	if err != nil {
		log.Fatal(message, err.Error())
	}
}

func printConvertLog(d1 interface{}, d2 interface{}) {
	log.Printf("[%v:%T]->[%v:%T]", d1, d1, d2, d2)
}
//main.go
func main() {
	strconvnotes.DoStrconvNotes()
}
```

```
//Output
2021/03/22 19:28:59 [10:string]->[10:int]
2021/03/22 19:28:59 [100:int]->[100:string]
2021/03/22 19:28:59 [true:string]->[true:bool]
2021/03/22 19:28:59 [100.0123:string]->[100.0123:float64]
2021/03/22 19:28:59 [100.0123:string]->[100.0123:float64]
2021/03/22 19:28:59 [-1000:string]->[-1000:int64]
2021/03/22 19:28:59 [-1000:string]->[-8:int64]
2021/03/22 19:28:59 [1_2:string]->[12:int64]
2021/03/22 19:28:59 [1004:string]->[1004:uint64]

```

### Format类函数

将指定类型格式化为string类型:FormatBool(),FormatFloat(),FormatInt(),FormatUnit()

#### `func strconv.FormatInt(i int64, base int) string`

> FormatInt returns the string representation of i in the given base, for 2 <= base <= 36. The result uses the lower-case letters 'a' to 'z' for digit values >= 10.

FormatInt将int64型以指定的基数转换并转成string型返回，2<=base<=36.当基数大于10时，将会使用a到z的小写字母表示数字

#### `func strconv.FormatUint(i uint64, base int) string`

> FormatUint returns the string representation of i in the given base, for 2 <= base <= 36. The result uses the lower-case letters 'a' to 'z' for digit values >= 10.

与FormatInt相同，仅适用于uint64类型

#### `func strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string`

> FormatFloat converts the floating-point number f to a string, according to the format fmt and precision prec. It rounds the result assuming that the original was obtained from a floating-point value of bitSize bits (32 for float32, 64 for float64).
>
> The format fmt is one of 'b' (-ddddp±ddd, a binary exponent), 'e' (-d.dddde±dd, a decimal exponent), 'E' (-d.ddddE±dd, a decimal exponent), 'f' (-ddd.dddd, no exponent), 'g' ('e' for large exponents, 'f' otherwise), 'G' ('E' for large exponents, 'f' otherwise), 'x' (-0xd.ddddp±ddd, a hexadecimal fraction and binary exponent), or 'X' (-0Xd.ddddP±ddd, a hexadecimal fraction and binary exponent).
>
> The precision prec controls the number of digits (excluding the exponent) printed by the 'e', 'E', 'f', 'g', 'G', 'x', and 'X' formats. For 'e', 'E', 'f', 'x', and 'X', it is the number of digits after the decimal point. For 'g' and 'G' it is the maximum number of significant digits (trailing zeros are removed). The special precision -1 uses the smallest number of digits necessary such that ParseFloat will return f exactly.

### 示例

```go
//type_to_string.go
package strconvnotes

import (
	"fmt"
	"strconv"
)

func TypeToString() {
	//String to type
	typeToString()
}

func typeToString() {
	// int to string
	var i1 int64 = -4000
	si1 := strconv.FormatInt(i1, 16)
	PrintConvertLog(i1, si1)
	//uint to string
	var i2 uint64 = 4000
	si2 := strconv.FormatUint(i2, 16)
	PrintConvertLog(i2, si2)
	//-------------------------------
	// float to string with format 'f'(no exponent)
	fv := 3.1415926535
	sf1 := strconv.FormatFloat(fv, 'f', 6, 64)
	PrintConvertLog(fv, sf1)
	//float to string with format 'b'(a binary exponent)
	sf2 := strconv.FormatFloat(fv, 'b', -1, 64)
	PrintConvertLog(fv, sf2)
	//float to string with foramt 'E'(a decimal exponent)
	sf3 := strconv.FormatFloat(fv, 'E', -1, 64)
	PrintConvertLog(fv, sf3)
	//-------------------------------
}
```

```
Output:
2021/03/26 16:06:46 [-4000:int64]->[-fa0:string]
2021/03/26 16:06:46 [4000:uint64]->[fa0:string]
2021/03/26 16:06:46 [3.1415926535:float64]->[3.141593:string]
2021/03/26 16:06:46 [3.1415926535:float64]->[7074237751826244p-51:string]
2021/03/26 16:06:46 [3.1415926535:float64]->[3.1415926535E+00:string]
```



## Append类函数

AppendType类型函数将指定Type转换成字符串之后放入slice中：AppendBool(),AppendFloat(),AppendInt(),AppendUint()

#### `func AppendInt(dst []byte, i int64, base int) []byte`

> AppendInt appends the string form of the integer i, as generated by FormatInt, to dst and returns the extended buffer.

### 示例

```go
//type_to_string.go
package strconvnotes

import (
	"fmt"
	"strconv"
)

func TypeToString() {
	//String to type
	typeToString()
}

func typeToString() {
	//Append Type
	s := []byte("Format Result:")
	//AppendInt
	s = strconv.AppendInt(s, -100, 10)
	s = append(s, ' ')
	//AppendBool
	s = strconv.AppendBool(s, true)
	s = append(s, ' ')
	//AppendUint
	s = strconv.AppendUint(s, 100, 16)
	s = append(s, ' ')
	//AppendFloat
	s = strconv.AppendFloat(s, fv, 'f', -1, 64)
	s = append(s, ' ')
	fmt.Println(string(s))
}

```

```
Output:
Format Result:-100 true 64 3.1415926535 
```

