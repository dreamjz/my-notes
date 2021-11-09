# 基本语法

## 编码

默认情况下，python3源码文件以`UTF-8`编码，可以为源码文件指定不同的编码：

```
# -*- coding: cp-1252 -*-
```

上述定义允许在源文件中使用`Windows-1252`字符集，对应语言为保加利亚语、白罗斯语、马其顿语、俄语、塞尔维亚语

## 标识符

- 第一个字符必须是字母或下划线`_`
- 标识符的其他部分由字母，数字和下划线组成
- 对大小写敏感

在python3中可以使用中文做变量名，非ASCII标识符也是允许的

## 保留字

python保留字不能用于标志符名称，使用`keyword.kwlist`可以输出所有的保留字

```sh
>>> import keyword
>>> keyword.kwlist
['False', 'None', 'True', '__peg_parser__', 'and', 'as', 'assert', 'async', 'await', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```

## 注释

单行注释使用`#`,多行注释可以用`'''`和`"""`

```python
# python-basics/syntax/comment.py
# single line comment

'''
    mulitline comment
'''

"""
    mulitline comment
"""

print("Hello world")
```

## 缩进

python 使用缩进表示代码块，无需使用大括号`{}`,缩进的空格是可变的，但是同一代码块的缩进相同空格数(建议为4个空格)

```python
if true :
    print("true")
else:
    print("false")
  print("else") # 
```

```
  File "/home/kesa/MyDocuments/MyGitRepo/python-notes/python-projects/python-basics/code-indentation.py", line 5
    print("else")
                 ^
IndentationError: unindent does not match any outer indentation level
```

缩进不一致将会报错

## 多行语句

python可以通过反斜杠`\`实现多行语句:

```python
// 
a = 1+ \
    2+ \
    3
print(a) # 6
```

## 引号

python 可以使用单引号`'`,双引号`"`和三个双引号`"""`,其中`"""`可以使用多行文本

```python
str1 = 'ABC'
str2 = "EFG"
str3 = """
A
B
C
"""

print(str1)
print(str2)
print(str3)


```

```sh
$ python3 quotation.py 
ABC
EFG

A
B
C
```







