---
title: Java并发编程

date: 2020-5-21 22:20:12  

categories: 2020年5月

tags: [Java, Concurrency]

---

介绍Java并发编程中的常用方法，如AtomicInteger，信号量Semaphore，这些方法的实现原理和源码值得深究。

<!-- more -->


# Java原子操作AtomicInteger

JDK1.5之后的java.util.concurrent.atomic包里，多了一批原子处理类。AtomicBoolean、AtomicInteger、AtomicLong、AtomicReference。主要用于在高并发环境下的高效程序处理,来帮助我们简化同步处理.

AtomicInteger，一个提供原子操作的Integer的类。在Java语言中，++i和i++操作并不是线程安全的，在使用的时候，不可避免的会用到synchronized关键字。而AtomicInteger则通过一种线程安全的加减操作接口。

我们先来看看AtomicInteger给我们提供了什么接口:

        public final int get() //获取当前的值
    public final int getAndSet(int newValue)//获取当前的值，并设置新的值
    public final int getAndIncrement()//获取当前的值，并自增
    public final int getAndDecrement() //获取当前的值，并自减
    public final int getAndAdd(int delta) //获取当前的值，并加上预期的值

下面通过两个简单的例子来看一下 AtomicInteger 的优势在哪:

普通线程同步:
        
        
        class Test2 {
                private volatile int count = 0;
        
                public synchronized void increment() {
                          count++; //若要线程安全执行执行count++，需要加锁
                }
        
                public int getCount() {
                          return count;
                }
        }
使用AtomicInteger:

        
        class Test2 {
                private AtomicInteger count = new AtomicInteger();
        
                public void increment() {
                          count.incrementAndGet();
                }
           //使用AtomicInteger之后，不需要加锁，也可以实现线程安全。
               public int getCount() {
                        return count.get();
                }
        }


使用AtomicInteger是非常的安全的.而且因为AtomicInteger由硬件提供原子操作指令实现的。在非激烈竞争的情况下，开销更小，速度更快。

我们来看看AtomicInteger是如何使用非阻塞算法来实现并发控制的:
AtomicInteger的关键域只有一下3个：

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
    static {    
             try {        
                    valueOffset = unsafe.objectFieldOffset (AtomicInteger.class.getDeclaredField("value"));   
            } catch (Exception ex) { 
                   throw new Error(ex); 
            }
        }
    private volatile int value;
    
这里， unsafe是java提供的获得对对象内存地址访问的类，注释已经清楚的写出了，它的作用就是在更新操作时提供“比较并替换”的作用。实际上就是AtomicInteger中的一个工具。
valueOffset是用来记录value本身在内存的便宜地址的，这个记录，也主要是为了在更新操作在内存中找到value的位置，方便比较。
注意：value是用来存储整数的时间变量，这里被声明为volatile，就是为了保证在更新操作时，当前线程可以拿到value最新的值（并发环境下，value可能已经被其他线程更新了）。
这里，我们以自增的代码为例，可以看到这个并发控制的核心算法：
    
    /**
    *Atomicallyincrementsbyonethecurrentvalue.
    *
    *@returntheupdatedvalue
    */
    publicfinalintincrementAndGet(){
    for(;;){
        //这里可以拿到value的最新值
        intcurrent=get();
        intnext=current+1;
    if(compareAndSet(current,next))
        returnnext;
        }
    }
    
    publicfinalbooleancompareAndSet(intexpect,intupdate){
    //使用unsafe的native方法，实现高效的硬件级别CAS
            returnunsafe.compareAndSwapInt(this,valueOffset,expect,update);
    }
优点总结:
最大的好处就是可以避免多线程的优先级倒置和死锁情况的发生，提升在高并发处理下的性能。

# 信号量Semaphore
Semaphore也叫信号量，在JDK1.5被引入，可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。

Semaphore内部维护了一组虚拟的许可，许可的数量可以通过构造函数的参数指定。

访问特定资源前，必须使用acquire方法获得许可，如果许可数量为0，该线程则一直阻塞，直到有可用许可。

访问资源后，使用release释放许可。

Semaphore和ReentrantLock类似，获取许可有公平策略和非公平许可策略，默认情况下使用非公平策略。

## 应用场景
Semaphore可以用来做流量分流，特别是对公共资源有限的场景，比如数据库连接。

假设有这个的需求，读取几万个文件的数据到数据库中，由于文件读取是IO密集型任务，可以启动几十个线程并发读取，但是数据库连接数只有10个，这时就必须控制最多只有10个线程能够拿到数据库连接进行操作。这个时候，就可以使用Semaphore做流量控制。
    
    public class SemaphoreTest {
        private static final int COUNT = 40;
        private static Executor executor = Executors.newFixedThreadPool(COUNT);
        private static Semaphore semaphore = new Semaphore(10);
        public static void main(String[] args) {
            for (int i=0; i< COUNT; i++) {
                executor.execute(new ThreadTest.Task());
            }
        }
    
        static class Task implements Runnable {
            @Override
            public void run() {
                try {
                    //读取文件操作
                    semaphore.acquire();
                    // 存数据过程
                    semaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                }
            }
        }
    }

## 实现原理
代码源于JDK1.8
Semaphore实现主要基于java同步器AQS，不熟悉的可以移步 [深入浅出java同步器](https://www.jianshu.com/p/d8eeb31bee5c)。

内部使用state表示许可数量。

### 非公平策略

acquire实现，核心代码如下：
    
    final int nonfairTryAcquireShared(int acquires) {
        for (;;) {
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
acquires值默认为1，表示尝试获取1个许可，remaining代表剩余的许可数。

如果remaining < 0，表示目前没有剩余的许可。

当前线程进入AQS中的doAcquireSharedInterruptibly方法等待可用许可并挂起，直到被唤醒。

release实现，核心代码如下：
    
    protected final boolean tryReleaseShared(int releases) {
        for (;;) {
            int current = getState();
            int next = current + releases;
            if (next < current) // overflow
                throw new Error("Maximum permit count exceeded");
            if (compareAndSetState(current, next))
                return true;
        }
    }
releases值默认为1，表示尝试释放1个许可，next代表如果许可释放成功，可用许可的数量。

通过unsafe.compareAndSwapInt修改state的值，确保同一时刻只有一个线程可以释放成功。

许可释放成功，当前线程进入到AQS的doReleaseShared方法，唤醒队列中等待许可的线程。

也许有人会有疑问，非公平性体现在哪里？

当一个线程A执行acquire方法时，会直接尝试获取许可，而不管同一时刻阻塞队列中是否有线程也在等待许可，如果恰好有线程C执行release释放许可，并唤醒阻塞队列中第一个等待的线程B，这个时候，线程A和线程B是共同竞争可用许可，不公平性就是这么体现出来的，线程A一点时间都没等待就和线程B同等对待。

### 公平策略

acquire实现，核心代码如下：
    
    protected int tryAcquireShared(int acquires) {
        for (;;) {
            if (hasQueuedPredecessors())
                return -1;
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
acquires值默认为1，表示尝试获取1个许可，remaining代表剩余的许可数。
可以看到和非公平策略相比，就多了一个对阻塞队列的检查。

如果阻塞队列没有等待的线程，则参与许可的竞争。
否则直接插入到阻塞队列尾节点并挂起，等待被唤醒。

release实现，和非公平策略一样。

# 参考资料
【1】https://www.jianshu.com/p/509aca840f6d

【2】https://www.ibm.com/developerworks/cn/java/j-jtp11234/

【3】https://www.jianshu.com/p/0090341c6b80

【4】https://www.jianshu.com/p/d8eeb31bee5c