## Shiro

Java安全框架，能够用于身份验证、授权、加密和会话管理。
历史漏洞：https://avd.aliyun.com/search?q=Shiro
漏洞触发：CookieRememberMeManager黑盒特征：数据包cookie有没有rememberme

漏洞原理：利用到原生类的readwriteObjec/writeObje反序列化操作导致漏洞（相比多了对数据的aes加密和base64编码），所以利用要考虑这两点

## Shiro反序列化链分析

当获取用户请求时，大致的关键处理过程如下：

· 获取Cookie中rememberMe的值
· 对rememberMe进行Base64解码
· 使用AES进行解密
· 对解密的值进行反序列化

由于AES加密的Key是硬编码的默认Key，因此攻击者可通过使用默认的Key对恶意构造的序列化数据进行加密，当CookieRememberMeManager对恶意的rememberMe进行以上过程处理时，最终会对恶意数据进行反序列化，从而导致反序列化漏洞

## Shiro 550利用链分析

1. 获取AES加密属性

成功登录：rememberMeSuccessfulLogin断点（动态调试）
AES/CBC/PKCS5Padding
key=kPH+bIxk5D2deZiIxcaaaA==
DEFAULT_CIPHER_KEY_BYTES = Base64.decode("kPH+bIxk5D2deZiIxcaaaA==");

key：[-112, -15, -2, 108, -116, 100, -28, 61, -99, 121, -104, -120, -59, -58, -102, 104]
iv：[6, -28, 13, 52, -37, -71, 7, -11, 112, 30, -61, 7, -19, 87, -55, -99]

2. 将Payload进行加密（AES+BASE64）

- 借助URLDNS链(安全开发讲过)
- 借助CC&CB链(下节课分析这个链)

问题1：为什么不能用fastjson里面的jdbc那个链？

fastjson链找有setxxx或getxxx方法里面触发rce
shiro链找有readObject或toString方法里面触发rce

问题2：对比利用工具中出现的多个链是什么情况导致？

java -jar ysoserial.jar URLDNS "http://" > urldns.txt

3. 将rememberMe数据发包

- 借助java-chains项目生成Poc