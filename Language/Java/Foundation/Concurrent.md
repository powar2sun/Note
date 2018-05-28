#   Concurrent and Parallel
![concurrentAndParallel](https://github.com/powar2sun/Note/blob/master/Language/pictures/concurrentAndParallel.png)
####    Amdahl
Amdahl 定律旨在说明，多核 CPU 对系统进行优化时，优化的效果取决于 CPU 的数量以及系统中的串行化程序的比重；
如果仅关注于提高 CPU 数量而不降低程序的串行化比重，也 无法提高系统性能。
![amdahl](https://github.com/powar2sun/Note/blob/master/Language/pictures/amdahl.png)
####    Point key
*   原子性
    *   对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行
    *   只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作
*   可见性(一致性)
    *   共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新
    *   普通的共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性
    *   synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性
#    Parallel Architecture 并行框架
*   位级并行    bit-level  
    *   32位比8位系统运行加法较快  
*   指令级并行   instruction-level
    *   包括流水线、乱序执行和猜测执行指令等
*   数据级并行   data-level
    *   单指令多数据 SIMD 架构，如 图像处理；现代GPU（图形处理器）也因图像处理的特点而演化成了极其强大的数据并行处理器
*   任务级并行   task-level
    *   共享内存的多处理器系统
![oneMemoryMultiProcessor](https://github.com/powar2sun/Note/blob/master/Language/pictures/oneMemoryMultiProcessor.png)
    *   分布式内存的多处理器系统
![multiMemoryProcesser](https://github.com/powar2sun/Note/blob/master/Language/pictures/multiMemoryProcesser.png)    

#   Process VS Thread
![processAndThread](https://github.com/powar2sun/Note/blob/master/Language/pictures/processAndThread.png)

*   内存空间
    *   进程间，独立的内存空间
    *   线程间，共享父进程的内存，各自也拥有自己的小内存
*   通信方式
    *   进程
        *   管道      Pipe(有亲缘关系进程间的通信),named pipe(无亲缘关系进程间的通信)
        *   信号      Signal用于通知接受进程有某种事件发生，除了用于进程间通信外，进程还可以发送信号给进程本身
        *   报文      Message消息队列是消息的链接表,通过读写权限的控制实现进程间通信
        *   共享内存   使得多个进程可以访问同一块内存空间，是最快的可用IPC形式
        *   信号量    Semaphore主要作为进程间以及同一进程不同线程之间的同步手段
        *   套接口    Socket可用于不同机器之间的进程间通信
    *   线程
        *   共享父进程内存空间
        *   临界区
        *   互斥量
        *   信号量
        *   事件驱动
*   上下文切换
    *   进程
        *   进程切换JVM内存空间发生变化
    *   线程                
        *   线程切换JVM内存空间不会变化
####    Thread State 线程状态
![threadLifecycle](https://github.com/powar2sun/Note/blob/master/Language/pictures/threadLifecycle.png)        
####    happens-before原则
*   次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
    *   这个规则是用来保证程序在单线程中执行结果的正确性，但无法保证程序在多线程中执行的正确性
*   锁定规则：一个unLock操作先行发生于后面对同一个锁额lock操作
    *   无论在单线程中还是多线程中，同一个锁如果被锁定的状态，那么必须先对锁进行了释放操作，后面才能继续进行lock操作
*   volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
    *   如果一个线程先去写一个变量，然后一个线程去进行读取，那么写入操作肯定会先行发生于读操作
*   传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
    *   happens-before原则具备传递性
*   线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作
*   线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
*   线程终结规则：线程中所有的操作都先行发生于线程的终止检测
*   对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法

#   Concurrent Control
####    锁
*   互斥锁 Mutex
    *   第一个切入的线程加锁后，其他竞争失败者继续回去 睡觉直到再次接到通知、竞争
        *   Java中使用synchronized紧跟同步代码块（也可修饰方法）的方式同步代码
    *   递归锁(recursive mutex)，可重入锁(reentrant mutex)
        *   同一个线程可以多次获取同一个递归锁，不会产生死锁.指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响
        *   Java的两种互斥锁实现以及读写锁实现均支持重入(如 synchronized 和 ReentrantLock等等 )
    *   非递归锁(non-recursive mutex)，不可重入锁(non-reentrant mutex)
        *   同一个线程不可多次获取同一个非递归锁，可能产生死锁
    *   读写锁
        *   支持两种模式的锁，当采用写模式上锁时与互斥锁相同，是独占模式。但读模式上锁可以被多个读线程读取，写时使用互斥锁，读时采用共享锁，故又叫共享-独 占锁
        *   共享版的互斥锁，如果对一个临界区大部分是读操作而只有少量的写操作，读写锁在一定程度上能够降低线程互斥产生的代价
    *   条件变量
        *   Condition甚至可以理解成一种“特殊”的Semaphore。特殊之处就在于它没有count（资源数）。它也有wait和signal两种行为
            *   每次condition调用wait的时候，表示它想等待某个事件的发生（不管之前有没有发生过），所以一定是加入到等待队列当中
            *   调用signal的时候，表示这个事件发生了，如果有线程在队列里等待，则取出其中一个来执行，后面的继续等待后续事件
        *   以一种无竞争的方式等待某个条件的发生。当该条件没有发生时，线程会一直处于休眠状态。当被其它线程通知条件已经发生时，线程才会被唤醒从 而继续向下执行
*   自旋锁
自旋锁与互斥锁有点类似，只是自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是 否该自旋锁的保持者已经释放了锁，"自旋"一词就是因此而得名。
其作用是为了解决某项资源的互斥使用。因为自旋锁不会引起调用者睡眠，所以自旋锁的效率远 高于互斥锁。虽然它的效率比互斥锁高，但是它也有些不足之处： 
1、自旋锁一直占用CPU，他在未获得锁的情况下，一直运行－－自旋，所以占用着CPU，如果不能在很短的时 间内获得锁，这无疑会使CPU效率降低。 
2、在用自旋锁时有可能造成死锁，当递归调用时有可能造成死锁，调用有些其他函数也可能造成死锁  
自旋锁比较适用于锁使用者保持锁时间比较短的情况。正是由于自旋锁使用者一般保持锁时间非常短，因此选择自旋而不是睡眠是非常必要的，自旋锁的效率远高于互斥锁          
*   信号量 Semophore
    *   二进制信号量(binary semaphore)：只允许信号量取0或1值，其同时只能被一个线程获取
    *   整型信号量（integer semaphore)：信号量取值是整数，它可以被多个线程同时获得，直到信号量的值变为0
    *   记录型信号量（record semaphore)：每个信号量s除一个整数值value（计数）外，还有一个等待队列List。
    *   P操作申请资源： （1）S减1； （2）若S减1后仍大于等于零，则进程继续执行； （3）若S减1后小于零，则该进程被阻塞后进入与该信号相对应的队列中，然后转入进程调度
    *   V操作 释放资源： （1）S加1； （2）若相加结果大于零，则进程继续执行； （3）若相加结果小于等于零，则从该信号的等待队列中唤醒一个等待进程，然后再返回原进程继续执行或转入进程调度
    *   （1） 测试控制该资源的信号量。 　　 （2） 若此信号量的值为正，则允许进行使用该资源。进程将信号量减1。 　　 （3） 若此信号量为0，则该资源目前不可用，进程进入睡眠状态，直至信号量值大于0，进程被唤醒，转入步骤（1）。
    *　　（4） 当进程不再使用一个信号量控制的资源时，信号量值加1。如果此时有进程正在睡眠等待此信号量，则唤醒此进程
####    悲观锁 Pessimistic Concurrency Control
如果一个事务执行的操作对某行数据应用了锁，那只有当这个事务把锁释放，其他事务才能够执行与该锁冲突的操作，
悲观并发控制主要用于数据争用激烈的环境，以及发生并发冲突时使用锁保护数据的成本要低于回滚事务的成本的环境中
*   多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题
*   一个线程持有锁会导致其它所有需要此锁的线程挂起
*   若优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险
####    乐观锁 Optimistic Concurrency Control-CAS
在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则让返回用户错误的信息，让用户决定如何去做。比较典型的就是Compare and Swap(CAS)
CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)：我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可
*   java.util.concurrent.atomic包提供对CAS操作的支持
*   ABA问题
    *   内存值原来是A，被一个线程修改为B，最后又被改成了A，则CAS认为此内存值并没有发生改变，但实际上是有被其他线程改过的，这种情况对依赖过程值的情景的运算结果影响很大
    *   通过版本号（version）的方式来解决ABA问题，乐观锁每次在执行数据的修改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1操作
*   乐观锁保证一个共享变量的原子操作，几个变量，乐观锁将变得力不从心   
*   长时间自旋可能导致开销大
####    死锁VS饥饿
*   线程在执行过程中等待锁释放，如果存在多个线程相互等待已经被加锁的资源，就会造成死锁
*   持续得不到锁的情况我们称之为饥饿,Java默认采用非公平的互斥锁
####    锁的内存模型 Lock Memory Model
*   代码重排
    *   指编译器对用户代码进行优化以提高代码的执行效率，优化前提是不改变代码的结果，即优化前后代码执行结果必须相同
    *   编译器重排是根据指令之间是否有数据依赖关系来决定的
*   指令重排
    *   处理器在执行指令期间，会把指令按照自适应处理的最优情形进行重新排序，使指令执行时间变得更短（绝大多数情形下，前提是不改变程序的执行结果）    
*   CPU缓存 Cache    
    *   由于既要满足存储容量需求又要满足CPU的高速存取需求，目前的缓存被设计为多级L1, L2 。。。。。。Ln
    *   处理器缓存执行指令结果于自己的cache上，等满足一定条件如遇到请求刷新的指令，再将缓存结果一并输出
*   内存屏障
    *   处理器提供一组机器指令来确保指令的顺序要求，它告诉处理器在继续执行前提交所有尚未处理的载入和存储指令，同样的也可以要求编译器不要对给定点以及周围指令序列进行重排    
    *   具体的确保措施在程序语言级别的体现就是内存模型的定义.内存屏障做了两件事情：拒绝重排，更新缓存.Java使用happens-before规则来屏蔽具体细节保证，指导JVM在指令生成的过程中穿插屏障指令
    
 ####  多线程三概念
 *  原子性
    *   一个操作(可能包含自操作)要么全部执行(生效),要么全部不执行(不生效)
 *  可见性
    *   多线程共享变量时,一个线程对共享变量的修改,其它线程能够立即看到
        *   即CPU的缓存主机内存的数据一致性问题
 *  顺序性
    *   程序执行的顺序按照代码的先后顺序执行
        *   CPU代码优化,可能对指令重排序.但要保证单线程执行结果和代码结果的最终一致性
        
####    Java如何保证原子性
*   锁和同步
```
// 上锁
lock.lock();
...
// 放锁
lock.lock(); 
nlock();
//同步
synchronized(anyObject){
    ...
}

synchronized 非静态方法  锁住当前实例
synchronized 静态方法   锁住类对象Class
synchronized 代码块    锁住(anyObject)
```
*   CAS   
```
AtomicInteger ai=new AtomicInteger()
ai.incrementAndGet();
```

####    Java如何保证可见性
```
volatile variable;
```

####    Java如何保证顺序性
```
1.volatile能够保证一定的顺序性
2.synchronized和lock,将执行单元单线程化
3 Happens-Before原则
    传递原则
        A操作在B操作之前,B操作在C操作之前,那么A操作肯定在C操作之前
    锁定原则
        unlock肯定发生获取在同一个锁lock之前
    volatile原则
        对volatile变量的读操作发生在写操作之前
    程序次序原则
        一个线程内,按照代码顺序执行
    线程启动原则
        线程的start()先发生于此线程的其它动作
    线程终结原则
        线程的终止操作先发生于其它操作之前
    线程中断原则
        线程的interrupt()先发生于对该中断异常的获取
    对象终结原则
        一个对象的构造先发生于它的finalize
```

####    volatile使用场景
*   无须保证原子性,但却需要可见性的场景.如停止线程的FLAG
```
// 如果不用volatile修饰,即使其它处改变了running,该线程也不一定立即终止
volatile boolean running = true;
new Thread(()->{
    while(running){
        ...
    }
}).start()

public void stop(){
    running = false;
}
```

####    其它保证线程安全的方法
```
1. ThreadLocal
2. 不可变对象
3. 避免多线程的共享变量
```

####    sleep() VS wait()
wait()  Object类方法       暂时放锁    等待其它线程对同一对象的notify()激活  进入就绪队列

```
/*
执行结果
    A is running
    B is running
    B release lock
    A is ended
    B is ended
*/

public class WaitDemo{
    public static void main(String[] args){
        Thread threadA = new Thread(()->{
            synchronized (WaitDemo.class){
                System.out.println(new Date()+"A is running");
                WaitDemo.class.wait();
                System.out.println(new Date()+"A is ended");
            }
        });
        threadA.start();
        Thread threadB = new Thread(()->{
            synchronized(WatiDemo.class){
                System.out.println(new Date()+"B is ended");
                WaitDemo.class.notify();
                
                 // 延长执行时间
                 // B在notify()之后,A并未执行,因为A还没有得到锁,只是进入该锁的等待队列
                for(long i=0;i<200000;i++){
                    for(long j=0;j<200000;j++){}
                }
                System.out.println(new Date()+"B relases lock");
                // 拖延时间
                for(long i=0;i<200000;i++){
                    for(long j=0;j<200000;j++){}
                }
                System.out.println("B is ended");
            }
        });
        
        // 不使用sleep()延长执行时间
        for(long i=0;i<200000;i++){
            for(long j=0;j<200000;j++){}
        }
    
        threadB.start();
    }
}
```

sleep() Thread类静态方法   不会放锁     实际上并不要求有锁,若有锁也不放,暂时让出CPU时间片

```
/*
执行结果
    A is running
    A is ended
    B is running
    B is ended
*/
public class SleepDemo{
    public static void main(String[] args){
        Thread threadA = new Thread(()->{
            synchronized (SleepDemo.class){
                System.out.println(new Date()+"A is running");
                Thread.sleep(2000); // A拿到锁睡眠,但并未释放锁,其它线程无法执行.只有等待该线程结束后放锁
                System.out.println(new Date()+"A is ended");
            }
        });
        threadA.start();
        
        Thread threadB = new Thread(()->{
            synchronized(SleepDemo.class){
                System.out.println(new Date()+"B is ended");
                Thread.sleep(2000);
                System.out.println("B is ended");
            }
        });
        
        // 不使用sleep()延长执行时间
        for(long i=0;i<200000;i++){
            for(long j=0;j<200000;j++){}
        }
    
        threadB.start();
    }
}
```

*   对象锁(内置锁)
    *   每个类对象都有一把内置锁
    *   同步实例方法获取的是该对象的内置锁
    *   若多线程调用同一实例的同步方法需要竞争锁
    *   若多线程调用不同实例的同步方法并不竞争锁
*   类级锁(静态锁)
    *   类只有一把Class锁
    *   多线程调用同一个类的不同静态方法会产生锁竞争
    
####    锁对象
*   重入锁
    *   与内置锁一样是排它锁
        *   公平锁与非公平锁,构造函数时可指定
        *   tryLock()
```
Lock lock = new ReentrantLock(true);
lock.lock();
...
lock.unlock();
```    
*   读写锁
    *   多线程的读无需锁,读写和写写需要锁,ReadWriteLock使用此场景
        *   ReentrantReadWriteLock有两个静态内部类 ReadLock和WriteLock
        *   获得读锁后(readLock()),其它线程可以获取读锁但不能获取写锁
        *   获得写锁后(writeLock()),其它线程既不能获取读锁也不能获取写锁
```
//A 和 B 获取的是读锁,可以并行执行,C 要获取写锁,只能等A和B释放后才能获得
public class Demo{
    public static void main(String[] args){
        ReadWriteLock rwl = new ReentrantReadWriteLock();
        
        new Thread(()->{
            rwl.readLock().lock();
            sout("A started with read lock");
            Thread.sleep(3000);
            sout("A ended");
            rwl.readLock().unlock();
        }).start();    
        
        new Thread(()->{
            rwl.readLock().lock();
            sout("B started with read lock");
            Thread.sleep(3000);
            sout("B ended");
            rwl.readLock().unlock();
        }).start();
        
        new Thread(()->{
            Lock lock = rwl.writeLock();
            lock.lock();
            sout("C started with write lock");
            Thread.sleep(1000);
            sout("C ended");
            lock.unlock();
        }).start();
    }
}
```

*   条件锁
    *   理解概念,每个可重入锁都可以newCondition()绑定若干个条件的对象
        *   await() <--> signal(),signalAll()
        *   调用上述方法的前提是已经获得该条件对象对应的重入锁
        *   signal()和signalAll()只能唤醒相同条件对象的等待
        *   一个重入锁上可有多个条件变量，不同线程可等待不同的条件，更加细粒度的的线程间通信
```
public class ConditionTest{
    public static void main(String[] args){
        Lock lock=new ReentrantLock();
        Condition condition=lock.newCondition();
        
        new Thread(()->{
            lock.lock();
            sout
        }).start();
    }
}
```


####    信号量    
信号量维护一个许可集，可通过acquire()获取许可（若无可用许可则阻塞），通过release()释放许可，从而可能唤醒一个阻塞等待许可的线程。
与互斥锁类似，信号量限制了同一时间访问临界资源的线程的个数，并且信号量也分公平信号量与非公平信号量。
而不同的是，互斥锁保证同一时间只会有一个线程访问临界资源，而信号量可以允许同一时间多个线程访问特定资源。
所以信号量并不能保证原子性。      
信号量的一个典型使用场景是限制系统访问量。
每个请求进来后，处理之前都通过acquire获取许可，若获取许可成功则处理该请求，若获取失败则等待处理或者直接不处理该请求
*   acquire()
*   release()

####    线程间通信
*   CountDownLatch
    *   场景:某个线程需要等待一个或多个线程操作结束,才开始执行
```
public class CountDownLatchDemo{
    public static void main(String[] args){
    int totals = 3;
    CountDownLatch countDown = new CountDownLatch(total);
    long start=System.currentTimeMillis();
    for(int i=0;i<total;i++){
        new Thread(()->{
            sout(String.format("",new Date(),"Thread-"+i,"started"));
            Thread.sleep(2000);
            coutDown.countDown();
            sout(String.format("%s\t%s %s",new Date(),"Thread-"+i,"ended"));
        }).start();
        
        countDown.await();
        long stop=System.currentTimeMillis();
        sout("Total Cost time is: %s ms",(stop-start));
    }
    
    }
}
```
*   CyclicBarrier
    *   CyclicBarrier是让多个线程互相等待某一事件的发生，然后同时被唤醒。
    而上文讲的CountDownLatch是让某一线程等待多个线程的状态，然后该线程被唤醒
    *   每个线程都不会在其它所有线程执行await()方法前继续执行，
    而等所有线程都执行await()方法后所有线程的等待都被唤醒从而继续执行
```
public class CyclicBarrierDemo{
    public static void main(String[] args){
        int totalThread = 5;
        CyclicBarrier barrier=new CyclicBarrier(totalThread);
        for(int i=0;i<totalThread;i++){
            String threadName="Thread"+i;
            new Thread(()->{
                sout(String.format("%s\t%s %s",new Date(),threadName,"is waiting"));
                barrier.await();
                System.out.println(String.format("%s\t%s %s", new Date(), threadName, "ended"));
            }).start();
        }
    }
}
```    
*   Phaser
    *   一种任务可以分为多个阶段，现希望多个线程去处理该批任务，
    对于每个阶段，多个线程可以并发进行，但是希望保证只有前面一个阶段的任务完成之后才能开始后面的任务
    *   多个线程必须等到其它线程的同一阶段的任务全部完成才能进行到下一个阶段，
    并且每当完成某一阶段任务时，Phaser都会执行其onAdvance方法
```
public class PhaserDemo {
  public static void main(String[] args) throws IOException {
    int parties = 3;
    int phases = 4;
    final Phaser phaser = new Phaser(parties) {
      @Override  
      protected boolean onAdvance(int phase, int registeredParties) {  
          System.out.println("====== Phase : " + phase + " ======");  
          return registeredParties == 0;  
      }  
    };
    
    for(int i = 0; i < parties; i++) {
      int threadId = i;
      Thread thread = new Thread(() -> {
        for(int phase = 0; phase < phases; phase++) {
          System.out.println(String.format("Thread %s, phase %s", threadId, phase));
          phaser.arriveAndAwaitAdvance();
        }
      });
      thread.start();
    }
  }
}

```    
