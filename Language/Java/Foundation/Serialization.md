#   Serialization
##  what?
Serialization是指把类或者基本的数据类型持久化(persistence)到数据流(Stream)中
包括文件、字节流、网络数据流
##  why?
*   对象持久化
*   对象复制
*   对象传输
##  when？
*   序列化默认写入流的数据
    *   对象所属类
    *   类的签名
    *   所有非transient和非static
        *   transient修饰的变量在反序列化后是默认值（而非对象当时值）
        *   static修饰的变量，是类变量不作为对象反序列化的内容
    *   引用的对象也将序列化
    *   多对象指向同一对象，会采用sharing reference机制
##  how？
*   implements Serializable
*   implements Externalizable
    *   Externalizable继承了Serializable，该接口中定义了两个抽象方法：writeExternal()与readExternal()。
    *   Override writeExternal()和 readExternal()
    *   若没有在这两个方法中定义序列化实现细节，则输出的内容为空。
    *   还有一点值得注意：在读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中。
    *   所以，实现Externalizable接口的类必须要提供一个public的无参的构造器。
*   using ObjectInputStream and ObjectOutputStream

### Note
*   ArrayList是如何序列化和反序列化？
    *   ArrayList实现了java.io.Serializable接口，但elementData是transient的
    *   JVM会试图调用对象类里的 writeObject 和 readObject 方法，进行用户自定义的序列化和反序列化
        *   在使用ObjectOutputStream的writeObject方法和ObjectInputStream的readObject方法时，会通过反射的方式调用对象类中的writeObject 和 readObject 方法
    *   ArrayList的elementData为什么是transient，为了优化存储。因为ArrayList实际上是动态数组，如果不transient将会序列化很多null 
*   如何自定义序列化和反序列化策略
    *   在类中增加writeObject 和 readObject 方法可以实现自定义序列化策略
*   Serializable空接口，它是怎么保证只有实现了该接口的方法才能进行序列化与反序列化的呢？
    *   通过ObjectOutputStream的writeObject的调用栈：writeObject ---> writeObject0 --->writeOrdinaryObject--->writeSerialData--->invokeWriteObject
    *   其中作了类型检查(obj instanceof Serializable|Enum)    
### Example
```java

import java.io.Serializable;

public class DemoObject implements Serializable {
    /**
    * JVM是否允许反序列化，不仅取决于类路径和功能代码是否一致，
    * 一个非常重要的一点是两个类的序列化 ID 是否一致（就是 private static final long serialVersionUID)
    * 有些时候，通过改变序列化 ID 可以用来限制某些用户的使用
    */
    private static final long serialVersionUID = 9527L;

    private String name;
    private int age;

    public DemoObject(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}



public class DemoSerialization {
    public static void main(String[] args) throws Exception {
        /**
         * 序列化
         */
        DemoObject powar = new DemoObject("powar", 28);
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(powar);
        /**
         * 反序列化
         */
        ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
        DemoObject dot = (DemoObject) ois.readObject();
        System.out.println(dot.getName());
    }
}

```
### Switcher (JAVA,JSON)
*   https://github.com/FasterXML/jackson-databind
*   阿里工程师开源的FastJson        
####    Tutorial
*   http://www.hollischuang.com/archives/1158