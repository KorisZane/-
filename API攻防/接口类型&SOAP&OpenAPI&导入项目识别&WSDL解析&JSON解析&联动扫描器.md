## Soap API

在学习SOAP注入之前，先来介绍一下Web Service，Web Service是一个平台独立的、低耦合的、自包含的、基于可编程的Web的应用程序，可使用开放的XML（标准通用标记语言下的一个子集）标准来描述、发布、发现、协调和配置这些应用程序，用于开发分布式的交互操作的应用程序。

Web Service是一种远程调用技术，实质就是一个程序向外界暴露出了一个可通过Web调用的API，如果对传入的参数不做限制就有可能导致SQL注入等漏洞的产生。

Web Service有三要素，分别为soap，wsdl和uddl。

uddl用于提供发布和查询Web Service方法；

wsdl是Web Service服务描述语言，用于web服务说明，它是一个xml文档，用于说明一组soap消息如何访问接口；

soap是一种简单的基于XML的协议，它使应用程序通过HTTP来交换信息，用于分布式环境的基于信息交换的同行协议，描述传递信息的格式和规范，它可以用于连接web服务和客户端之间的接口，是一个可以在不同操作系统上运行的不同语言编写的程序之间的传输通信协议，格式为XML

## 判断方式

0、固定的页面显示内容
1、数据包里面xml格式 存在特征soap字符
2、*URL后加?wsdl能成功显示xml格式数据*

## WSDL解析

- Burp插件
=- Apifox
- Postman
- SoapUI


## OpenAPI

[[专题漏洞&SPEL表达式&SSTI模版&Swagger接口&Actuator泄露&Spring特检]]

1. 探针：

URL扫描文件目录(参考101天)

2. JSON配置解析

Apifox
Reqable
Postman

3. 自动化扫描：

自动发包后续联动扫描器（Xray,Burp,AWVS,Goby等）
只要支持代理配置的均可测试，对请求的数据包进行漏洞探针