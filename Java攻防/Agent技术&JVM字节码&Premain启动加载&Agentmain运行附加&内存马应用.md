## Java_Agent

是一种可以在JVM启动时或运行时附加的工具，它可以拦截并修改类文件字节码。Java Agent通常用于实现AOP（面向切面编程）、性能监控、日志记录等功能

参考

https://docs.qq.com/scenario/link.html?url=https%3A%2F%2Fdocs.dingtalk.com%2Fi%2Fnodes%2FQPGYqjpJYrPrKaQ5UE6eZkY38akx1Z5N&pid=300000000%24CpIbGVVOJlCb&cid=144115210464040226&nlc=1

## Java Agent有两种加载方式

1. Premain使用

	在JVM启动时通过命令行参数-javaagent:path/to/xx.jar来指定
```
mvn package

mvn clean package

java -cp . zero.overflow.Main

java -javaagent:D:\V2024-11\116\MyAgent\target\MyAgent-1.0-SNAPSHOT-jar-with-dependencies.jar -cp . zero.overflow.Main
```

2. Agent使用

在JVM已经启动后，通过Attach API动态地附加到正在运行的JVM进程上。

```
public class AgentMainTest {  
    public static void agentmain(String agentArgs, Instrumentation inst)  
    {  
        inst.addTransformer(new MyTransformer(),true);  
        for (Class<?> clazz : inst.getAllLoadedClasses()){  
            if (clazz.getName().equals("zero.overflow.Fox")){  
                try {  
                    inst.retransformClasses(clazz);  
                } catch (UnmodifiableClassException e) {  
                    throw new RuntimeException(e);  
                }  
            }  
        }  
    }  
}
```

动态注入

```
public class Main {  
    public static void main(String[] args) throws AgentLoadException, IOException, AgentInitializationException, AttachNotSupportedException {  
        VirtualMachine attach = VirtualMachine.attach("37416");  
        attach.loadAgent("D:\\V2024-11\\116\\MyAgent\\target\\MyAgent-1.0-SNAPSHOT-jar-with-dependencies.jar");  
    }  
}
```

添加依赖

```
<dependency>
<groupId>com.sun</groupId>
<artifactId>tools</artifactId>
<version>1.8</version>
<scope>system</scope>
<systemPath>${java.home}/../lib/tools.jar</systemPath>
</dependency>
```

3. Agent内存马注入：

- 利用javassist注入Filter内存马

```
<dependency>
<groupId>org.javassist</groupId>
<artifactId>javassist</artifactId>
<version>3.30.2-GA</version>
</dependency>
```

- 冰蝎内存马项目：

https://github.com/rebeyond/memShell