# uniplaces/Carbon

# 简介

[uniplaces](https://github.com/uniplaces)/[carbon](https://github.com/uniplaces/carbon)是[Time](https://pkg.go.dev/time#Time)的简单扩展，基于PHP的[Carbon](http://carbon.nesbot.com/)库，特性：

- 内部集成[Time](https://golang.org/pkg/time/#Time)，可以使用所有Time的功能
- 支持时间运算
- 支持常用的日期格式
- 计算时间差异更简单

## 快速开始

使用`go get`安装

```sh
go get github.com/uniplaces/carbon@latest
```

或在`go module`中导入

```
require github.com/uniplaces/carbon latest
```

```go
package main

import (
	"fmt"
	"github.com/uniplaces/carbon"
	"time"
)

func main() {
	fmt.Printf("Right now is %s\n", carbon.Now().DateTimeString())

	today, _ := carbon.NowInLocation("Japan")
	fmt.Printf("Right now in Japan is %s\n", today)

	fmt.Printf("Tomorrow is %s\n", carbon.Now().AddDay())
	fmt.Printf("Last week is %s\n", carbon.Now().SubWeek())

	nextOlympics, _ := carbon.CreateFromDate(2016, time.August, 5, "Europe/London")
	nextOlympics = nextOlympics.AddYears(4)
	fmt.Printf("Next Olympics are in %d\n", nextOlympics.Year())

	if carbon.Now().IsWeekend() {
		fmt.Printf("Happy time")
	}
}

```

```sh
$ go run ./main.go 
Right now is 2021-11-04 10:18:59
Right now in Japan is 2021-11-04 11:18:59
Tomorrow is 2021-11-05 10:18:59
Last week is 2021-10-28 10:18:59
Next Olympics are in 2020
```

`carbon`使用很便捷，其完全兼容标准库的`time.Time`类型，可以直接调用`time.Time`的方法，因为`carbon.Carbon`直接将`time.Time`内嵌到结构体中:

```go
// carbon@v0.1.6/carbon.go
// The Carbon type represents a Time instance.
// Provides a simple API extension for Time.
type Carbon struct {
	time.Time
	weekStartsAt time.Weekday
	weekEndsAt   time.Weekday
	weekendDays  []time.Weekday
	stringFormat string
	Translator   *Translator
}
```

`carbon`简化了创建操作，`time`创建一个`Time`对象，若不为本地或UTC时区，需要自行调用`LoadLocation`加载对应时区，然后传给`time.Date`方法创建,`carbon`可以直接通过时区名称来创建

## 时区

引用自维基百科的定义：

>时区是地球上的区域使用同一个时间定义。以前，人们通过观察太阳的位置（时角）决定时间，这就使得不同经度的地方的时间有所不同（地方时）。1863年，首次使用时区的概念。时区通过设立一个区域的标准时间部分地解决了这个问题。 世界各国位于地球不同位置上，因此不同国家，特别是东西跨度大的国家日出、日落时间必定有所偏差。这些偏差就是所谓的时差。

例如，日本东京位于东九区，北京位于东八区，东九区时间比东八区快一个小时，日本时间11:00的时候中国时间为10:00

在linux中，时区文件一般存放在`/usr/share/zoneinfo`的目录中，时区文件为二进制文件，可以执行`info tzfile`查看说明

时区名称一般为`city`,`country/city`,`contitent/city`,例如`Asia/Shanghai`,`Asia/Hong_Kong`,也有特殊如UTC,Local等

Golang为了可移植性，集成了时区文件，在linux中放在`/usr/local/go/lib/time/zoneinfo.zip`中

使用`time`创建某个时区的时间，需要先加载时区，而使用`carbon`可以直接通过时区名称创建

```go
package main

import (
	"fmt"
	"github.com/uniplaces/carbon"
	"log"
	"time"
)

func main() {
	// creat date with time
	fmt.Println("Create date with time:")
	loc, err := time.LoadLocation("Japan")
	if err != nil {
		log.Fatal("failed to load location:", err)
	}
	d := time.Date(2021, time.November, 4, 10, 51, 20, 0, loc)
	fmt.Printf("time in japan is :%s\n", d)
	// create date with carbon
	fmt.Println("Create date with carbon:")
	c, err := carbon.Create(2021, time.November, 4, 10, 51, 20, 0, "Japan")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("time in japan is :%s\n", c)
}
```

## 时间运算

使用`time`进行时间运算需要预先定义`time.Duration`对象，`time`预定义的只有纳秒到小时的精度(为了避免夏时制[^1]带来的困扰)：

```go
// Common durations. There is no definition for units of Day or larger
// to avoid confusion across daylight savings time zone transitions.
//
// To count the number of units in a Duration, divide:
//	second := time.Second
//	fmt.Print(int64(second/time.Millisecond)) // prints 1000
//
// To convert an integer number of units to a Duration, multiply:
//	seconds := 10
//	fmt.Print(time.Duration(seconds)*time.Second) // prints 10s
//
const (
	Nanosecond  Duration = 1
	Microsecond          = 1000 * Nanosecond
	Millisecond          = 1000 * Microsecond
	Second               = 1000 * Millisecond
	Minute               = 60 * Second
	Hour                 = 60 * Minute
)
```

复杂时长需要使用`time.ParseDuration`构造，如"3m20s","1h20m"等。若想要增加`year/month/day`需要使用`time.Time`的`AddDate`方法

```go
package main

import (
	"fmt"
	"github.com/uniplaces/carbon"
	"log"
	"time"
)

func main() {
	outputFormat := "%-30s:%s\n"
	// calculate date with 'time'
	fmt.Println("Calculate date with 'time'")
	now := time.Now()

	fmt.Printf(outputFormat, "now", now)
	fmt.Printf(outputFormat, "one second later", now.Add(time.Second))
	fmt.Printf(outputFormat, "one minute later", now.Add(time.Minute))
	fmt.Printf(outputFormat, "one hour later", now.Add(time.Hour))

	dur, err := time.ParseDuration("3m20s")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf(outputFormat, "3 minutes and 20 seconds later", now.Add(dur))

	dur, err = time.ParseDuration("2h30m")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf(outputFormat, "2 hours and 30 minutes later", now.Add(dur))
	// call AddDate instead of Add
	fmt.Printf(outputFormat, "3 days and 2 hours later", now.AddDate(0, 0, 3).Add(time.Hour*2))

	// calculate date with 'carbon'
	fmt.Println("Calculate date with 'carbon'")
	cNow := carbon.Now()

	fmt.Printf(outputFormat, "now", cNow)
	fmt.Printf(outputFormat, "one second later", cNow.AddSecond())
	fmt.Printf(outputFormat, "one minute later", cNow.AddMinute())
	fmt.Printf(outputFormat, "one hour later", cNow.AddHour())
	fmt.Printf(outputFormat, "3 minutes and 20 seconds later", cNow.AddMinutes(3).AddSeconds(20))
	fmt.Printf(outputFormat, "2 hours and 30 minutes later", cNow.AddHours(2).AddMinutes(30))
	fmt.Printf(outputFormat, "3 days and 2 hours later", cNow.AddDays(3).AddHours(2))
}

```

```sh
$ go run ./main.go     
Calculate date with 'time'
now                           :2021-11-04 12:36:55.955753326 +0800 CST m=+0.000066627
one second later              :2021-11-04 12:36:56.955753326 +0800 CST m=+1.000066627
one minute later              :2021-11-04 12:37:55.955753326 +0800 CST m=+60.000066627
one hour later                :2021-11-04 13:36:55.955753326 +0800 CST m=+3600.000066627
3 minutes and 20 seconds later:2021-11-04 12:40:15.955753326 +0800 CST m=+200.000066627
2 hours and 30 minutes later  :2021-11-04 15:06:55.955753326 +0800 CST m=+9000.000066627
3 days and 2 hours later      :2021-11-07 14:36:55.955753326 +0800 CST
Calculate date with 'carbon'
now                           :2021-11-04 12:36:55
one second later              :2021-11-04 12:36:56
one minute later              :2021-11-04 12:37:55
one hour later                :2021-11-04 13:36:55
3 minutes and 20 seconds later:2021-11-04 12:40:15
2 hours and 30 minutes later  :2021-11-04 15:06:55
3 days and 2 hours later      :2021-11-07 14:36:55

```

值得注意的是`carbon`和`time`的时间操作会返回一个新的对象，原对象不会改变

除了上例出现的方法之外，`carbon`还提供了：

- `AddQuarters/AddQuarter`:增加季度
- `AddCenturies/AddCentury`：增加世纪
- `AddWeekdays/AddWeekday`:增加工作日，会跳过非工作日
- `AddWeeks/AddWeek`:增加星期

在`Add*`方法中传入负数值表示减少，也可以调用响应的`Sub*`方法

## 时间比较

`time.Time`可以使用方法`Before/After/Equal`进行时间的比较，`carbon`除了使用这些方法之外，提供了额外的多组方法，每组一个简短名一个详细名：

- `Eq/EqualTo`:是否相等
- `Ne/NotEqualTo`:是否不等
- `Gt/GreaterThan`:是否在指定时间之后
- `Lt/LessThan`:是否在指定时间之前
- `Lte/LessThanOrEqual`:是否在指定时间之前或相等
- `Between`:是否在指定的时间段内

此外，还提供了判断用的方法：

- `IsMongday/IsTuesday/.../IsSunday`:判断周几
- `IsWeekday/IsWeekend/IsLeapYear/IsPast/IsFuture`:判断

```go
// go-daily-lib-note/uniplace-carbon/diff-date/main.go
package main

import (
	"fmt"
	"github.com/uniplaces/carbon"
)

func main() {
	outputFormat := "%-30s:%t\n"

	date1, _ := carbon.CreateFromDate(2010, 1, 1, "Asia/Shanghai")
	date2, _ := carbon.CreateFromDate(2011, 2, 1, "Asia/Shanghai")
	date3, _ := carbon.CreateFromDate(2010, 12, 1, "Asia/Shanghai")
	fmt.Println("date1:", date1)
	fmt.Println("date1:", date2)
	fmt.Println("date1:", date3)
	fmt.Printf(outputFormat, "date1 equal to  date2", date1.Eq(date2))
	fmt.Printf(outputFormat, "date1 not equal to date2", date1.Ne(date2))

	fmt.Printf(outputFormat, "date1 greater than date2", date1.Gt(date2))
	fmt.Printf(outputFormat, "date1 less than date2", date1.Lt(date2))

	fmt.Printf(outputFormat, "date3 between date1 and date2", date3.Between(date1, date2, true))

	now := carbon.Now()
	fmt.Printf("%-30s:%s\n", "now", now)
	fmt.Printf(outputFormat, "is weekday", now.IsWeekday())
	fmt.Printf(outputFormat, "is weekend", now.IsWeekend())
	fmt.Printf(outputFormat, "is leap year", now.IsLeapYear())
	fmt.Printf(outputFormat, "is past", now.IsPast())
	fmt.Printf(outputFormat, "is future", now.IsFuture())
}

```

```sh
$ go run ./main.go
date1: 2010-01-01 09:32:41
date1: 2011-02-01 09:32:41
date1: 2010-12-01 09:32:41
date1 equal to  date2         :false
date1 not equal to date2      :true
date1 greater than date2      :false
date1 less than date2         :true
date3 between date1 and date2 :true
now                           :2021-11-05 09:32:41
is weekday                    :true
is weekend                    :false
is leap year                  :false
is past                       :true
is future                     :false
```

还可以使用`carbon`计算两个`date`之间相差多少秒，分，时，天等

```go
// go-daily-lib-note/uniplace-carbon/calculate-diff-date/main.go
package main

import (
	"fmt"

	"github.com/uniplaces/carbon"
)

func main() {
	date1, _ := carbon.CreateFromDate(2021, 1, 1, "Asia/Tokyo")
	date2, _ := carbon.CreateFromDate(2022, 1, 1, "Asia/Tokyo")

	fmt.Println(date1.DiffInYears(date2, false))   // 1
	fmt.Println(date1.DiffInMonths(date2, false))  // 12
	fmt.Println(date1.DiffInDays(date2, false))    // 365
	fmt.Println(date1.DiffInHours(date2, false))   // 8760
	fmt.Println(date1.DiffInMinutes(date2, false)) // 525600
	fmt.Println(date1.DiffInSeconds(date2, false)) // 3153600

}
```

## 格式化

 `time.Time`的`Format`方法进行格式格式化是需要传入指定的字符串`2006-01-02 15:04:05.000`,这里的时间格式是固定的，可以按照`0(ms)1(M)2(d)3(h)4(m)5(s)6(y)`的方式记忆

```go
// go-daily-lib-note/uniplace-carbon/date-format/main.go
package main

import (
	"fmt"
	"time"
)

func main() {
	now := time.Now()
	fmt.Println(now.Format("2006-01-02 15:04:05.000")) // 2021-11-06 03:57:13.845
	fmt.Println(now.Format("2006/01/02 15/04/05.000")) // 2021/11/06 03/57/13
	fmt.Println(now.Format("15:04:05.000"))            // 03:57:13.845
}
```

Go也内置一些标准时间格式：

```go
// time/format.go
const (
	Layout      = "01/02 03:04:05PM '06 -0700" // The reference time, in numerical order.
	ANSIC       = "Mon Jan _2 15:04:05 2006"
	UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
	RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
	RFC822      = "02 Jan 06 15:04 MST"
	RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
	RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
	RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
	RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
	RFC3339     = "2006-01-02T15:04:05Z07:00"
	RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
	Kitchen     = "3:04PM"
	// Handy time stamps.
	Stamp      = "Jan _2 15:04:05"
	StampMilli = "Jan _2 15:04:05.000"
	StampMicro = "Jan _2 15:04:05.000000"
	StampNano  = "Jan _2 15:04:05.000000000"
)
```

`uniplace/carbon`在上述格式之外提供了额外的格式：

```go
// src/github.com/uniplaces/carbon
const (
  DefaultFormat       = "2006-01-02 15:04:05"
  DateFormat          = "2006-01-02"
  FormattedDateFormat = "Jan 2, 2006"
  TimeFormat          = "15:04:05"
  HourMinuteFormat    = "15:04"
  HourFormat          = "15"
  DayDateTimeFormat   = "Mon, Aug 2, 2006 3:04 PM"
  CookieFormat        = "Monday, 02-Jan-2006 15:04:05 MST"
  RFC822Format        = "Mon, 02 Jan 06 15:04:05 -0700"
  RFC1036Format       = "Mon, 02 Jan 06 15:04:05 -0700"
  RFC2822Format       = "Mon, 02 Jan 2006 15:04:05 -0700"
  RSSFormat           = "Mon, 02 Jan 2006 15:04:05 -0700"
)
```

需要注意的是`time`库默认使用`2006-01-02 15:04:05.999999999 -0700 MST`格式，而`uniplace/carbon`默认使用更简洁的`2006-01-02 15:04:05`

## 高级特性

### 修饰符（modifier）

修饰符(modifier)为针对一些特定的时间操作，如开始和结束的时间，下个周一，下个工作日等

```go
// go-daily-lib-note/date-modifier/main.go
package main

import (
	"fmt"
	"time"

	"github.com/uniplaces/carbon"
)

func main() {
	outputFormat := "%-20s: %s\n"
	now := carbon.Now()

	fmt.Printf(outputFormat, "Start of day", now.StartOfDay())
	fmt.Printf(outputFormat, "End of day", now.EndOfDay())
	fmt.Printf(outputFormat, "Start of month", now.StartOfMonth())
	fmt.Printf(outputFormat, "End of month", now.EndOfMonth())
	fmt.Printf(outputFormat, "Start of year", now.StartOfYear())
	fmt.Printf(outputFormat, "Start of decade", now.StartOfDecade())
	fmt.Printf(outputFormat, "End of decade", now.EndOfDecade())
	fmt.Printf(outputFormat, "Start of century", now.StartOfCentury())
	fmt.Printf(outputFormat, "End of century", now.EndOfCentury())
	fmt.Printf(outputFormat, "Start of week", now.StartOfWeek())
	fmt.Printf(outputFormat, "End of week", now.EndOfWeek())
	fmt.Printf(outputFormat, "Next Wednesday", now.Next(time.Wednesday))
	fmt.Printf(outputFormat, "Previous Wednesday", now.Previous(time.Wednesday))

}
```

```sh
$ go run ./main.go
Start of day        : 2021-11-06 00:00:00
End of day          : 2021-11-06 23:59:59
Start of month      : 2021-11-01 00:00:00
End of month        : 2021-11-30 23:59:59
Start of year       : 2021-01-01 00:00:00
Start of decade     : 2020-01-01 00:00:00
End of decade       : 2029-12-31 23:59:59
Start of century    : 2000-01-01 00:00:00
End of century      : 2099-12-31 23:59:59
Start of week       : 2021-11-01 00:00:00
End of week         : 2021-11-07 23:59:59
Next Wednesday      : 2021-11-10 00:00:00
Previous Wednesday  : 2021-11-03 00:00:00
```

### 自定义工作日和周末

不同的地区每周开始会有所不同，`uniplace/carbon`可以自定义每周的开始和周末：

```go
// go-daily-lib-note/uniplace-carbon/custom-weekend/main.go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/uniplaces/carbon"
)

func main() {
	date, err := carbon.Create(2021, 11, 6, 0, 0, 0, 0, "Asia/Shanghai")
	if err != nil {
		log.Fatal(err)
	}
	date.SetWeekStartsAt(time.Sunday)
	date.SetWeekEndsAt(time.Saturday)
	date.SetWeekendDays([]time.Weekday{time.Monday, time.Tuesday, time.Thursday, time.Friday})

	fmt.Printf("Today is %s,weekend? %t\n", date.Weekday(), date.IsWeekend())
}

```

## 总结

时间的处理是个复杂的问题，需要考虑诸如时区，夏令时，闰秒，闰年等问题，使用第三方时间处理库能够很大提升开发效率

## 参考

1. [uniplaces](https://github.com/uniplaces)/[carbon](https://github.com/uniplaces/carbon) github repo
2. [Go 每日一库之 carbon](https://darjun.github.io/2020/02/14/godailylib/carbon/) darjun blog
3. [时区](https://zh.wikipedia.org/wiki/%E6%97%B6%E5%8C%BA) 维基百科
4. [time](https://pkg.go.dev/time) time godocs



[^1]:[**夏时制**](https://zh.wikipedia.org/wiki/%E5%A4%8F%E6%97%B6%E5%88%B6)（美国及加拿大英语：daylight time），又称**夏令时**、**日光节约时间**（美国及加拿大称为daylight saving time，简称DST；英国与其他地区称为Summer Time），是一种在[夏季](https://zh.wikipedia.org/wiki/夏季)月份牺牲正常的日出时间，而将时间调快的做法

