+++
categories = ["Miscs"]
type = "post"
author = "frostwong"
topics = ["PHP"]
tags = ["PHP"]
date = "2016-01-09T21:11:33+08:00"
description = "动态类型和强类型"
draft = false
title = "谈谈动/静态类型和强/弱类型"
+++

作为一个PHP程序员，这个问题本来不应该是我考虑的。
我需要知道仅仅是如果我的程序需要接受一个integer作为输入，拿到输入后最好能`intval($var)`一下，保证输入的是integer。而让我感到不理解的是，为什么PHP的强制类型转换会做成`(int)$var`这种方式，按照正常人的理解，不管`int/string`是关键字还是函数，要么作为`int $var`，这样具有迷惑性，因为在别的语言里这都是用来**声明变量**的，要么`int($var)`，这都很容易理解，然而。。。

好了，想到这个问题是因为这两天算是深入的用了Python的一些功能，当然主要还是用来处理日志，当我发现当我将两个从`dict`中取出的值相加，然后和一个数字的值对比时，并没有出现我要的结果。于是就查了下Python的类型。原来Python是**动态类型**，同时是**强类型**。

我看到网上很多人对这个问题还挺迷惑。刚看了PHP对于类型的解释，其实很能说明问题。

> PHP 在变量定义中不需要（或不支持）明确的类型定义；变量类型是根据使用该变量的上下文所决定的。也就是说，如果把一个 `string` 值赋给变量 `$var`，`$var` 就成了一个 `string`。如果又把一个`integer` 赋给 `$var`，那它就成了一个`integer`。

这，就是标准的动态类型了。相应的，`var a = 20 :Int`，声明了变量`a=20`，同时指定该变量的类型是`Int`，如果`var a = '20': Int`在编译时就会报错，没错这就是静态类型（这是Swift的语法）。有人可能就会拿这个举例说C也是静态类型——的确，我也会认为它是静态类型，因为它也需要指定类型才可以定义——然而，判断是否是静态类型的根据并不在此，而是像[知乎@姚培森的答案](https://www.zhihu.com/question/19918532#answer-18824124)中说的，是根据它是否所有程序都是**well behaved**。这个就太深了，我就不深究了，毕竟对C的研究也不深，说错了还不如不说。

那再来看看让我误解的Python，无疑，Python和PHP一样在定义变量时也是不需要指定的，但对PHP来说，

```php
$a = 1;
$b = '2';
echo $a + $b;
```

这样的代码完全没有问题。但到了Python这里

```python
a = 1
b = '2'
print(a + b) # 没错，我选择Python3
```

结果呢:

```
Trt recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

这就是强类型和弱类型的区别了。强类型不允许出现**forbidden behaviors**。

从这一点也就可以理解为什么Python的JIT很早前就做出来了，而前段时间鸟哥还在说之前尝试做过PHP的JIT，但发现难度太大，而现在的PHP7实际就是在为后面的JIT铺路呢。

但让我不解的是既然Python都在这方面占了优势了，为什么还是性能不行呢？



