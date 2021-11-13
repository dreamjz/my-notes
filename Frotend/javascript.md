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

#### 字符串长度

使用`String`的属性`length`获取字符串的长度，其中：

- 一个中文算一个字符，一个英文算一个字符
- 一个标点符号（包括中文标点、英文标点）算一个字符
- 一个空格算一个字

```js

```



## 引用数据类型

只有一种即 Object ,内置的对象 Function、Array、Date、RegExp、Error等都是属于 Object 类型

## 参考

1. [MDN Web Docs](https://developer.mozilla.org/zh-CN/) 