## HTTP/HTTPS差异

### 加密方式

HTTP：使用明文传输，数据在传输过程中可以被任何人截获和查看。

HTTPS：通过SSL/TLS协议对数据进行加密，确保数据在传输过程中不被第三方截获和篡改。

### 身份验证

HTTP：不需要进行身份验证，任何人都可以访问网站。

HTTPS：通过数字证书和SSL/TLS协议验证服务器的身份，防止"中间人攻击"。

***浏览器抓包和burp抓包都是在本地数据未加密之前抓包，wireshirk是抓取网卡，https数据包已经被加密***

## 身份验证鉴权技术

Cookie，Session，Token，JWT，oauth2等

参考链接 https://mp.weixin.qq.com/s/Z6rt_ggCA8dNVJPgELZ44w

### 应用场景

Cookie+Session简单，建议在内网使用；

Token相对完善，推荐在跨域外网使用；

JWT推荐使用，常用在SSO单点登录中；

OAuth灵活方便，对于第三方系统登录更友好。

### OAuth2技术

授权框架，使网站和Web应用程序能够请求对另一个应用上的用户帐户进行有限访问。至关重要的是，OAuth允许用户授予此访问权限，而无需向请求应用程序公开其登录凭据。这意味着用户可以微调他们想要共享的数据，而不必将其帐户的完全控制权移交给第三方。

#### 四种验证模式

authorization_code 授权码模式

implicit code 简单模式

password 密码模式

client_credentials 客户端模式

#### 授权码模式流程

![流程1](./images/oauth2流程1.jpg)
![关键参数](./images/oauth2参数1.jpg)
![流程2.jpg](./images/oauth2流程2.jpg)
![关键参数](./images/oauth2参数2.jpg)![示例](./images/oauth2演示.png)

#### 安全漏洞问题

1、redirect_url 校验不严格导致code被劫持到恶意网站（fuzz各种bypass方式）

2、client_id与redirect_url 不一致造成滥用劫持

3、A应用生成的code可以用在B应用上

4、state未设置csrf防护，导致csrf风险

5、scope提权，将低scope权限的code用于高权限场景

6、HTTP劫持，网络层中间人攻击

7、点击劫持：通过点击劫持，恶意网站会在以下位置加载目标网站： 透明 iFrame（参见 [ iFrame ]）覆盖在一组虚拟的顶部 精心构造的按钮直接放置在 目标站点上的重要按钮。当用户单击可见的 按钮，他们实际上是在单击一个按钮（例如“授权” 按钮）在隐藏页面上。

#### Authorization头

参考链接 https://juejin.cn/post/7300812626279251987

请求包里有Authorization可能用如下验证方式

1、Basic认证

2、Digest认证

3、Bearer认证

4、JWT认证

5、API密钥认证

6、双因素认证

7、其他一些认证方式