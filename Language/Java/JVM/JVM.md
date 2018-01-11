##	JVM
####    内存模型
![jvmModel](https://github.com/powar2sun/Note/blob/master/Language/pictures/jvmModel.png)
####    模型参数
![jvmSize](https://github.com/powar2sun/Note/blob/master/Language/pictures/jvmSize.png)
####    方法区 Method Area
![methodArea](https://github.com/powar2sun/Note/blob/master/Language/pictures/methodArea.png)
####    栈 Stack
![stack](https://github.com/powar2sun/Note/blob/master/Language/pictures/stack.png)
####    Book for understanding JVM
http://book.51cto.com/art/201107/278857.htm

*  GC 示例
   *  https://www.devh.net/yidongnan/blog/4v312p6ben9jnbg403tsnkvgan

####    内存分配
![jvmStorage](https://github.com/powar2sun/Note/blob/master/Language/pictures/jvmStorage.png)

*   Eden:新生代优先分配新生成对象
    *   Java中常见的创建对象的方式有使用new关键字进行创建，也可以使用java.lang.Class.forName进行动态加载
    *   JVM中会将Main类以及通过import引入的关联项形成JVM网状结构，保证每个Class都有一份自己的私有池并且放在PermSize或者MetaSpace中
    *   动态装载负责的是运行时装载一些类的定义，而不是初始化。先从符号向量中寻找这个类是否已经加载，如果已经加载则直接使用，否则从相应的包中获取这个Class定义，然后装载起来，装载的单位也是以Class为单位，并不是以jar包为单位
*   Tunured:大对象直接进入老年代
    *   比遇到一个大对象更加坏的消息就是遇到一群“朝生夕灭”的“短命大对 象”，写程序的时候应当避免
    *   JVM提供了一个-XX:PretenureSizeThreshold参数，令大于这个设置值的对象直接在老年代中分配。目的是避免在Eden区及两个Survivor区之间发生大量的内存拷贝
    *   注意PretenureSizeThreshold参数只对Serial和ParNew两款收集器有效，Parallel Scavenge收集器不认识这个参数,如遇必须使用此参数的场合，可以考虑ParNew加CMS的收集器组合
*   Tunered:长期存活的对象进入老年代
    *   JVM给每个对象定义了一个对象年龄（Age）计数器。如果对象在Eden出生并经过第一次Minor GC后仍然存活，将被移动到Survivor空间(能容纳的情况下)，并将对象年龄+1。每熬过一次 Minor GC，Age就+1岁，当它的年龄增加到一定程度（默认15）时，就会被晋升到老年代中。
    *   对象晋升老年代的年龄阈值，通过参数 -XX:MaxTenuringThreshold来设置。
    *   动态对象年龄的晋升策略
        *   在eden和survivor中可以来回被minor gc多次，这个次数超过了-XX:MaxTenuringThreshold
        *   在发生minor gc时，发现to survivor无法放下这些对象，就会进入old
        *   新申对象，大于eden区域的一半大小时直接进入old，也可设置参数-XX:PretenureSizeThreshold指定当超过这个值就直接进入old
        *   为更好地适应不同程序的内存状况并不总是要求对象的年龄必须达到MaxTenuringThreshold才能晋升老年代，如果在 Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到 MaxTenuringThreshold中要求的年龄
*   空间分配担保
    *   在发生Minor GC时，虚拟机会检测之前每次晋升到老年代的平均大小是否大于老年代的剩余空间大小，如果大于，则改为直接进行一次Full GC
    *   GC会将HandlePromotionFailure开关打开，避免Full GC过于频繁。若出现了HandlePromotionFailure失败，那就只好重新发起一次Full 
*   GC：两大GC
    *   大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够的空间进行分配时，虚拟机将发起一次Minor GC
    *   虚拟机提供了-XX:+PrintGCDetails这个收集器日志参数，让JVM打印内存回收日志，并且在进程退出的时候 输出当前内存各区域的分配情况
    *   新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快
    *   年代GC（Major GC / Full GC）：指发生在老年代的GC，其经常会伴随至少一次（并非绝对）的Minor GC。它的速度一般会比Minor GC慢10倍以上
    
####    对象访问
*   Object Access
    *   使用句柄池
        *   reference中存储的是稳定的句柄地址，在对象被移动（GC时非常普遍）时只会改变句柄中的实例数据指针，而reference本身不用被修改     
    *   直接引用
        *   节省了一次指针定位的时间开销，由于对象的访问在Java中非常频繁，因此这类开销积少成多后也是 一项非常可观的执行成本

![objAcc1](https://github.com/powar2sun/Note/blob/master/Language/pictures/objectAccessOne.png)
![objAcc2](https://github.com/powar2sun/Note/blob/master/Language/pictures/objectAccessTwo.png)

*   Object Structure
    *   对象头
        *   32位8字节  System.out.print(sizeOf(new Object()));
        *   64位16字节 System.out.print(sizeOf(new Object()));
    *   实例数据
        *   boolean,byte  -(Memory required)-> 1 bytes
        *   short,char    -(Memory requierd)-> 2 bytes
        *   int,float     -(占用)->   4 字节
        *   long,double   -(占用)->   8 字节
        *   reference类型  32位占4字节，64位占8字节
    *   对其填充
        *   HotSpot的对齐方式为8字节对齐
        *   (对象头 + 实例数据 + padding) % 8等于0;  且0 <= padding < 8
        
####    指针压缩
对象占用的内存大小收到VM参数UseCompressedOops的影响

*   对象头压缩 开启（-XX:+UseCompressedOops）对象头大小为12bytes（64位机器）

```
static class A{
    int a;
}
//关闭指针压缩： 16+4=20 不是8的倍数，-(+padding)-> =24
//开启指针压缩： 12+4=16 已是8的倍数，-(无需pandding)-> =16
```
*   64位机器上，数组对象的对象头占用24个字节，启用压缩之后占用16个字节.比普通对象占用内存多是因为需要额外的空间存储数组的长度






















