+++
date = "2017-09-17T11:44:40+08:00"
title = "postman 配置 Host"
categories = ["http"]
tags = ["postman"]
isCJKLanguage = true
+++

通常我们是无法从办公网直接访问线上环境的服务的, 这时就需要配置 Host, 最简单的方式是修改`/etc/hosts`文件, 配置类似
`10.111.111.111 inner.service.com`这种, 这样再 `ping inner.service.com`时就会发现这个域名已经指向了10.111.111.111. 用这种方法可选的工具跨平台目前也就是[SwitchHosts](https://github.com/oldj/SwitchHosts)比较好用. 具体到 Mac 平台有一个[gasmask](https://github.com/2ndalpha/gasmask)可用. 如果你不喜欢用 js 写的客户端, 这个原生应用可能更符合你的胃口.

有些时候我们可能不想这么做. 比如你想直接调 通过 ip 来访问服务. 还以前面的这个服务为例, 10.111.111.111这台机器部署了多个服务, 而如果你直接调 ip 时 Nginx 会去查找默认的服务, 也就是你在配置 Nginx 时这样的配置
![](https://ws1.sinaimg.cn/large/006tKfTcly1fjnksukqmbj309801u0so.jpg)

但很多时候Nginx 都是多 vhosts 的, 这就需要在调用 IP 的时候在 Headers 中加上`Host: inner.service.com`.

这时候用 Postman 的各位可能又不同意了, 为什么我设置了 Host 却发现这一项是黄色的.这样
![](https://ws1.sinaimg.cn/large/006tKfTcly1fjnkvp3srnj30nj04ujrs.jpg)

这其实是在提示你需要给你的 Chrome 安装[Postman Interceptor](https://chrome.google.com/webstore/detail/postman-interceptor/aicmkgpgakddgnaphhhpliifpcfhicfo?hl=en), 然后在 postman 里面打开
![](https://ws4.sinaimg.cn/large/006tKfTcly1fjnky8pmf4j305y01twee.jpg)
这个开关, 这时你就会发现那个黄色的提示不见了,而你也可以正常的调通接口了.