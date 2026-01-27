## JSRpc

劫持程序运行时的控制台

1. 找到加密加密函数
	![](rp1.png)

2. 替换文件，加入连接代码
	![](rp2.png)
	*注册的方法要和文件里写的方法名一致*

3. 启动jsrpc服务端
	![](rp3.png)

4. 在控制台执行注入脚本jsEnv_Dev.js
	![](rp4.png)

5. 触发连接代码，等待服务端上线
	![](rp5.png)

6. 访问地址
	![](rp6.png)

## JSRpc配合插件

1. 配置rpc_auto.py脚本并启动
	![](rr1.png)

2. 调用接口抓包查看是否正常加密
	![](rr2.png)

3. 配置插件调用接口
	![](rr3.png)


## JsEncrypter

将代码写入到模版中

phantomjs ./phantomjs_server.js