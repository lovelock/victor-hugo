+++
categories = ["Javascript"]
title  = "JavaScript代码约定"
isCJKLanguage = true
date = "2015-05-02T21:08:49+08:00"
draft = false
topics = ["javascript"]
tags = ["javascript"]
+++

原文标题：Code Conventions for the JavaScript Programming Language
作者：Douglas Crockford

这是JavaScript编程中的一系列代码约定和规则。灵感来自于Sun的Code Conventions for the Java Programming Language。当然，因为JavaScript不是Java，所以这篇文章也完全不同于前者。

对于一个组织而言，代码库的质量和软件的长期价值成正比。在它的生命周期内，一个程序会经手很多人。如果一个程序可以清晰的传达它的结构和和特点，在不远的将来修改时就不太可能会崩溃。

编码约定有助于减少代码的脆性。

所有JavaScript代码都是公开发送的。它也应该总是出版物的质量。

整齐很重要。

## JavaScript文件

JavaScript程序应该以`.js`文件保存和传输。
JavaScript不应该嵌入在HTML文件中，除非代码是针对一个特定的session。将JS代码嵌入在HTML中，会导致由于不能通过缓存和压缩来减小页面体积而使其显著增加。

`<script src=filename.js`标签在`<body></body>`标签中出现的越晚越好。这减小了其他页面组件的的脚本加载而带来的影响。没有必要用`language`或`type`属性。决定MIME类型是服务器而不是脚本标签的职责。

## 缩进

缩进的单位是4个空格。应该避免使用制表符，因为（在21世纪的今天）仍然没有取代制表符的标准。使用空格会产生更大的文件体积，但在内网这个增长不会很明显，而且这个差异会被minification缩小。

## 行长度

避免长于80个字符的行。当一个语句在一行不合适时，可能需要断行。把断行放在操作符之后，理想情况下放在逗号后面。操作符后面的换行会减少因为插入分号而掩盖的复制粘贴错误的可能性。下一行应该缩进8个空格。

## 注释

写注释时慷慨一些吧。留一些信息给将来可能会读到的人（可能是你自己），他可能会需要理解你做的事情。注释需要写好并写清晰，就像它们要标注的代码一样。偶尔一个幽默的金块儿可能会得到感恩，但挫折和怨恨可不能。

保持注释的更新很重要。有错误的注释会让代码更难阅读和理解。

只做有意义的注释。专注于那些不能被立即看到的。不要浪费阅读者的时间去看这些东西：
```javascript
i = 0; // Set i to zero
```
通常用行注释。留着块注释做正式的文档。

## 变量声明

所有的变量都应该在使用前声明。JavaScript不要求如此，但这样做可以让代码更易读并且容易检测到那些没有声明的隐式的全局变量。隐式的全局变量永远都不要用。全局变量的使用要尽量减少。

在函数题中，`var`语句应该是第一条语句。

最好给每个变量单独的行和注释。如果可能的话，尽量用字母顺序给它们排序。
```javascript
var currentEntry, // currently selected table entry
    level,        // indentation level
    size;         // size of table
```
JavaScript没有块作用域，所以在代码块中定义变量可能会让经验丰富的C系程序员很困惑。在函数开头定义所有的变量。

## 函数声明

所有的函数都应该显定义再使用。内部函数应该位于`var`语句之后。这样可以更容易的看出在这个作用域内包含的变量。

函数名和参数列表左括号之间不应该有空格。右括号和大括号之间应该有一个空格。函数体应该缩进4个空格。
