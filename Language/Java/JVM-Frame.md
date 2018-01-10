##  架构与机制
####    Fraem
![archtecture](https://github.com/powar2sun/Note/blob/master/Language/pictures/archtecture.png)
####    Mechanism
![structure](https://github.com/powar2sun/Note/blob/master/Language/pictures/structure.png)
####    Reference
*   Tutorial
    *   http://www.importnew.com/17770.html
    *   http://www.hollischuang.com/archives/1003
    *   https://www.tuicool.com/articles/BVz2qqq
*   Practice
    *   http://dbaplus.cn/news-21-173-1.html
*   Books
    *   http://7xkt0f.com1.z0.glb.clouddn.com/%5B%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E8%99%9A%E6%8B%9F%E6%9C%BA%EF%BC%9AJVM%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E4%B8%8E%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%5D.%E5%91%A8%E5%BF%97%E6%98%8E.%E9%AB%98%E6%B8%85%E6%89%AB%E6%8F%8F%E7%89%88.pdf
    
####    JVM 类型
*   Client
    *   使用的是一个代号为C1的轻量级编译器
    *   适合需要快速启动和较小内存空间的应用，它适合交互性的应用，比如GUI
    *   Client VM的编译器没有像Server VM一样执行许多复杂的优化算法
*   Server
    *   采用相对重量级,代号为C2的编译器
    *   重执行效率的应用的最佳选择。不同之处包括：编译策略、默认堆大小、内嵌策略
    *   该编译器支持许多和在C++编译器上执行的一样的优化，同时还包括许多传统的编译器无法实现的优化
    
####    JVM 工作模式
`java -X`

*   Xint代表解释模式(interpreted mode)，强制JVM以解释方式执行所有的字节码。降低运行速度10倍或更多
*   Xcomp代表编译模式(compiled mode)，会把所有的字节码编译成本地代码，没有让JVM启用JIT编译器的全部功能
*   Xmixed代表混合模式(mixed mode)，默认工作模式，同时使用编译模式和解释模式。多次被调用的部分将其编译成本地代码以提高执行效率。调用很少的方法在解释模式下会继续执行，从而减少编译和优化成本

工作模式
*   设置
    *   `java -server -Xint -version`
    *   `java -client -Xcomp -version`
*   获取
    *   `java -version`
    *   `System.out.print(System.getProperty("java.vm.name"))`
    *   `System.out.print(System.getProperty("java.vm.info"))`
    
####    JVM Thread
们会关注到线程独有的譬如内存区域中的程序计数器、虚拟机栈、本地栈和栈帧、局部变量数组、操作数栈、动态链接，以及线程共享的堆、非堆内存、内存管理、
即时编译JIT、方法区、类文件结构、类加载器、运行时常量池、异常表、符号表以及Interned字符串等等,如果使用 jconsole 或者其它调试器，你会看到很多线程在后台运行

|线程类型|线程说明|
|--|--|
|虚拟机线程（VM thread）|这个线程等待 JVM 到达安全点操作出现。这些操作必须要在独立的线程里执行，因为当堆修改无法进行时，线程都需要 JVM 位于安全点。这些操作的类型有：stop-the-world 垃圾回收、线程栈 dump、线程暂停、线程偏向锁（biased locking）解除|
|周期性任务线程|这线程负责定时器事件（也就是中断），用来调度周期性操作的执行|
|GC 线程|这些线程支持 JVM 中不同的垃圾回收活动|
|编译器线程|在运行时将字节码动态编译成本地平台相关的机器码|
|信号分发线程|这个线程接收发送到 JVM 的信号并调用适当的 JVM 方法处理|

####    JVM 常用参数
*   -Xms为初始化为HeapSize的空间，即被Commited的尺寸
*   -Xmx为最大的HeapSize空间，有些尚未被Commited，但是已经被进程所Reserved
*   -Xmn设置Young的空间大小，此时NewSize和MaxNewSize一致
    *   -XX:NewSize=128m 
    *   -XX:PermSize = 64M
    *   -XX:MaxPermSize= 64M
*   -XX:NewRatio= 3 为Tenured:Young的初始尺寸比例（设置了大小就不再设置此值），此时Young占用整个HeapSize的1/4大小  
*   -XX:SurvivorRatio= 6：为Eden:Survivor比例大小，此时一个Survivor占用Young的1/8大小
*   -Xss=256k为ThreadStack空间大小，jdk 1.5以后默认是1M
*   -XX:MaxTenuringThreshold=3：一般一个对象在Young经过多少次GC后会被移动到OLD区
*   回收算法的参数（+用/-不用）
    *   -XX:+UseParNewGC：对Yong区域启用并行回收算法
    *   -XX:-UseParallelGC：一种较老的并行回收算法-表示不用
    *   -XX:+UseParallelOldGC：对Tenured区域使用并行回收算法
    *   -XX:ParallelGCThread=10：并行的个数，一般和CPU个数相对应
    *   -XX:+UseAdaptiveSizepollcy：收集器自动根据实际情况进行一些比例以及回收算法调整
    *   -XX:CMSFullGCsBeforeCompaction= 3：多少次GC后会进行压缩碎片
    *   -XX:+UseCmsFullCompactAtFullCollction：打开老年代压缩 以下3个参数为永久带的回收参数
        *   -XX:+UseConcMarkSweepGC
        *   -XX:+CMSClassUnloadingEnabled
        *   -XX:+CMSPermGenSweepingEnabled
    *   -XX:MinHeapFreeRatio这是指剩余空间百分比多少时，开始减小commited的内存        
    *   -XX:MaxHeapFreeRatio指剩余空间百分比多少时，开始增加commited的内存，直到-Xmx大小
    *   -XX:MaxGCPauseMillis指GC最大的暂停时间，当超过这个时间，那么JVM会适当调整内存比例（前提是使用的是基于比例的YONG和设置）
    *   -XX:+UseConcMarkSweepGC 启动并发GC，一般针对Tenured区域
    *   -XX:+CMSIncrementalMode增量GC，将内存切块，分布在多个局部去GC
-XX:CMSInitiatingOccupancyFraction在并发GC下，由于一边使用，一边GC，就不能在不够用的时候GC，默认情况下是在使用了68%的时候进行GC
    
    
    
    
    
    
    
    
    
    
    
