+++
categories = ["Php"]
title  = "composer 问题小记"
date = "2017-07-14T22:41:18+08:00"
author = "frostwong"
type = "post"
topics = ["PHP", "Composer"]
+++

今天使用 composer 遇到了一些问题,记录一下.

首先是 composer 对于依赖冲突的无能为力. 比如我的项目依赖了 a 项目,而 a 项目依赖了 1.0版本的 b 项目. 而我的项目自己又依赖了 2.0版本的 b 项目, composer 除了报告冲突之外什么也做不了.

这个问题是我在开发[bran: HTTP Api 单元测试框架](https://github.com/subtlephp/bran)时遇到的, 具体是我在业务项目中用到了 laravel database 相关的包, 它的一个依赖在`require-dev`中指定了 phpunit 4.*版本. 而bran 依赖 phpunit 6.*. 那么问题来了, 因为 bran 是后加入的依赖, phpunit 6.*的包装不了.

我分析了一下, 首先 bran 是一个 单元测试框架, 所以它对于业务项目来说其实是应该放在`require-dev`里面的. 而 phpunit 一般来说也应该永远放在`require-dev`中, 但 bran 是基于 phpunit 的,所以phpunit 是 bran 的一个组件,而非开发时的依赖. 所以我就把 phpunit 的依赖移入 bran 的`require`中. 然后执行`composer update --no-dev`这样就不会安装4.*版本的 phpunit 了.

这样问题又来了, 我的业务项目要基于 bran 写单元测试, 测试用例写在`tests`目录下, 而之前我已经在业务项目的 `composer.json`中添加了

```json
"autoload-dev": {
    "psr-4": {
        "Tests\\Api\\": "tests/Api"
    }
}
```

这时再执行单元测试时会报找不到`tests/Api`中的类, 原来是因为如果`composer update`的时候加上了`--no-dev`, 相应的`composer.json`中的`autoload-dev`字段也不会生效了. 最终也只能妥协的把这部分配置移动到`autoload`里面了.

这里讲的是一个特例,可以通过这种取巧的手段来解决, 但如果发生冲突的不是像 phpunit 这种包呢?我们又该怎么办?