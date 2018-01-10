##  JVM Garbage Collection
垃圾回收算法可以分为两部分

*   对象的查找算法
*   真正的回收方法

主流的四个垃圾回收器

*   Serial GC
*   Throughput/Parallel GC
*   CMS GC
*   G1 GC

|分类标准|相应描述|
|--|--|
|线程数|<li>串行垃圾回收器:一次只使用一个线程进行垃圾回收<li>并行垃圾回收器:开启多个线程同时进行垃圾回收,可以缩短 GC 的停顿时间。|
|工作模式|<li>并发式垃圾回收器:与应用程序线程交替工作，以尽可能减少应用程序的停顿时间<li>独占式垃圾回收器:停止应用程序中的其他所有线程，直到垃圾回收过程结束|
|碎片处理方式|<li>压缩式垃圾回收器:会在回收完成后，对存活对象进行压缩整理，消除回收后的碎片<li>非压缩式垃圾回收器|
|工作的内存区间|<li>新生代垃圾回收器<li>老年代垃圾回收器|

|GC性能指标|相应描述|
|--|--|
|吞吐量|<li>系统总运行时间=应用程序耗时+GC 耗时<li>指在应用程序的生命周期内，应用程序所花费的时间和系统总运行时间的比值|
|停顿时间|指垃圾回收器正在运行时，应用程序的暂停时间。|
|垃圾回收器负载|和吞吐量相反，垃圾回收器负载指来记回收器耗时与系统运行总时间的比值|
|垃圾回收频率|<li>指垃圾回收器多长时间会运行一次<li>通常增大堆空间可以有效降低垃圾回收发生的频率，但是可能会增加回收产生的停顿时间|
|反应时间|指当一个对象被称为垃圾后多长时间内，它所占据的内存空间会被释放|

####    对象引用
*   StrongReference: 强引用
    *   使用最普遍的引用，垃圾回收器不会回收它。当内存空间不足时，JVM不会靠随意回收强引用的对象来解决内存不足的问题，而是抛出 OutOfMemoryError
    *   要注意的是当方法运行完毕，强引用对象已不存在了，所以它们指向的对象都会被 JVM 回收
    *   若想中断强引用和某个对象之间的关联，显式将引用赋值为null，JVM在合适的时间就会回收该对象。
*   SoftReference: 软引用
    *   软引用是用来描述一些有用但并不是必需的对象，用 java.lang.ref.SoftReference 类来表示
    *   软引用关联着的对象，只有在内存不足的时候 JVM 才会回收该对象
    *   这个特性很适合用来实现缓存：比如网页缓存、图片缓存等

```
SoftReference<String> sr = new SoftReference<String>(new String("hello"));
```

*   WeakReference: 弱引用
    *   弱引用的对象拥有更短暂的生命周期,GC扫描它所管辖的内存区域时一旦发现弱引用的对象，不管当前内存空间足够与否，都会回收它的内存
    *   由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象
    
```
WeakReference<String> sr = new WeakReference<String>(new String("hello"));
```

*   PhantomReference: 虚引用
    *   虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收
    *   要注意的是:虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之关联的引用队列中

```
ReferenceQueue<String> queue = new ReferenceQueue<String>();
PhantomReference<String> pr = new PhantomReference<String>(new String("hello"), queue);
```      

####    对象存活判断
由于引用计数法无法解决对象循环引用的问题，因此主流的 JVM 倾向于使用可达性分析

*   Reference Counting: 引用计数
    *   此算法Cons：垃圾对象间相互引用，从而使垃圾回收器无法识别，引起内存泄漏
*   引用树遍历
    *   沿着对象的根句柄向下查找到活着的节点，并标记下来；其余没有被标记的节点就是死掉的节点，这些对象就是可以被回收的
    *   那些发现不能到达 GC Roots 的对象并不会立即回收，在真正回收之前，对象至少要被标记两次
        *   当第一次被发现不可达时，该对象会被标记一次，同时调用此对象的 finalize()方法（如果有）
        *   第二次被发现不可达后，对象被回收。利用 finalisze() 方法，对象可以逃离一次被回收的命运，但是只有一次

```
//逃命钩子，Once
public class EscapeFromGC(){
   public static EscapeFromGC hook;
   @Override
   protected void finalize() throws Throwable {
      super.finalize();
      System.out.println("finalize mehtod executed!");
      EscapeFromGC.hook = this;
}
```

####    通用垃圾回收算法

|which|Prons|Cons|
|--|--|--|
|Mark-Sweep / 标记-清除|简单<li>标记-清除算法将垃圾回收分为两个阶段：标记阶段和清除阶段|效率低下且会产生很多不连续内存，分配大对象时，容易提前引起另一次垃圾回收|
|Copying / 复制|效率较高，不用考虑内存碎片化<li>现有的内存空间分为两快，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收<li>Java 的新生代串行垃圾回收器中使用了复制算法的思想。新生代分为 eden 空间、from 空间、to 空间 3 个部分。其中 from 空间和 to 空间可以视为用于复制的两块大小相同、地位相等，且可进行角色互换的空间块|存在空间浪费|
|Mark-Compact / 标记-整理(压缩)<li>标记-压缩算法是一种老年代的回收算法，它在标记-清除算法的基础上,将所有的存活对象压缩到内存的一端|避免了内存碎片化|GC 暂停时间增长,因为你需要将所有的对象都拷贝到一个新的地方，还得更新它们的引用地址|

####    实用垃圾回收算法

|what|how|
|--|--|
|Incremental Collecting: 增量回收算法|<li>GC线程和用户线程交替执行，一次只回收一片区域取代一次性回收全部垃圾造成的长时间停顿<li>虽然减少了停顿时间，但是线程上下文切换的成本增加|
|Generational Collecting: 分代回收算法|<li>根据待回收对象的特性，不同阶段采用不同回收策略的分代执行回收算法<li>HotSpot在年轻代采用标记-复制算法，在年老代采用标记-压缩算法|
|Concurrent Collecting: 并发回收算法||

####    垃圾回收器对比

|what|when|how|why|setting|
|--|--|--|--|--|
|Serial|Serial GC 作用于新生代，Serial Old GC 作用于老年代垃圾收集|二者皆采用了串行回收与 "Stop-the-World"，Serial 使用的是标记-复制算法，Serial Old使用的是标记-压缩算法|基于串行回收的垃圾回收器适用于大多数对于暂停时间要求不高的 Client 模式下的 JVM|使用 -XX:+UserSerialGC 手动指定使用 Serial 回收器执行内存回收任务|
|Throughput/Parallel|Parallel 作用于新生代，Parallel Old 作用于老年代|并行回收和 "Stop-the-World"，Parallel 使用的是标记-复制算法，Parallel Old 使用的是标记-压缩算法|程序吞吐量优先的应用场景中，在 Server 模式下内存回收的性能较为不错|使用 -XX:+UseParallelGC 手动指定使用 Parallel 回收器执行内存回收任务|
|CMS,Concurrent-Mark-Sweep|老年代垃圾回收器，又称作 Mostly-Concurrent 回收器|使用了标记清除算法，分为初始标记( Initial-Mark，Stop-the-World )、并发标记( Concurrent-Mark )、再次标记( Remark，Stop-the-World )、并发清除( Concurrent-Sweep )|并发低延迟，吞吐量较低。经过CMS收集的堆会产生空间碎片，会带来堆内存的浪费|使用 -XX:+UseConcMarkSweepGC 来手动指定使用 CMS 回收器执行内存回收任务|
|G1,Garbage First|没有采用传统物理隔离的新生代和老年代的布局方式，仅仅以逻辑上划分为新生代和老年代，选择的将 Java 堆区划分为 2048 个大小相同的独立 Region 块|使用了标记压缩算法|基于并行和并发、低延迟以及暂停时间更加可控的区域化分代式服务器类型的垃圾回收器|使用 -XX:UseG1GC 来手动指定使用 G1 回收器执行内存回收任务|

*   开始进行标记前暂停应用线程以便JVM可以尽快执行，称之为安全点（Safe Point），这会触发一次Stop The World(STW)暂停。
*   暂停时间的长短并不取决于堆内对象的多少也不是堆的大小，而是存活对象的多少。因此，调高堆的大小并不会影响到标记阶段的时间长短
*   当标记阶段完成后，GC开始进入下一阶段，删除不可达对象

####    Serial GC
*   使用单线程进行垃圾回收
*   独占式垃圾回收
*   没有线程切换的开销，在较小应用或硬件平台限制的场合，其性能超过并行回收器和并发回收器

![serialGC](https://github.com/powar2sun/Note/blob/master/Language/pictures/serialGC.png)

####    ParNew GC
*   并行回收器,简单地将串行回收器多线程化
*   并行回收器 也是独占式的回收器
*   由于并行回收器使用多线程进行垃圾回收,在并发能力比较强的CPU上，它产生的停顿时间要短于串行回收器
*   在单 CPU 或者并发能力较弱的系统中，并行回收器的效果不会比串行回收器好

####    Parallel GC
*   Parallel Scavenge 收集器的目标则是达到一个可控制的吞吐量（Throughput）
*   在对吞吐量敏感的系统中，可以考虑使用。参数-XX:ParallelGCThreads用于设置垃圾回收时的线程数量

![parallelGC](https://github.com/powar2sun/Note/blob/master/Language/pictures/parallelGC.png)

####    CMS GC
*   CMS( Concurrent Mark-Sweep ) 是以牺牲吞吐量为代价来获得最短回收停顿时间的垃圾回收器
*   适用于对停顿比较敏感，并且有相对较多存活时间较长的对象（老年代较大）的应用程序
*   CMS 虽然减少了回收的停顿时间，但是降低了堆空间的利用率,会产生空间碎片
*   默认当老年代使用68%的时候，CMS就开始行动了。 – XX:CMSInitiatingOccupancyFraction =n 来设置这个阀值

![cmsGC](https://github.com/powar2sun/Note/blob/master/Language/pictures/cmsGC.png)

####    G1 GC
*   G1 GC是JDK 1.7中正式投入使用的用于取代 CMS 的压缩回收器，仍然属于分代垃圾回收器
*   G1 GC 仍然会区分年轻代与老年代，年轻代依然分有 Eden 区与 Survivor 区
*   首先将堆分为大小相等的 Region，然后追踪每个 Region 垃圾堆积的价值大小，在后台维护一个优先列表，根据允许的收集时间优先回收价值最大的Region
*   同时 G1 GC 采用 Remembered Set 来存放 Region 之间的对象引用以及其他回收器中的新生代与老年代之间的对象引用，从而避免全堆扫描

![g1GC](https://github.com/powar2sun/Note/blob/master/Language/pictures/g1GC.png)

随着 G1 GC 的出现，Java 垃圾回收器通过引入 Region 的概念，从传统的连续堆内存布局设计，逐步走向了物理上不连续但是逻辑上依旧连续的内存块；
这样能够将某个 Region 动态地分配给 Eden、Survivor、老年代、大对象空间、空闲区间等任意一个。
每个 Region 都有一个关联的 Remembered Set（简称RS），RS 的数据结构是 Hash 表，里面的数据是 Card Table （堆中每 512byte 映射在 card table 1byte）。
简单的说RS里面存在的是Region中存活对象的指针。当Region中数据发生变化时，首先反映到Card Table中的一个或多个Card上，RS通过扫描内部的Card Table得知Region中内存使用情况和存活对象。
在使用Region过程中，如果Region被填满了，分配内存的线程会重新选择一个新的Region，空闲Region被组织到一个基于链表的数据结构（LinkedList）里面，这样可以快速找到新的Region

*   G1 GC 的特性如下
    *   并行性：G1在回收期间，可以有多个GC线程同时工作，有效利用多核计算能力
    *   并发性：G1拥有与应用程序交替执行的能力，部分工作可以和应用程序同时执行
    *   分代GC：G1依然是一个分代回收器，但是和之前的各类回收器不同，它同时兼顾年轻代和老年代
    *   空间整理：G1在回收过程中，会进行适当的对象移动，不像CMS只是简单地标记清理对象
    *   可预见性：为了缩短停顿时间，G1建立可预存停顿的模型，这样在用户设置的停顿时间范围内，G1会选择适当的区域进行收集，确保停顿时间不超过用户指定时间
    
![g1GCflow](https://github.com/powar2sun/Note/blob/master/Language/pictures/g1GCflow.png)    


*   初始标记（标记一下GC Roots能直接关联的对象并修改TAMS值，需要STW但耗时很短）
*   并发标记（从GC Root从堆中对象进行可达性分析找存活的对象，耗时较长但可以与用户线程并发执行）
*   最终标记（为了修正并发标记期间产生变动的那一部分标记记录，这一期间的变化记录在Remembered Set Log 里，然后合并到Remembered Set里，该阶段需要STW但是可并行执行）
*   筛选回收（对各个Region回收价值排序，根据用户期望的GC停顿时间制定回收计划来回收）；













