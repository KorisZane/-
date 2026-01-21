## 常见的身份验证技术

1、JWT
2、Shiro
3、Spring Security
4、OAuth 2.0
5、SSO
6、JAAS等

## JWT

JWT(JSON Web Token)是由服务端用加密算法对信息签名来保证其完整性和不可伪造；Token里可以包含所有必要信息，这样服务端就无需保存任何关于用户或会话的信息；
JWT用于身份认证、会话维持等。由三部分组成，header、payload与signature

### 引入依赖

```
<dependency>
<groupId>com.auth0</groupId>
<artifactId>java-jwt</artifactId>
<version>3.4.0</version>
</dependency>
```

### JWT生成代码

```
String jwtdata = JWT.create()  
        //头部 不加默认算法HMAC256  
        //.withHeader()        //payload部分  
        .withClaim("username", "qiyi")  
        .withClaim("password", "123456")  
        //签名部分  
        .sign(Algorithm.HMAC256("qiyisec"));  
System.out.println(jwtdata);  
jwtdecrypto(jwtdata);
```

### JWT解密代码

```
public static void jwtdecrypto(String jwtdata) {  
    DecodedJWT qiyisec = JWT.require(Algorithm.HMAC256("qiyisec")).build().verify(jwtdata);  
    String username = qiyisec.getClaim("username").asString();  
    String password = qiyisec.getClaim("password").asString();  
    System.out.println(username+" "+password);  
}
```

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwYXNzd29yZCI6IjEyMzQ1NiIsInVzZXJuYW1lIjoicWl5aSJ9.lvEFzV5ksl0kJncurWGbWHWLlwgkNBD3ZUlcQ76ZKo4
```

JWT分成三个部分，第一个部分是加密算法，第二个部分是加密部分，第三个是校验签名，要修改加密的部分需要加密签名才可以修改成功

在修改cookie时，即使知道账号密码，不知道jwt密钥也无法攻击jwt
### 安全问题

[https://mp.weixin.qq.com/s/xH_v825bNqDszwmMOe8CBw](https://mp.weixin.qq.com/s/xH_v825bNqDszwmMOe8CBw)

## 身份验证-Spring Security

Spring Security安全框架，是Spring Boot底层安全模块默认的技术选型，可以实现强大的Web安全控制。
WebSecurityConfigurerAdapter：自定义Security策略
AuthenticationManagerBuilder：自定义认证策略
@EnableWebSecurity：开启WebSecurity模式
"认证"和"授权"(访问控制)
"认证"(Authentication)
"授权"(Authorization)
这个概念是通用的，而不是只在 Spring Security 中存在。
参考官网 https://spring.io/projects/spring-security

### 使用案例

1、新建Spring Security+web+thymeleaf项目
2、配置application.properties模版解析
3、添加前端页面文件到templates目录
4、创建路由控制器并指向前端页面文件

properties下添加

spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration


RouterController.java

```
package com.example.security.Controller;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.PathVariable;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestParam;  
  
@Controller  
public class RouterController {  
    @RequestMapping({"/", "/index"})  
    public String index() {  
        return "index";  
    }  
  
    @RequestMapping("/toLogin")  
    public String toLogin() {  
        return "views/login";  
    }  
  
    @RequestMapping("/level1/{id}")  
    public String level1(@PathVariable int id) {  
        return "views/level1"+id;  
    }  
  
    @RequestMapping("/level2/{id}")  
    public String level2(@PathVariable int id) {  
        return "views/level2"+id;  
    }  
  
    @RequestMapping  
    public String level3(@RequestParam int id) {  
        return "views/level3"+id;  
    }  
}
```

SecurityConfig.java

```
package com.example.security.Controller;  
  
import org.springframework.stereotype.Controller;  
import org.springframework.web.bind.annotation.PathVariable;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestParam;  
  
@Controller  
public class RouterController {  
    @RequestMapping({"/", "/index"})  
    public String index() {  
        return "index";  
    }  
  
    @RequestMapping("/toLogin")  
    public String toLogin() {  
        return "views/login";  
    }  
  
    @RequestMapping("/level1/{id}")  
    public String level1(@PathVariable int id) {  
        return "views/level1"+id;  
    }  
  
    @RequestMapping("/level2/{id}")  
    public String level2(@PathVariable int id) {  
        return "views/level2"+id;  
    }  
  
    @RequestMapping  
    public String level3(@RequestParam int id) {  
        return "views/level3"+id;  
    }  
}
```