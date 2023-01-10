# File Inclusion（文件包含漏洞）

## 简介

### 成因

文件包含漏洞产生的原因是代码在进行文件包含的时候，没有对用户可控制的输入进行过滤，从而造成文件内容泄漏，或者代码执行。

## PHP中的文件包含

文件包含是 PHP 的一种常见用法，主要由 4 个函数完成:

```
include()
require()
include_once() 
require_once()
```

`include()` 和 `require()` 的区别: `require()` 如果在包含过程中出错，就会直接退出，不执行后续语句 `include()` 如果在包含过程中出错，只会提出警告，但不影响后续语句的执行。

带 `once` 和不带 `once` 的区别: 带 `once` 的会判断你在加载这个文件之前是否已经加载过了文件，避免重复加载。

当使用这 4 个函数包含一个新的文件时，该文件将作为 PHP 代码执行，PHP 内核并不会在意该被包含的文件是什么类型。所以如果被包含的是 txt 文件、图片文件、远程 URL，也都将作为 PHP 代码执行。

### 本地文件包含

本地文件包含，如果没有任何限制，可以尝试获取一些敏感信息：

```
/etc/passwd // 账户信息
/etc/shadow // 账户密码文件
```

更多敏感信息路径: https://www.cnblogs.com/v1vvwv/p/Sensitive-Information-Leakge.html

也可以通过修改一些文件，来进行命令执行。一般在访问网页的时候，访问相关信息会写入日志文件，session文件

#### session文件包含

**利用条件: **已知session文件的存储位置。

session文件的存储位置可以通过 `phpinfo()` 来获取，也可以尝试使用默认的存储路径，如linux下默认存储在 `/var/lib/php/session` 。

例如下面代码:

```
<?php
    session_start();

    $name=$_GET['username'];
    $_SESSION["username"]=$name;

    $filename  = $_GET['filename'];
    include($filename);
?>
```

传入username后，就可以在存储session的文件夹下看到写入传入参数的文件了

![截屏2022-09-22 下午8.53.22](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-09/%E6%88%AA%E5%B1%8F2022-09-22%20%E4%B8%8B%E5%8D%888.53.22.png)



然后根据Cookie中的 PHPSESSID，拼接路径，进行文件包含。

![截屏2022-09-22 下午8.55.45](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-09/%E6%88%AA%E5%B1%8F2022-09-22%20%E4%B8%8B%E5%8D%888.55.45.png)

就能看到代码被执行了

![截屏2022-09-22 下午8.57.24](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-09/%E6%88%AA%E5%B1%8F2022-09-22%20%E4%B8%8B%E5%8D%888.57.24.png)

### 远程文件包含

伪协议