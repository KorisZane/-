[[JavaEE应用&JNDI注入&RMI服务&LDAP服务&DNS服务&高版本限制绕过]]

![](JNDI.jpg)
![](jndi版本.jpg)

## 序列化

序列化是将Java对象转换成字节流的过程。而反序列化是将字节流转换成Java对象的过程，java序列化的数据一般会以标记(ac ed 00 05)开头，base64编码的特征为rO0AB，JAVA常见的序列化和反序列化的方法有JAVA 原生序列化和JSON 类（fastjson、jackson）序列化等。

Java中可分为：原生反序列化类(ObjectInputStream.readObject()、SnakeYaml、XMLDecoder等)、三方组件反序列化(Fastjson、Jackson、Xstream等)

## 原生序列化类函数

- SnakeYaml：
	完整的YAML1.1规范Processor，支持Java对象的序列化/反序列化
- XMLDecoder：xml语言格式序列化类函数接口
- ObjectInputStream.readObject()：任何类如果想要序列化必须实现
- 继承java.io.Serializable接口

## 第三方组件

- Log4j：

Apache的一个开源项目，是一个基于Java的日志记录框架。
历史漏洞：https://avd.aliyun.com/search?q=Log4j
漏洞触发：logger.error logger.info

- Shiro：

Java安全框架，能够用于身份验证、授权、加密和会话管理。
历史漏洞：https://avd.aliyun.com/search?q=Shiro
漏洞触发：CookieRememberMeManager黑盒特征：数据包cookie有没有rememberme

- Jackson：

当下流行的json解释器，主要负责处理Json的序列化和反序列化。
历史漏洞：https://avd.aliyun.com/search?q=Jackson
漏洞触发：readValue

- XStream：

开源Java类库，能将对象序列化成XML或XML反序列化为对象
历史漏洞：https://avd.aliyun.com/search?q=XStream
漏洞触发：fromXML

- FastJson：

阿里巴巴公司开源的json解析器，它可以解析JSON格式的字符串，支持将JavaBean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean。
历史漏洞：https://avd.aliyun.com/search?q=fastjson

漏洞触发：parseObject parse

## Java安全-反序列化-JNDI注入

JNDI全称为 Java Naming and DirectoryInterface（Java命名和目录接口），是一组应用程序接口，为开发人员查找和访问各种资源提供了统一的通用接口，可以用来定义用户、网络、机器、对象和服务等各种资源。JNDI支持的服务主要有：DNS、LDAP、CORBA、RMI等。
RMI：远程方法调用注册表
LDAP：轻量级目录访问协议


JDK 6u45、7u21之后：

java.rmi.server.useCodebaseOnly的默认值被设置为true。当该值为true时，将禁用自动加载远程类文件，仅从CLASSPATH和当前JVM的java.rmi.server.codebase指定路径加载类文件。使用这个属性来防止客户端VM从其他Codebase地址上动态加载类，增加RMI ClassLoader安全性。

JDK 6u141、7u131、8u121之后：

增加了com.sun.jndi.rmi.object.trustURLCodebase选项，默认为false，禁止RMI和CORBA协议使用远程codebase的选项，因此RMI和CORBA在以上的JDK版本上已经无法触发该漏洞，但依然可以通过指定URI为LDAP协议来进行JNDI注入攻击。

JDK 6u211、7u201、8u191之后：

增加了com.sun.jndi.ldap.object.trustURLCodebase选项，默认为false，禁止LDAP协议使用远程codebase的选项，把LDAP协议的攻击途径也给禁了


## 归类总结

代码审计Sink点：

1、JDK(ObjectInputStream.readObject)
2、XMLDecoder.readObject
3、Yaml.load
4、XStream.fromXML
5、ObjectMapper.readValue
6、JSON.parse JSON.parseObject
7、CookieRememberMeManager
8、logger.error logger.info

