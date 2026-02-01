## 查杀平台

https://n.shellpub.com/
https://s.threatbook.com/
https://www.virustotal.com/
[https://ti.aliyun.com/#/webshell](https://ti.aliyun.com/#/webshell)

其他：牧云、河马、D盾、长亭百川、WebDir+等

## 工具项目

https://github.com/xiaogang000/XG_NTAI

## 参考文章

https://mp.weixin.qq.com/s/vTEReWXH2ZpNakD_zBsUkw
[https://mp.weixin.qq.com/s/hE8p0w9IFWUjXugn1N2Z4g](https://mp.weixin.qq.com/s/hE8p0w9IFWUjXugn1N2Z4g)
https://mp.weixin.qq.com/s/4YBwAYeeldQV7dt_7hdEJA

## 静态免杀

1. 加密混淆

	XOR AES BASE64 字符反转等

2. 利用注释

	/\*\*/等关键字，如Runtime/\*\*/.getRuntime()/\*\*/.exec

3. 改变特征

	常见代码中变量或字符关键字修改

4. 反射机制

	利用反射获取类进行调用

5. 字节码加载

	BCEL ClassLoader等

6. 远程分离加载

	URL远程加载Class文件调用类方法

## WebShell管理工具

动态：传输协议，流量特征等
