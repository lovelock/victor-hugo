![screenshot](https://github.com/lovelock/hugo_theme_tech/blob/master/screenshot.png)

# 功能

* 简洁明了
* TOC支持
* 更好的中文字体支持
* Google Analytics
* 多说评论
* 分享（目前仅支持新浪微博）


# 安装

[hugoThemes#Installing Themes](https://github.com/spf13/hugoThemes#installing-themes).

# 配置

**config.yaml**

``` toml
baseurl = "http://unixera.com/"
languageCode = "en-us"
title = "Me & Web"
copyright = "(c) 2013-2016 Frost Wong. All rights reserved."
theme = "beg"
# Pagination
canonifyurls = true
paginate = 10
paginatePath = "page"

[params]
    toc = true
    duoshuoShortname = "UnixAgain"
    Slogan = "致力于创作可操作性更好的文章"
    SyntaxHighlightTheme = "github.min.css"
    Author = "Frost Wong"
```

更多配置信息请参考[我的博客源码](https://github.com/lovelock/blog-hugo)。

**文章示例**

``` toml
+++
date = "2016-10-10T17:53:14+08:00"
title = "在本地单机部署Hadoop/Storm运行环境"
categories = ["Java"]
tags = ["Storm", "Kafka", "Java"]
+++

Contents here
```

# 联系我

个人博客: http://unixera.com  
GitHub主页：https://github.com/lovelock  
新浪微博：@UnixAgain
