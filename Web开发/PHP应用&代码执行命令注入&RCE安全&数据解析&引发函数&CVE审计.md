## 容易引发代码执行的PHP函数

```
assert()
pcntl_exec()
array_fi lter()
preg_replace
array_map()
require()
array_reduce()
require_once()
array_diff_uassoc()
register_shutdown_function()
array_diff_ukey()
register_tick_function()
array_udiff()
set_error_handler()
array_udiff_assoc()
shell_exec()
array_udiff_uassoc()
stream_fi lter_register()
array_intersect_assoc()
system()
array_intersect_uassoc()
usort()
array_uintersect()
uasort()
array_uintersect_assoc()
uksort()
array_uintersect_uassoc()
xml_set_character_data_handler()
array_walk()
xml_set_default_handler()
array_walk_recursive()
xml_set_element_handler()
create_function()
xml_set_end_namespace_decl_handler()
escapeshellcmd()
xml_set_external_entity_ref_handler()
exec()
xml_set_notation_decl_handler()
include
xml_set_processing_instruction_handler()
include_once()
xml_set_start_namespace_decl_handler()
ob_start()
xml_set_unparsed_entity_decl_handler()
```

PHP 提供代码执行 (Code Execution) 类函数主要是为了方便研发人员处理各类数据，然而当研发人员不能合理使用这类函数时或使用时未考虑安全风险，则很容易被攻击者利用执行远程恶意的PHP代码，威胁到系统的安全。

eval，assert，preg_replace(高版本被限制)，create_function等

https://www.yisu.com/ask/52559195.html

https://www.jb51.net/article/264470.htm

```
<?php  
//eval($_GET['q']);  
//assert(@$_GET['q']);  
//preg_replace("/pregStr/e",$_GET["q"],"cmd_pregStr");  
create_function('$a,$b', ';}phpinfo();/*');  
  
//function func($a, $b){  
//  ;}phpinfo();/*  
//}
```

## 命令执行

为了方便处理相关应用场景的功能，PHP系统中提供命令执行类函数，如 exec()、system() 等。当研发人员不合理地使用这类函数，同时调用的变量未考虑安全因素时，容易被攻击者利用执行不安全的命令调用。

exec，system，passthru，popen，shell_exec，pcntl_exec，`等

## 代码审计案例

1、Yccms 代码执行

https://mp.weixin.qq.com/s/4i4MLsNAlMuLjySBc_rySw

2、CmsEasy 代码执行

https://xz.aliyun.com/t/2577

3、BJCMS 命令执行

https://blog.csdn.net/qq_44029310/article/details/125860865

4、WBCE 命令执行

https://developer.aliyun.com/article/1566395

5、D-Link 命令执行

https://xz.aliyun.com/t/2941

6、安恒明御安全网关

https://blog.csdn.net/smli_ng/article/details/128301954