---
title: Introduction
date: '2021-11-07'
categories:
 - Frontend
publish: true
---

# 简介

## Web

**WWW**(**W**orld **W**ide **W**eb) 万维网，通常称为Web[^1]

##  网页

网页是构成网站的基本元素，由文字，图像，超链接等元素构成

## 浏览器

浏览器是网页运行的平台，常见的浏览器有Chrome,Safari,Firefox,Edge等

### 组成

浏览器由两个部分组成：

- 渲染引擎（浏览器内核）
- Javascript 引擎

#### 渲染引擎

渲染引擎[^2]即浏览器内核，用于解析`HTML`和`CSS`，决定了浏览器如何显示网页的内容及样式；渲染引擎的不同是浏览器兼容问题的根本原因

常见的浏览器内核如下：

| Browser | Rendering Engine |
| :-----: | ---------------- |
| Chrome  | Blink            |
| Safari  | Webkit           |
| Firefox | Gecko            |

#### Javascript 引擎

Javascript 引擎也称 JavaScript 解释器，用于解析网页中的js代码，处理并运行

浏览器本身不会执行js程序，其通过js引擎来逐行解释代码后执行

常见浏览器的js引擎如下：

| Browser | **JavaScript engine**                                        |
| :-----: | :----------------------------------------------------------- |
| chrome  | V8                                                           |
| Safari  | Nitro                                                        |
| Firefox | SpiderMonkey（1.0-3.0）/ TraceMonkey（3.5-3.6）/ JaegerMonkey（4.0-） |

### 工作原理




---

本文采用[Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可


## 参考

1. [认识Web和Web标准](https://web.qianguyihao.com/01-HTML/01-%E8%AE%A4%E8%AF%86Web%E5%92%8CWeb%E6%A0%87%E5%87%86.html#web%E3%80%81%E7%BD%91%E9%A1%B5%E3%80%81%E6%B5%8F%E8%A7%88%E5%99%A8)  qianguyihao
2. [浏览器的介绍](https://web.qianguyihao.com/01-HTML/02-%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9A%84%E4%BB%8B%E7%BB%8D.html) qianguyihao






[^1]:The **[World Wide Web](https://en.wikipedia.org/wiki/World_Wide_Web)** (**WWW**), commonly known as the **Web** -- wikipedia

[^2]: A **[browser engine](https://en.wikipedia.org/wiki/Browser_engine)** ([also known as](https://en.wikipedia.org/wiki/Browser_engine#Name_and_scope) a **layout engine** or **rendering engine**) is a core [software component](https://en.wikipedia.org/wiki/Software_component) of every major [web browser](https://en.wikipedia.org/wiki/Web_browser). The primary job of a browser engine is to transform [HTML](https://en.wikipedia.org/wiki/HTML) documents and other resources of a [web page](https://en.wikipedia.org/wiki/Web_page) into an interactive visual representation on a [user](https://en.wikipedia.org/wiki/User_(computing))'s device -- wikipedia
