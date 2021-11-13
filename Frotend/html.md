# HTML

## 简介

HTML(**H**yper **T**ext **M**arkup **L**anguage):超文本标记语言，负责描述文档语义的语言

### 超文本

所谓超文本有两层含义：

- 图片，音频，视频，动画，多媒体等超出了文本限制
- 可以从一个文件链接到另一个文件，与世界各地的文件连接，即：超链接文本

### 标记语言

HTML 不是编程语言，是描述性的标记语言，主要有两层含义：

- 标记语言是一套标记标签，如`<a>`,`<img>`,`<h1>`表示超链接，图片，标题等
- 标记语言无需编译，直接由浏览器解析执行

### 发展历史

![20151001_1001](.//image/20151001_1001.png)

**XHTML：** XHTML：Extensible Hypertext Markup Language，可扩展超文本标注语言。 XHTML的主要目的是为了**取代HTML**，也可以理解为HTML的升级版。 HTML的标记书写很不规范，会造成其它的设备(ipad、手机、电视等)无法正常显示。 XHTML与HTML4.0的标记基本上一样。 XHTML是**严格的、纯净的**HTML。

### 名词

- 网页 ：由各种标记组成的一个页面就叫网页。
- 主页(首页) : 一个网站的起始页面或者导航页面。
- 标记： 比如`<p>`称为开始标记 ，`</p>`称为结束标记，也叫标签。每个标签都规定好了特殊的含义。
- 元素：比如`<p>内容</p>`称为元素.
- 属性：给每一个标签所做的辅助信息。
- XHTML：符合XML语法标准的HTML。
- DHTML：dynamic，动态的。`javascript + css + html`合起来的页面就是一个 DHTML。
- HTTP：超文本传输协议。用来规定客户端浏览器和服务端交互时数据的一个格式。SMTP：邮件传输协议，FTP：文件传输协议。

## 编写HTML

#### Hello World

在 vscode 中新增`hello_world.html`，输入`html:5`生成 html 内容骨架，在 `body`中写入：

```html
<!-- HTML/hello_world.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Hello World</h1>
</body>
</html>
```

用浏览器打开即可看到标题`Hello World`

## 基本结构

HTML 标签有：

- 双边标记 ： 成对出现，如 `<div></div>`,`<a></a>`等
- 单边标记 ： 如`<br />`,`<hr />`等

属性和标记之间，属性之间，使用空格隔开；属性值在双引号内

HTML 骨架标签：

| 标签            |        说明     |
| :--------------| -------- |
| `<html></html>` |根标签   |
| `<head></head>` |文档头部 |
| `<body></body>` |文档主体 |

### 文档声明

标准的 HTML 页面，第一行为 `<!DOCTYPE ...>`,即 文档声明(DocType Declaration),简称 DTD

DTD 可告知浏览器使用哪种 HTML 或 XHTML 规范

#### 规范

HTML4.01里面规定了**普通**和**XHTML**两大种规范，大规范下有`Strict`,`Traditional`,`Frameset`三种小规范

- **strict**：这种模式里面的要求更为严格。这种严格体现在哪里？有一些标签不能使用。 比如，u标签，就是给一个本文加下划线，但是这和HTML的本质有冲突，因为HTML最好是只负责语义，不要负责样式，而u这个下划线是样式。所以，在strict中是不能使用u标签的。XHTML1.0更为严格，因为这个体系本身规定比如标签必须是小写字母、必须严格闭合标签、必须使用引号引起属性等等。

- **Transitional**：这种模式就是没有一些别的规范。

- **Frameset**：在框架的页面使用。

在 HTML 5 中极大的简化了 DTD，只有一种形式

```html
<!DOCTYPE html>
```

- 所有标记元素都要正确的嵌套，不能交叉嵌套。正确写法举例：`<h1><font></font></h1>`

- 所有的标记都必须小写。

- 所有的标签都必须闭合。
  - 双标签：`<span></span>`
  - 单标签：`<br>` 建议写成 `<br />` `<hr>` 建议转成 `<hr />`，还有`<img src=“URL” />`

- 所有的属性值必须加引号。`<font color="red"></font>`

- 所有的属性必须有值。`<hr noshade="noshade">`、`<input type="radio" checked="checked" />`

- XHTML文档开头必须要有DTD文档类型定义

### html

`html`标签的`lang`属性用于指定页面的语言，如`<html lang="en">`

### head

head 标签内部常见的标签如下：

- title : 指定网页标题
- base : 为页面上的链接指定默认地址
- meta : 提供页面基本信息
- link : 定义了文档与外部资源之间的关系
- style : 定义了HTML文档的样式文件引用地址
- script : 标签用于加载脚本文件

常见编码：

**ASCII码：** 用1个字节(8位二进制)来表示一个字符，共可以表示256个字符。 美国的国家语言是英语，只要能表示0-9、a-z、A-Z、特殊符号。

**ANSI编码：** **每个国家为了显示本国的语言，都对ASCII码进行了扩展**。用2个字节(16位二进制)来表示一个汉字，共可以表示2^16＝65536个汉字。例如： 中国的ANSI编码是GB2312编码(简体)，对6763汉字进行编码，含600多特殊字符。另外还有GBK(简体)。 日本的ANSI编码是JIS编码。 台湾的ANSI编码是BIG5编码

**GBK：** 对GB2312进行了扩展，用来显示罕见的、古汉语的汉字。现在已经收录了2.1万左右。并提供了1890个汉字码位。K的含义就是“扩展”。

**Unicode编码(统一编码)：** 用4个字节(32位二进制)来表示一个字符，想法不错，但效率太低。例如，字母A用ASCII表示的话一个字节就够，可用Unicode编码的话，得用4个字节表示，造成了空间的极大浪费

**UTF-8(Unicode Transform Format)编码：** 根据字符的不同，选择其编码的长度。比如：一个字符A用1个字节表示，一个汉字用2个字节表示。

#### meta

meta 常见属性：

- charset : 定义文档的字符编码
- content : 定义与 http-equiv 或 name 属性相关的元信息
- http-equiv : 把 content 属性关联到 HTTP 头部
- name : content 属性关联到一个名称
  - application-name : 规定页面所代表的 Web 应用程序的名称
  - author : 规定文档的作者的名字
  - description : 规定页面的描述,搜索引擎会把这个描述显示在搜索结果中
  - generator : 规定用于生成文档的一个软件包（不用于手写页面）
  - keywords : 规定一个逗号分隔的关键词列表 - 相关的网页（告诉搜索引擎页面是与什么相关的）

### body

body 的常见属性：

- `bgcolor`：设置整个网页的背景颜色。
- `background`：设置整个网页的背景图片。
- `text`：设置网页中的文本颜色。
- `leftmargin`：网页的左边距。IE浏览器默认是8个像素。
- `topmargin`：网页的上边距。
- `rightmargin`：网页的右边距。
- `bottommargin`：网页的下边距。
- `link`：表示默认显示的颜色
- `alink`：表示鼠标点击但是还没有松开时的颜色
- `vlink`：表示点击完成之后显示的颜色

## 排版标签

排版标签主要有：

- h1-h6 ： 标题，使用`<h1>`到`<h6>`定义，h1 字体最大，大小依次变小
- p ： 段落，可以把 HTML 文档分割为若干段落
- hr ： 水平线， 水平分隔线（horizontal rule）可以在视觉上将文档分隔成各个部分
- br ： 换行，文本强制换行
- div ： 可以把标签中的内容分割为独立的区块，必须单独占据一行
  - div在浏览器中，默认是不会增加任何的效果的，但是语义变了，div中的所有元素是一个小区域
  -  div标签是一个**容器级**标签，里面什么都能放，甚至可以放div自己
- span ： 和div的作用一致，但不换行
  - span也是表达“小区域、小跨度”的标签
  - 是一个**文本级**的标签。 就是说，span里面只能放置文字、图片、表单元素。 span里面不能放p、h、ul、dl、ol、div
- pre : 将保留标签内部所有的空白字符(空格、换行符)，原封不动地输出结果（告诉浏览器不要忽略空格和空行）

## 超链接

### 链接外部文件

```html
<a href="another.html">点击进入另外一个文件</a>
<a href="http://www.baidu.com" target="_blank">baidu</a>
```

### 锚链接

给超链接起一个名字，作用是**在本页面或者其他页面的的不同位置进行跳转**,定义在`name`和`id`属性中

说明：name属性是HTML4.0以前使用的，id属性是HTML4.0后才开始使用。为了向前兼容，因此，name和id这两个属性都要写上，并且值是一样的

跳转至锚点时使用`#name/id`的形式

```html
<a id="top" name="top">顶部</a>
<a href="#top">回到顶部</a>
```

### 常见属性

- `href`：目标URL
- `title`：悬停文本
- `name`：主要用于设置一个锚点的名称
- target : 告诉浏览器用什么方式来打开目标页面。属性有以下几个值：
  - `_self`：在同一个网页中显示（默认值）
  - `_blank`：**在新的窗口中打开**。
  - `_parent`：在父窗口中显示
  - `_top`：在顶级窗口中显示

## 图片标签

img 用于显示图片，是一个单标签：

```html
<img src="url" />
```

**src**

src(source): 用于指定图片的URL，可以使用绝对路径和相对路径

```html
<!--相对路径-->
<img src="./image1.png" />
<!--绝对路径-->
<img src="https://xxx/xxx.png" />
```

**alt**

当图片不可用（无法显示）的时候，代替图片显示的内容

## 列表标签

**ul**

ul(unordered list)，无序列表，列表项用 li(list item)表示

```html
<ul>
	<li>默认1</li>
	<li>默认2</li>
	<li>默认3</li>
</ul>
```

**ol**

ol(ordered list),有序表

```html
<ol>
	<li>默认1</li>
	<li>默认2</li>
	<li>默认3</li>
</ol>
```

**dl**

dl(definition list),子元素：

- dt(definition title) : 列表标题
- dd(definition description) : 列表项

```html
<dl>
	<dt>第一条</dt>
	<dd>xxx</dd>
	<dt>第二条</dt>
	<dd>xxx</dd>
</dl>
```

## 表格标签

表格标签用`<table>`表示。 一个表格`<table>`是由每行`<tr>`组成的，每行是由每个单元格`<td>`组成的

```html
<table>
	<tr>
		<td>A</td>
		<td>01</td>
	</tr>
	<tr>
		<td>B</td>
		<td>02</td>
	</tr>
</table>
```

## 框架标签

如果我们希望在一个网页中显示多个页面，可以使用框架标签

`frameset`和`frame`已经从 Web标准中删除，建议使用内嵌框架 iframe 代替

```html
<body>
 	<a href="文字页面.html" target="myframe">Google</a><br>
 	<iframe src="https://google.com" width="400" height="400" name="myframe"></iframe>
 </body>
```

## 表单标签

表单标签用`<form>`表示，用于与服务器的交互。表单就是收集用户信息的，就是让用户填写的、选择的。

**属性：**

- `name`：表单的名称，用于JS来操作或控制表单时使用；
- `id`：表单的名称，用于JS来操作或控制表单时使用；
- `action`：指定表单数据的处理程序，一般是PHP，如：action=“login.php”
- `method`：表单数据的提交方式，一般取值：get(默认)和post

注意：表单和表格嵌套时，是在`<form>`标记中套`<table>`标记。

form标签里面的action属性和method属性,action属性就是表示，表单将提交到哪里。 method属性表示用什么HTTP方法提交，有get、post两种。

**get提交和post提交的区别：**

GET方式： 将表单数据，以"name=value"形式追加到action指定的处理程序的后面，两者间用"?"隔开，每一个表单的"name=value"间用"&"号隔开。 特点：只适合提交少量信息，并且不太安全(不要提交敏感数据)。

POST方式： 将表单数据直接发送(隐藏)到action指定的处理程序。POST发送的数据不可见。Action指定的处理程序可以获取到表单数据。 特点：可以提交海量信息，相对来说安全一些，提交的数据格式是多样的(Word、Excel、rar、img)。

**Enctype：** 表单数据的编码方式(加密方式)，取值可以是：application/x-www-form-urlencoded、multipart/form-data。Enctype只能在POST方式下使用。

- Application/x-www-form-urlencoded：**默认**加密方式，除了上传文件之外的数据都可以
- Multipart/form-data：**上传附件时，必须使用这种编码方式**。

### 输入框

input：输入标签（文本框）

用于接收用户输入。

```html
<input type="text" />
```

**属性：**

- **`type="属性值"`**：文本类型。属性值可以是：
  - `text`（默认）
  - `password`：密码类型
  - `radio`：单选按钮，名字相同的按钮作为一组进行单选（单选按钮，天生是不能互斥的，如果想互斥，必须要有相同的name属性。name就是“名字”。 ）。非常像以前的收音机，按下去一个按钮，其他的就抬起来了。所以叫做radio。
  - `checkbox`：多选按钮，**name 属性值相同的按钮**作为一组进行选择。
  - `checked`：将单选按钮或多选按钮默认处于选中状态。当`<input>`标签设置为`type="radio"`或者`type=checkbox`时，可以用这个属性。属性值也是checked，可以省略。
  - `hidden`：隐藏框，在表单中包含不希望用户看见的信息
  - `button`：普通按钮，结合js代码进行使用。
  - `submit`：提交按钮，传送当前表单的数据给服务器或其他程序处理。这个按钮不需要写value自动就会有“提交”文字。这个按钮真的有提交功能。点击按钮后，这个表单就会被提交到form标签的action属性中指定的那个页面中去。
  - `reset`：重置按钮，清空当前表单的内容，并设置为最初的默认值
  - `image`：图片按钮，和提交按钮的功能完全一致，只不过图片按钮可以显示图片。
  - `file`：文件选择框。 提示：如果要限制上传文件的类型，需要配合JS来实现验证。对上传文件的安全检查：一是扩展名的检查，二是文件数据内容的检查。
- **`value="内容"`**：文本框里的默认内容（已经被填好了的）
- `size="50"`：表示文本框内可以显示**五十个字符**。一个英文或一个中文都算一个字符。 注意**size属性值的单位不是像素哦**。
- `readonly`：文本框只读，不能编辑。因为它的属性值也是readonly，所以属性值可以不写。 用了这个属性之后，在google浏览器中，光标点不进去；在IE浏览器中，光标可以点进去，但是文字不能编辑。
- `disabled`：文本框只读，不能编辑，光标点不进去。属性值可以不写。

### 下拉列表

select：下拉列表标签

`<select>`标签里面的每一项用`<option>`表示。select就是“选择”，option“选项”。

select标签和ul、ol、dl一样，都是组标签。

**`<select>`标签的属性：**

- `multiple`：可以对下拉列表中的选项进行多选。属性值为 multiple，也可以没有属性值。也就是说，既可以写成 `multiple=""`，也可以写成`multiple="multiple"`。
- `size="3"`：如果属性值大于1，则列表为滚动视图。默认属性值为1，即下拉视图。

**`<option>`标签的属性：**

- `selected`：预选中。没有属性值。

### 表单的语义化

```html
	<form>
		<fieldset>
		<legend>账号信息</legend>
		姓名：<input value="name" >name<br>
		密码：<input type="password" value="pwd" size="50"><br>
		</fieldset>

		<fieldset>
		<legend>其他信息</legend>
		性别：<input type="radio" name="gender" value="male" checked="">男
			  <input type="radio" name="gender" value="female" >女<br>
		</fieldset>

	</form>
```

### label 

```html
<input type="radio" name="sex" /> 男
<input type="radio" name="sex" /> 女
```

对于上面这样的单选框，我们只有点击那个单选框（小圆圈）才可以选中，点击“男”、“女”这两个文字时是无法选中的；于是，label标签派上了用场。

本质上来讲，“男”、“女”这两个文字和input标签时没有关系的，而label就是解决这个问题的。我们可以通过label把input和汉字包裹起来作为整体。

解决方法如下：

```html
<input type="radio" name="sex" id="nan" /> <label for="nan">男</label>
<input type="radio" name="sex" id="nv"  /> <label for="nv">女</label>
```

上方代码中，让label标签的**for 属性值**，和 input 标签的 **id 属性值相同**，那么这个label和input就有绑定关系了。

当然了，复选框也有label：（任何表单元素都有label）

```html
<input type="checkbox" id="kk" />
<label for="kk">10天内免登陆</label>
```
