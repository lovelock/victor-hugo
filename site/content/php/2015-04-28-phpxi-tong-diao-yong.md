+++
categories = ["Php"]
title  = "PHP系统调用"
date = "2015-04-28T23:30:54+08:00"
topics = ["PHP"]
type = "post"
+++

通常认为PHP的系统调用有以下几种方式：

1. `system()`输出并返回最后一行Shell结果。
2. `exec()`不输出结果，返回最后一行Shell结果，所有结果可以保存到一个返回的数组。
3. `passthru()`只调用命令，把命令的运行结果原样的标准输出到标准输出设备上。

上面三种我目前还没有用到过，下面简单介绍一下另外一种方式`shell_exec`。

应用背景：有大量数据需要处理，对一个大数组中的所有元素进行复杂的操作，这个任务其实是不适合交给PHP来做的。这也是我第一次意识到带垃圾回收机制的语言和C语言的差别。
由于对这种特性的不了解，刚开始尝试在进行完一次任务之后将生成的变量`unset`掉，但发现其实不起作用，仍然会引起内存不足从而退出的错误信息。查看了[风雪之隅](http://www.laruence.com)的
文章[深入理解PHP内存管理之谁动了我的内存](http://www.laruence.com/2011/03/04/1894.html)之后知道了原来`unset`确实会释放内存，但释放的内存并没有像C语言那样交回给操作系统，而是用PHP
自己的内存管理API进行的。PHP并不是简单的向操作系统要内存，而是会要一个大块的内存，然后把其中一块分配给申请者，这样当再有逻辑来申请内存的时候就不需要再向操作系统申请了，避免了频繁
的系统调用。当然带来的问题就是占用的内存总量只会增加不会减少，因此`unset`在此没有实际的意义。
理想的情况是这样的，对数组的一个元素执行完所有操作，将与该元素相关的内存释放，再进行下一个操作，由于所有的元素的操作是几乎一样的，所以占有的内存应该不会有大的增加，问题是前面已经
说了，PHP不会真正意义上释放内存，那么就没有解决办法了吗？
当然有。
大致的思路是这样的，先处理第一个元素，然后用`shell_exec`执行这个文件本身。结果就是`shell_exec`这个命令已经发出，已经成功的fork了一个进程，脚本执行退出之后将会反复进行这步操作，直
至数组中的最后一个元素。这样的好处就是将处理过程串联起来，只有上一个任务结束之后才会触发下一个任务执行，不会造成CPU和内存占用过高的情况，完美的解决了我的问题。

> 中间遇到了一个另外一个问题，那就是怎样将参数传进去。比如要处理的数组是`$array = array($elem1, $elem2, $elem3)`，第一次执行时难道要将数组的元素作为命令行参数传进去？这样做显得也
> 太没有水平了，经过深入思考，解决方案应该是这样的（如果有更好的方法，也期待您能提出）

```PHP
<?php
$file = __FILE__;
$array = array($elem1, $elem2, $elem3);

if (!isset($argv[1])) {
    $id = $array[1];
} else {
    $id = /* 数组中$argv[1]元素之后的第一个元素 */
}

/* 对$id进行操作的代码 */

shell_exec("/usr/bin/env php " . $file . " " . $id . ">> /dev/null &");
?>
```

最后一步很关键，一定要把执行操作的代码放在执行文件的语句前面，这样只有在上一次操作完成之后才会继续执行该文件，进入下一个周期。

看来语言功底真是不行啊，组织的不够清晰。而且在实际项目的代码里由于用到的是数据库中查出的值，根据当前的输入查找比当前值大的下一个值是很方便的，所以用数组不是很容易表达。这也引出的另外
一个问题，就我实际项目而言，如果采用这种方法会增加很多查询数据库的次数，但由此增加的每次执行任务的开销相比完全无法完成而言，还是很值得的。
