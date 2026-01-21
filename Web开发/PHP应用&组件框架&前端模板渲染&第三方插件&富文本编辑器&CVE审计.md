## 模板引擎

模板引擎是为了让前端界(html)与程序代码(php)分离而产生的一种解决方案，简单来说就是html文件里再也不用写php代码了。Smarty的原理是变量替换原则，我们只需在html文件写好Smarty的标签即可，例{name}，然后调用Smarty的方法传递变量参数即可

Php模版框架：Smarty、Twig，codeeval等

老牌项目Smarty    Laravel框架内Blade    跨框架/独立项目Twing

### 基本使用

SSTI（Server Side Template Injection，服务器端模板注入）

下载 https://github.com/smarty-php/smarty/releases

下载源码到项目根目录下

index.tpl

```
<!Doctype html>  
<html>  
<head>  
    <title>{$title}</title>  
</head>  
<body>  
<h1>{$title}</h1>  
<h2>{$xd}</h2>  
</body>  
</html>
```

demo.php

```
<?php  
error_reporting(E_ALL);  
ini_set('display_errors', 1);  
require __DIR__.'/smarty/libs/Smarty.class.php';  
  
$smarty = new Smarty();  
  
#设置Smarty相关属性  
$smarty->template_dir = __DIR__.'/smarty/templates/';  
$smarty->compile_dir = __DIR__.'/smarty/templates_c/';  
$smarty->cache_dir = __DIR__.'/smarty/cache/';  
$smarty->config_dir = __DIR__.'/smarty/configs/';  
  
$smarty->assign('title', "你好");  
$smarty->assign('xd','小迪是gay');  
  
$smarty->display('index.tpl');
```

渲染index.tpl文件，访问demo.php就是被渲染的index.tpl

### 白盒思路

1 使用smarty模板引擎
2 版本存在已知的CVE漏洞
3 可控的渲染文件或变量

### 黑盒思路

参数提交CVE的poc

富文本编辑器思路同上