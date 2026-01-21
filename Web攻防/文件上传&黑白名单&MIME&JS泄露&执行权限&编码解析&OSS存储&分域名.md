## 文件上传实战情况

1. 执行权限
		文件上传后存储目录不给执行权限
		绕过条件：能控制上传文件存储目录
		
2. 解码还原
		数据做存储，解析固定（文件后缀名无关）
		文件上传后利用编码传输解码还原
		绕过：无法直接绕过
		
3. 分站存储
		upload.xiaodi8.com 上传
		images.xiaodi8.com 存储
		绕过：看存储的执行策略

4. oss对象
		Access控制-OSS对象存储-Bucket对象
		绕过：无法直接绕过

## 绕过方式

1. 前端限制
		上传允许的后缀格式文件，抓包修改文件后缀

2. .htacess
		上传一个叫.htacess的文件，文件内容
			```
			AddType application/x-httpd-php .png
			```
		上传png文件

3. MIME
		修改Content-Type文件头为image/jpeg

4. 文件头判断
		 修改文件头为允许上传文件后缀的文件头

5. 黑名单过滤不严

6. 低版本GET-%00截断
		自动解码一次
		/var/www/html/upload/x.php%00

7. 条件竞争
		上传文件先保存在临时目录之后再进行逻辑判断是否允许文件上传
			```
			<?php fputs(fopen('xiao.php','w'),'<?php eval($_REQUEST[1]);?>');?>
			```
			上传不断发包
			请求不断发包
			请求写入的shell检查shell是否生成

## 文件上传场景

1、注册用户上传地方

2、JS或API接口的代码

3、后台或其他管理页面

4、源码泄露或盲测文件

5、三方编辑器上传漏洞

6、特定的源码审计漏洞