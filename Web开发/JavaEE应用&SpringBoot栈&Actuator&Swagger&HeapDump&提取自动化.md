## Actuator

![actuator](./images/端点.jpg)
### 配置

```
server.port=8080  

management.server.port=8081  
management.endpoints.jmx.exposure.include=*  
management.endpoints.web.exposure.include=*  
management.endpoint.health.show-details=always
```

显示所有

```
server.port=8080  

management.server.port=8081  
management.endpoints.jmx.exposure.include=health 
management.endpoints.web.exposure.include=health 
management.endpoint.health.show-details=always
```

只显示health

### 基于application.properties

management.endpoints.web.exposure.include=*

### 基于application.yml

management:
	endpoints:
		web:
			exposure:
				include: '*'


### 安全配置：

```
application.properties
management.endpoint.env.enabled=false
management.endpoint.heapdump.enabled=false
```
#### application.yml
	management:
		endpoint:
			heapdump:
				enabled: false  启用接口关闭
			env:
				enabled: false  启用接口关闭

## 图像化Server&Client界面

Server 增加@EnableAdminServer标签，开放端口8088

Client 配置监控

```
server.port=8080  
  
spring.boot.admin.client.url="http://127.0.0.1:8088"  
  
management.endpoints.jmx.exposure.include=*  
management.endpoints.web.exposure.include=*  
management.endpoint.health.show-details=always  
  
#配置登录账号密码  
#spring.boot.admin.client.instance.prefer-ip=true  
#spring.security.user.name=admin  
#spring.security.user.password=123456  
#spring.boot.admin.client.instance.service-base-url="http://127.0.0.1:8080"
```

访问8088端口查看端点

## heapdump文件泄露

访问/actuator找到humpdimp文件下载

利用工具收集泄露信息
### JDumpSpider

java -jar JDumpSpider.jar heapdump文件名

### heapdump_tool

## 额外安全

[https://github.com/LandGrey/SpringBootVulExploit](https://github.com/LandGrey/SpringBootVulExploit)

[https://github.com/wh1t3zer/SpringBootVul-GUI](https://github.com/wh1t3zer/SpringBootVul-GUI)

### 示例

```
SpringCloud Gateway RCE（CVE-2022-22947）
->创建SpringCloud Gateway+Actuator项目
->更改项目版本及漏洞Gateway依赖版本
<spring-boot.version>2.5.2</spring-boot.version>
<spring-cloud.version>2020.0.3</spring-cloud.version>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-gateway</artifactId>
<version>3.1.0</version>
</dependency>
->启动项目进行测试
参考：https://www.cnblogs.com/qgg4588/p/18104875
```

## Swagger接口

Swagger是当下比较流行的实时接口文文档生成工具。接口文档是当前前后端分离项目中必不可少的工具，在前后端开发之前，后端要先出接口文档，前端根据接口文档来进行项目的开发，双方开发结束后在进行联调测试

参考 参考：[https://blog.csdn.net/lsqingfeng/article/details/123678701](https://blog.csdn.net/lsqingfeng/article/details/123678701)

1、引入依赖

```
<--2.9.2版本-->
<dependency>
<groupId>io.springfox</groupId>
<artifactId>springfox-swagger2</artifactId>
<version>2.9.2</version>
</dependency>
<dependency>
<groupId>io.springfox</groupId>
<artifactId>springfox-swagger-ui</artifactId>
<version>2.9.2</version>
</dependency>
```

```
<--3.0.0版本-->
<dependency>
<groupId>io.springfox</groupId>
<artifactId>springfox-boot-starter</artifactId>
<version>3.0.0</version>
</dependency>
```

### 配置访问

```
#application.properties
spring.mvc.pathmatch.matching-strategy=ant-path-matcher

#application.yml
spring
	mvc:
		pathmatch:
			matching-strategy: ant_path_matcher
```



2.X版本启动需要注释@EnableSwagger2
3.X版本不需注释，写的话是@EnableOpenApi
2.X访问路径：[http://ip:port/swagger-ui.html](http://ip:port/swagger-ui.html)
3.X访问路径：http://ip:port/swagger-ui/index.html

### 安全问题

```
自动化测试：Apifox Reqable Postman
泄漏应用接口：用户登录，信息显示，上传文件等
可用于对未授权访问，信息泄漏，文件上传等安全漏洞的测试.
```