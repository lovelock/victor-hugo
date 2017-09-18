+++
date = "2017-09-17T11:22:40+08:00"
title = " BasicAuth 简介"
categories = ["http"]
tags = ["postman"]
isCJKLanguage = true
+++

# BasicAuth 简介

我在维护的一个内部服务用到了 BasicAuth 的认证方式, 但总是有人不断的问我要怎么用, 这里详细的说一下.

## 使用场景

要知道 BasicAuth 的使用场景, 它是最简单的一种认证方式, 说白了就是用户名+密码, 这种方式有很多问题, 比如它通过网络发送用户名和密码, 而这些都是以一种很容易解码的形式表示的. 虽然它是用 base64_encode 加密过了, 但这种加密的作用也仅仅是**让油耗的用户不太可能在进行网络观测时无意中看到密码**, 而不能防止恶意用户. 所以也仅限在一些安全要求不是那么高的场景下使用.

## 原理和实现

其次要明白 BasicAuth 的原理和实现. 简单来说, 它是检查你的 Headers 中的`Authorization`. 从中解析出 username 和 password, 和服务器保存的进行对比, 如果一致则通过, 否则返回401状态码, 表示未授权的请求.

## 使用方法

下面详细说一下怎么调用一个有 BasicAuth 认证的服务. 比如你的 username 是 zhangsan, password是 helloworld(当然不建议用这种太简单的密码), 那么你只需要在你调用接口时在 Header 里加上一个 Header `Authorization: Basic $(base64_encode({username}:{password}))`即可. 更具体的, 也就是`Authorization: Basic emhhbmdzYW46aGVsbG93b3JsZA==`.

具体到 postman就更简单了, 它支持多种认证方式, 当调用需要BasicAuth 认证信息的接口时, 如图所示选择并填写即可.
![](https://ws1.sinaimg.cn/large/006tKfTcly1fjnke4l71nj30nv08qjs4.jpg)

具体到某一种语言, 就不太能全面覆盖了, 如果你用了某种封装好的客户端, 比如 PHP 的 guzzlehttp 或 Java 的 OKHttp之类, 它们肯定是会像 postman 一样对这种应用相当广泛的认证方式直接支持的. 如果你用的是自己封装的客户端,那么就可以按照前面说的自己生成`Authorization`即可.