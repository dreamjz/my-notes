# Javascript

## 简介

**JavaScript (** **JS** ) 是一种具有[函数优先](https://developer.mozilla.org/zh-CN/docs/Glossary/First-class_Function)的轻量级，解释型或即时编译型的编程语言。虽然它是作为开发Web 页面的脚本语言而出名的，但是它也被用到了很多[非浏览器环境](https://en.wikipedia.org/wiki/JavaScript#Uses_outside_Web_pages)中，例如 [Node.js](https://nodejs.org/)、 [Apache CouchDB](https://couchdb.apache.org/) 和 [Adobe Acrobat](https://www.adobe.com/devnet/acrobat/javascript.html)。JavaScript 是一种[基于原型编程](https://developer.mozilla.org/zh-CN/docs/Glossary/Prototype-based_programming)、多范式的动态脚本语言，并且支持面向对象、命令式和声明式（如函数式编程）风格

### 组成

JavaScript基础分为三个部分：

- **ECMAScript**：JavaScript 的**语法标准**。包括变量、表达式、运算符、函数、if语句、for语句等。
- **DOM**：Document Object Model（文档对象模型），操作**页面上的元素**的API。比如让盒子移动、变色、改变大小、轮播图等等。
- **BOM**：Browser Object Model（浏览器对象模型），操作**浏览器部分功能**的API。通过BOM可以操作浏览器窗口，比如弹框、控制浏览器跳转、获取浏览器分辨率等等。

通俗理解就是：ECMAScript 是 JS 的语法；DOM 和 BOM 浏览器运行环境为 JS提供的API

## 引入 Js

### 行内式

```html
<input type="button" value="click here" onclick="alert('Message')" />
```

- 可以将单行或少量 JS 代码写在HTML标签的事件属性中（以 on 开头的属性），比如放在上面的 `onclick`点击事件中
- 这种书写方式，不推荐使用，原因是：可读性差，尤其是需要编写大量 JS代码时，容易出错；引号多层嵌套时，也容易出错

### 内嵌式

在`<script>`标签中编写 js 代码 

```html
<script>
	console.log("js code in script element")
</script>
```

### 引入外部 JS 文件

```html
<script src="xxx.js"></script>
```

可以确保 html 文件和 js 文件是分开的，有利于代码的结构化和复用 

## 变量

### 字面量

即常量，不可改变，有三种：

- 数字
- 字符串
- 布尔字面量

```js
console.log(100) // Number 
console.log('String') // String
console.log(true) // Boolean
```

### 声明

ES6 之前统一使用`var`:

```js
var name;
```

ES6 可以使用`const`声明常量，`let`声明变量

```js
const name; // Constant
let age;
```

### 赋值

使用`=`进行赋值

```js
var name ;
name = 'MyName';
```

### 初始化

同时进行声明和赋值，称为变量的初始化

```js
var age = 18;
```

可以不声明直接赋值，会出现变量提升问题

```js
age = 18;
console.log(age)
```

## 标识符

在JS中所有的可以由我们**自主命名**的都可以称之为标识符。

例如：变量名、函数名、属性名、参数名都是属于标识符。通俗来讲，标识符就是我们写代码时为它们起的名字。

**标识符的命名规则**和变量的命令规则是一样的。看上面一段就可以了。

同样，标识符不能使用语言中保留的**关键字**及**保留字**

## 关键字

是指 JS 本身已经使用了的单词，我们不能再用它们充当变量、函数名等标识符。

JS 中的关键字如下：

```text
break、continue、case、default、

if、else、switch、for、in、do、while、

try、catch、finally、throw、

var、void、function、return、new、

this、typeof、instanceof、delete、with、

true、false、null、undefined
```

## 保留字

实际上就是预留的“关键字”。意思是现在虽然还不是关键字，但是未来可能会成为关键字，同样不 能使用它们当充当变量名、函数名等标识符。

JS 中的保留字如下：

```text
abstract、boolean、byte、char、class、const、

debugger、double、enum、export、extends、final、float、goto

implements、import、int、interface、long、native、package、

private、protected、public、short、static、super、synchronized、throws、

transient、volatile
```

## 基本数据类型

JavaScript 是一种「弱类型语言」，或者说是一种「动态语言」，这意味着不需要提前声明变量的类型，在程序运行过程中，类型会自动被确定

**JS 的变量数据类型，是在程序运行的过程中，根据等号右边的值来确定的**。而且，变量的数据类型是可以变化的

基本数据类型有5种：

- String
- Number
- Boolean
- Null
- Undefined

### String

字符串型可以是引号中的任意文本，其语法为：双引号 `""` 或者单引号 `''`

```js
var a = "Aa" 
var b = 'Bb'
console.log(a) // Aa
console.log(b) // Bb
```

注意：

- 引号必须成对出现
- 相同引号不可嵌套

#### 转义字符

在字符串中通过`\`来使用转义字符，如：`\n`,`\t`等

#### 长度

使用`String`的属性`length`获取字符串的长度，其中：

- 一个中文算一个字符，一个英文算一个字符
- 一个标点符号（包括中文标点、英文标点）算一个字符
- 一个空格算一个字

```js
var str1 = '字符串';
var str2 = 'js, 字符串';

console.log(str1.length); // 3
console.log(str2.length); // 7
```

#### 拼接

字符串之间使用`+`拼接，字符串和其他数据类型也可以拼接，拼接前，会把与字符串相加的这个数据类型转成字符串，然后再拼接成一个新的字符串

```js
var str3 = 'string '
console.log(str3+'String') // string String
console.log(str3+123) // string 123
console.log(str3+true) // string true
console.log(str3+null) // string null
console.log(str3+undefined) // string undefined
console.log(str3+{"a":"A"}) // string [object Object]
```

#### 不可变

字符串里面的值不可被改变。虽然看上去可以改变内容，但其实是地址变了，内存中新开辟了一个内存空间

#### 模板字面量

ES6中引入了**模板字符串**,可以更方便进行字符串的操作

**插入变量**

```js
var name = 'name'
var age = 100
console.log('Name: '+name+',age: '+age) // Name: name,age: 100
console.log(`Name: ${name},age: ${age}`) // Name: name,age: 100
```

**插入表达式**

```js
var a = 10
var b = 15
console.log('a+b is: ' + (a + b)) // a+b is: 25
console.log(`a+b is: ${a+b}`) // a+b is: 25
```

**换行**

模板字符串内部可以换行

```js
console.log(`line 1
line2`)
```

**函数调用**

模板字符串中可以进行函数调用

```js
console.log(`Name is: ${getName()}`); // Name is: my name
function getName() {
    return 'my name';
}
```

### Boolean

布尔类型只有两个值：

- true
- false

布尔值和数字相加时，true 为 1，false 为 0

### Number

js 中所有数字均为 Number 类型，包括整数和浮点数

```js
var a = 10
var b = 1.12
console.log(typeof a) // number
console.log(typeof b) // number
```

#### 取值范围

- 最大值：`Number.MAX_VALUE`，这个值为： 1.7976931348623157e+308
- 最小值：`Number.MIN_VALUE`，这个值为： 5e-324

如果使用 Number 表示的变量超过了最大值，则会返回Infinity。

- 无穷大（正无穷）：Infinity
- 无穷小（负无穷）：-Infinity

注意：`typeof Infinity`的返回结果是number

#### NaN

**NaN**是一个特殊的数字，表示Not a Number，非数值

```js
console.log('abc' / 100) // NaN
console.log('a'*'b') // NaN
```

注意：`typeof NaN`的返回结果是 number。

Undefined和任何数值计算的结果为 NaN。NaN 与任何值都不相等，包括 NaN 本身

```js
console.log(NaN===NaN) // false
```

#### 隐式转换

`-`、`*`、`/`、`%`这几个符号会自动进行隐式转换

```js
console.log('4'+3-100) // -57(43-100)
```

#### 运算精度

在JS中，整数的运算**基本**可以保证精确；但是**小数的运算，可能会得到一个不精确的结果**。所以，千万不要使用JS进行对精确度要求比较高的运算

```js
console.log(0.1+0.2) // 0.30000000000000004
```

计算机在做运算时，所有的运算都要转换成二进制去计算。然而，有些数字转换成二进制之后，无法精确表示。比如说，0.1和0.2转换成二进制之后，是无穷的，因此存在浮点数的计算不精确的问题

如果只是一些简单的精度问题，可以使用 `toFix()` 方法进行小数的截取

关于浮点数计算的精度问题，往往比较复杂。市面上有很多针对数学运算的开源库，比如[decimal.js](https://github.com/MikeMcl/decimal.js/)、 [Math.js](https://github.com/josdejong/mathjs)

- Math.js：属于很全面的运算库，文件很大，压缩后的文件就有500kb。如果你的项目涉及到大型的复杂运算，可以使用 Math.js。
- decimal.js：属于轻量的运算库，压缩后的文件只有32kb。大多数项目的数学运算，使用 decimal.js 足够了。

### Null

null 专门用来定义一个**空对象**

```js
console.log(typeof null) // object
```

### Underfined

- **声明**了一个变量，但没有**赋值**，此时它的值就是 `undefined`
- 如果你从未声明一个变量，就去使用它，则会报错（这个大家都知道）；此时，如果用 `typeof` 检查这个变量时，会返回 `undefined`
- 如果一个函数没有返回值，那么，这个函数的返回值就是 undefined
- 调用函数时，如果没有传参，那么，这个参数的值就是 undefined

#### Underfined 和 Null

null 和 undefined 有很大的相似性。看看 `null == undefined` 的结果为 `true` 也更加能说明这点

但是 `null === undefined` 的结果是 false。它们虽然相似，但还是有区别的，其中一个区别是，和数字运算时：

- 10 + null 结果为 10。
- 10 + undefined 结果为 NaN。

规律总结：

- 任何数据类型和 undefined 运算都是 NaN;
- 任何值和 null 运算，null 可看做 0 运算

## 引用数据类型

只有一种即 Object ,内置的对象 Function、Array、Date、RegExp、Error等都是属于 Object 类型

## 类型转换 TODO



## 运算符

JS 中的运算符，分类如下：

- 算数运算符
- 自增/自减运算符
- 一元运算符
- 逻辑运算符
- 赋值运算符
- 比较运算符
- 三元运算符（条件运算符）

###  算数运算符

| 运算符 |          描述          |
| :----- | :--------------------: |
| +      |     加、字符串连接     |
| -      |           减           |
| *      |           乘           |
| /      |           除           |
| %      | 获取余数（取余、取模） |

#### 运算规则

- 先算乘除、后算加减

- 小括号`( )`：能够影响计算顺序，且可以嵌套。没有中括号、没有大括号，只有小括号

- 百分号：取余，只关心余数

#### 浮点数运算的精度问题

浮点数值的最高精度是 17 位小数，但在进行算术计算时，会丢失精度，导致计算不够准确。比如：

```javascript
console.log(0.1 + 0.2); // 运算结果不是 0.3，而是 0.30000000000000004

console.log(0.07 * 100); // 运算结果不是 7，而是 7.000000000000001
```

因此，**不要直接判断两个浮点数是否相等**

### 自增 自减

自增分成两种：`a++`和`++a`。

- 一个变量自增以后，原变量的值会**立即**自增1。也就是说，无论是 `a++` 还是`++a`，都会立即使原变量的值自增1。

- **我们要注意的是**：`a`是变量，而`a++`和`++a`是**表达式**。

那这两种自增，有啥区别呢？区别是：`a++` 和 `++a`的值不同：（也就是说，表达式的值不同）

- `a++`这个表达式的值等于原变量的值（a自增前的值）。你可以这样理解：先把 a 的值赋值给表达式，然后 a 再自增。
- `++a`这个表达式的值等于新值 （a自增后的值）。 你可以这样理解：a 先自增，然后再把自增后的值赋值给表达式

自减和自增是类似的

### 一元运算符

一元运算符，只需要一个操作数

####  typeof

typeof就是典型的一元运算符，因为后面只跟一个操作数。

```javascript
var a = '123';
console.log(typeof a); // 打印结果：string
```

#### 正号 `+`

- 正号不会对数字产生任何影响。比如说，`2`和`+2`是一样的。

- 我们可以对一个其他的数据类型使用`+`，来将其转换为number

#### 负号 `-`

负号可以对数字进行取反

### 逻辑运算符

逻辑运算符有三个：

- `&&` 与（且）：两个都为真，结果才为真。
- `||` 或：只要有一个是真，结果就是真。
- `!` 非：对一个布尔值进行取反。

#### 非布尔值的与或运算

非布尔值进行**与或运算**时，会先将其转换为布尔值，然后再运算，但返回结果是**原值**。比如说：

```javascript
var result = 5 && 6; // 运算过程：true && true;
console.log('result：' + result); // 打印结果：6（也就是说最后面的那个值。）
```

上方代码可以看到，虽然运算过程为布尔值的运算，但返回结果是原值。

那么，返回结果是哪个原值呢？我们来看一下。

**与运算**的返回结果：（以多个非布尔值的运算为例）

- 如果第一个值为false，则执行第一条语句，并直接返回第一个值；不会再往后执行。
- 如果第一个值为true，则继续执行第二条语句，并返回第二个值（如果所有的值都为true，则返回的是最后一个值）。

**或运算**的返回结果：（以多个非布尔值的运算为例）

- 如果第一个值为true，则执行第一条语句，并直接返回第一个值；不会再往后执行。
- 如果第一个值为false，则继续执行第二条语句，并返回第二个值（（如果所有的值都为false，则返回的是最后一个值）。

实际开发中，我们经常是这样来做「容错处理」的：

当前端成功调用一个接口后，返回的数据为 result 对象。这个时候，我们用变量 a 来接收 result 里的图片资源。通常的写法是这样的：

```javascript
if (result.resultCode == 0) {
	var a = result && result.data && result.data.imgUrl || 'http://xxx/20160401_01.jpg';
}
```

上方代码的意思是，获取返回结果中的`result.data.imgUrl`这个图片资源；如果返回结果中没有 `result.data.imgUrl` 这个字段，就用 `http://xxx/20160401_01.jpg` 作为**兜底**图片。这种写法，在实际开发中经常用到。

#### 非布尔值的 `!` 运算

非布尔值进行**非运算**时，会先将其转换为布尔值，然后再运算，但返回结果是**布尔值**。

```javascript
let a = 10;
a = !a

console.log(a);  // false
console.log(typeof a); // boolean
```

###  比较运算符

比较运算符可以比较两个值之间的大小关系，如果关系成立它会返回true，如果关系不成立则返回false。

比较运算符有很多种，比如：

```text
>	大于号
<	小于号
>= 	大于或等于
<=  小于或等于
== 	等于
=== 全等于
!=	不等于
!== 不全等于
```

**比较运算符，得到的结果都是布尔值：要么是true，要么是false**

#### 数值的比较

（1）对于非数值进行比较时，会将其转换为数字然后再比较。

```javascript
console.log(1 > true); //false
console.log(1 >= true); //true
console.log(1 > "0"); //true

//console.log(10 > null); //true

//任何值和NaN做任何比较都是false

console.log(10 <= "hello"); //false
console.log(true > false); //true
```

（2）特殊情况：如果符号两侧的值都是字符串时，**不会**将其转换为数字进行比较。比较两个字符串时，比较的是字符串的**Unicode编码**。【非常重要，这里是个大坑，很容易踩到】

比较字符编码时，是一位一位进行比较。如果两位一样，则比较下一位

```javascript
// 比较两个字符串时，比较的是字符串的字符编码，所以可能会得到不可预期的结果
console.log("56" > "123");  // true
```



**因此**：当我们在比较两个字符串型的数字时，**一定一定要先转型**再比较大小，比如 `parseInt()`。

（3）任何值和NaN做任何比较都是false。

### 全等符号的强调

**全等在比较时，不会做类型转换**。如果要保证**绝对等于（完全等于）**，我们就要用三个等号`===`。例如：

```javascript
	console.log("6" === 6);		//false
	console.log(6 === 6);		//true
```

上述内容分析出：

- `==`两个等号，不严谨，"6"和6是true。
- `===`三个等号，严谨，"6"和6是false。

另外还有：**`==`的反面是`!=`，`===`的反面是`!==`**。例如：

```javascript
	console.log(3 != 8);	//true
	console.log(3 != "3");	//false，因为3=="3"是true，所以反过来就是false。
	console.log(3 !== "3");	//true，应为3==="3"是false，所以反过来是true。
```

### 运算符的优先级

运算符的优先级如下：（优先级从高到低）

- `.`、`[]`、`new`
- `()`
- `++`、`--`
- `!`、`~`、`+`（单目）、`-`（单目）、`typeof`、`void`、`delete`
- `%`、`*`、`/`
- `+`（双目）、`-`（双目）
- `<<`、`>>`、`>>>`
- 关系运算符：`<`、`<=`、`>`、`>=`
- `==`、`!==`、`===`、`!==`
- `&`
- `^`
- `|`
- `&&`
- `||`
- `?:`
- `=`、`+=`、`-=`、`*=`、`/=`、`%=`、`<<=`、`>>=`、`>>>=`、`&=`、`^=`、`|=`
- `,`

注意：逻辑与 `&&` 比逻辑或 `||` 的优先级更高。

备注：你在实际写代码的时候，如果不清楚哪个优先级更高，可以把括号运用上



## 参考

1. [MDN Web Docs](https://developer.mozilla.org/zh-CN/) 
2. 



