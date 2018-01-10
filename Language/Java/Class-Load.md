##  Class Loading
![loading](https://github.com/powar2sun/Note/blob/master/Language/pictures/loading.png)

*   装载
    1.  寻找*.class文件，并产生该文件的二进制数据流(loadClassData()方法)
    2.  解析该二进制数据流为方法区内的数据结构    
    3.  创建一个该类型的java.lang.Class实例（通过defineClass()创建一个Java类型对象） 
*   连接
    *   验证，class文件校验器需要四趟独立的扫描来完成验证工作
        1.  结构检查，a)对魔数进行检查，以判断该文件是否是一个正常的class文件。b）对主次版本号进行检查，以判断class文件是否与java虚拟机兼容。c）对class文件的长度和类型进行检查，避免class文件部分缺失或被附加内容
        2.  语义检查，a）final类不能拥有子类。b）final方法不能被重写(覆盖)。c）子类和超类之间没有不兼容的方法声明。d）检查常量池入口类型是否一致(如CONSTANT_Class常量池的内容是否指向一个CONSTANT_Utf8字符串常量池) e）检查常量池的所有特殊字符串确定它们是否是其所属类型的实例，以及是否符合特定的上下文无关语法、格式 
        3.  第三趟扫描为字节码验证，其验证内容和实现较为复杂，主要检验字节码是否可以被java虚拟机安全地执行
        4.  第四趟扫描在解析过程中进行，为对符号引用的验证。在动态连接过程中，通过保存在常量池的符号引用查找被引用的类、接口、字段、方法时，在把符号引用替换成直接引用时，首先需要确认查找的元素真正存在，然后需要检查访问权限、查找的元素是否是静态类成员而非实例成员。
    *   准备，为类变量分配内存、设置默认初始值
        *   内存设置初始值，而非对类变量真正地进行初始化。
        *   int i = 5，但实际上这里是分配内存并设置初始值为0
    *   解析
        *   在类的常量池中寻找类、接口、字段、方法的符号引用，将这些符号引用替换成直接引用
        
![classCyc](https://github.com/powar2sun/Note/blob/master/Language/pictures/classLifeCyc.png)        
##  Class Loader
JVM 启动时会用 bootstrap 类加载器加载一个初始化类，然后这个类会在public static void main(String[])调用之前完成链接和初始化
####    Tutorial
*   http://www.importnew.com/17105.html
*   http://www.cnblogs.com/zhguang/p/3154584.html
*   https://segmentfault.com/a/1190000005608960
####    类加载器
java的classloader一般是采用委托机制，即classloader都有一个parent classloader.
当它收到一个加载类的请求时，会首先请求parent classloader加载，如果parent classloader加载不到，才会自己去尝试加载
（如果自己也加载不到，则抛出ClassNotFoundException）。
采用这种机制的目 的，主要是从安全角度考虑。比如用户自己定义了一个java.lang.Object，把jdk中的覆盖了，那显然是有问题的
![classLoader](https://github.com/powar2sun/Note/blob/master/Language/pictures/classLoader.png)
####    加速类加载
共享类数据（CDS）是Hotspot JVM 5.0 的时候引入的新特性。在 JVM 安装过程中，安装进程会加载一系列核心 JVM 类（比如 rt.jar）到一个共享的内存映射区域
CDS 减少了加载这些类需要的时间(如无需连接验证过程)，提高了 JVM 启动的速度，允许这些类被不同的 JVM 实例共享，同时也减少了内存消耗。
####    Jar包 预加载
```
URL jarUrl = classA.getProtectionDomain().getCodeSource().getLocation();
JarFile jarFile = new JarFile(jarUrl.getPath());
Enumeration entries = jarFile.entries();
```
##  Class File
####    Tutorials
*   http://www.importnew.com/17086.html
*   http://rednaxelafx.iteye.com/blog/652719

####    类文件结构
```
ClassFile {
    u4            magic;
    u2            minor_version;
    u2            major_version;
    u2            constant_pool_count;
    cp_info        contant_pool[constant_pool_count – 1];
    u2            access_flags;
    u2            this_class;
    u2            super_class;
    u2            interfaces_count;
    u2            interfaces[interfaces_count];
    u2            fields_count;
    field_info        fields[fields_count];
    u2            methods_count;
    method_info        methods[methods_count];
    u2            attributes_count;
    attribute_info    attributes[attributes_count];
}
```

*   magic,minor_version,major_version
    *   类文件版本信息和编译该类的JDK版本
*   constant_pool_count,constant_pool[constant_pool_count-1]
    *   常量池符号表 cp_info
*   access_flags
    *   提供这个类的描述符列表
*   this_class
    *   提供这个类全名的常量池(constant_pool)索引，比如org/foo/Bar
*   super_class
    *   提供这个类的父类符号引用的常量池索引
*   interfaces_count,interfaces[interfaces_count]
    *   指向常量池的索引数组，提供那些被实现的接口的符号引用
*   fields_count,fields[fields_count]
    *   提供每个字段完整描述的常量池索引数组field_info
*       
*   methods_count,methods[methods_count]
    *   指向constant_pool的索引数组，用于表示每个方法签名的完整描述。method_info
    *   如果这个方法不是抽象方法也不是 native 方法，那么就会显示这个函数的字节码
*   attributes_count,attributes[attributes_count]
    *   不同值的数组，表示这个类的附加信息，attribute_info
    *   包括 RetentionPolicy.CLASS 和 RetentionPolicy.RUNTIME 注解
