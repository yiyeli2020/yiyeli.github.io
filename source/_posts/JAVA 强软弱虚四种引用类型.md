---

title: JAVA 强软弱虚四种引用类型

date: 2021-02-20 15:12:12

categories: 2021年2月

tags: [Java, Note]


---

来自JAVA面试必问的102个知识点-马士兵视频

<!-- more -->

# 重写Java中的finalize方法造成OOM内存溢出

<details>
    <summary>重写Java中的finalize方法造成OOM内存溢出</summary>
    
```
    @Override
    protected void finalize() throws Throwable {
        System.out.println("finalize");
    }
```
</details>

小米C++开发者转写Java时重写Java中的finalize方法造成OOM内存溢出，因为重写的finalize方法耗时多回收老对象变慢，新的对象不断涌入内存，使得内存出现OOM溢出


# 四种引用类型：强软弱虚引用[^2]

从Java SE2开始，就提供了四种类型的引用：强引用、软引用、弱引用和虚引用。Java中提供这四种引用类型主要有两个目的：第一是可以让程序员通过代码的方式决定某些对象的生命周期；第二是有利于JVM进行垃圾回收。

## 强引用
之前我们使用的大部分引用实际上都是强引用，这是使用最普遍的引用。

<details>
    <summary>强引用</summary>
    
```
    Max m = new Max();
    m = null;
    System.gc();//显式调用垃圾回收
    System.out.println(m);
    //阻塞main线程，给垃圾回收线程执行
    System.in.read();
```
</details>
如果一个对象具有强引用，那就类似于必不可少的物品，不会被垃圾回收器回收。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不回收这种对象。

如果想中断强引用和某个对象之间的关联，可以显示地将引用赋值为null，这样一来的话，JVM在合适的时间就会回收该对象。


## 软引用
软引用是用来描述一些有用但并不是必需的对象，在Java中用java.lang.ref.SoftReference类来表示。对于软引用关联着的对象，只有在内存不足的时候JVM才会回收该对象。因此，这一点可以很好地用来解决OOM的问题，并且这个特性很适合用来实现缓存：比如网页缓存、图片缓存等。


<details>
    <summary>软引用</summary>
    
```
       SoftReference<byte[]> m = new SoftReference<>(new byte[1024 * 1024 * 10]);

        System.out.println(m.get());
        System.gc();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(m.get());
        //在Run->Edit Configuration->VM Options中设置为-Xmx20M
        //下面再分配一个数组，堆20M已经装不下了，系统就会垃圾回收，先回收一次，会把软引用回收
        byte[] b = new byte[1024 * 1024 * 15];
        System.out.println(m.get());
```
</details>


这里为了能出现以下效果，可以再把堆的设置Run->Edit Configuration->VM Options中设置为-Xmx20M调的大一些例如30M
    
    [B@63947c6b
    [B@63947c6b
    null


软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被JVM回收，这个软引用就会被加入到与之关联的引用队列中。

当内存足够大时可以把数组存入软引用，取数据时就可从内存里取数据，提高运行效率

软引用在实际中有重要的应用，例如浏览器的后退按钮，这个后退时显示的网页内容可以重新进行请求或者从缓存中取出：

（1）如果一个网页在浏览结束时就进行内容的回收，则按后退查看前面浏览过的页面时，需要重新构建

（2）如果将浏览过的网页存储到内存中会造成内存的大量浪费，甚至会造成内存溢出这时候就可以使用软引用




## 弱引用

弱引用也是用来描述非必需对象的，当JVM进行垃圾回收时，无论内存是否充足，都会回收被弱引用关联的对象。在java中，用java.lang.ref.WeakReference类来表示。

弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。所以被软引用关联的对象只有在内存不足时才会被回收，而被弱引用关联的对象在JVM进行垃圾回收时总会被回收。


<details>
    <summary>弱引用</summary>
    
```
        WeakReference<Max> m = new WeakReference<>(new Max());

        System.out.println(m.get());
        System.gc();
        System.out.println(m.get());
```
</details>

在使用软引用和弱引用的时候，我们可以显示地通过System.gc()来通知JVM进行垃圾回收，但是要注意的是，虽然发出了通知，JVM不一定会立刻执行，也就是说这句是无法确保此时JVM一定会进行垃圾回收的。

 

弱引用还可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

当要获得weak reference引用的object时, 首先需要判断它是否已经被回收，如果wr.get()方法为空, 那么说明weakCar指向的对象已经被回收了。

应用场景：如果一个对象是偶尔的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么应该用 Weak Reference 来记住此对象。或者想引用一个对象，但是这个对象有自己的生命周期，你不想介入这个对象的生命周期，这时候就应该用弱引用，这个引用不会在对象的垃圾回收判断中产生任何附加的影响。


## 虚引用

虚引用，又称作幻象引用，如果一个对象具有虚引用，那么它和没有任何引用一样，被虚引用关联的对象引用通过get方法获取到的永远为null，也就是说这种对象在任何时候都有可能被垃圾回收器回收，通过这种方式关联的对象也无法调用对象中的方法。虚引用主要是用来管理堆外内存的，通过ReferenceQueue这个类实现，当一个对象被回收的时候，会向这个引用队列里面添加相关数据，给一个通知。

虚引用和前面的软引用、弱引用不同，它并不影响对象的生命周期。在java中用java.lang.ref.PhantomReference类表示。如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。虚引用主要用来跟踪对象被垃圾回收的活动。

虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。


<details>
    <summary>虚引用</summary>
    
```

    private static final List<Object> LIST=new LinkedList<>();
    //引用队列
    private static final ReferenceQueue<Max> QUEUE = new ReferenceQueue<>();

    public static void main(String[] args) throws IOException {

        PhantomReference<Max> phantomReference = new PhantomReference<>(new Max(), QUEUE);
        System.out.println(phantomReference.get());
        //在Run->Edit Configuration->VM Options中设置为-Xmx20M
        new Thread(() -> {
            while (true) {
                //该线程向队列中不断添加对象直至内存塞满，需要开始被回收了
                LIST.add(new byte[1024 * 1024]);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    Thread.currentThread().interrupt();
                }
                System.out.println(phantomReference.get());
            }
        }).start();

        //垃圾回收线程
        new Thread(() -> {
            while (true) {
                //在队列里寻找，直到找到虚引用对象才开始回收
                Reference<? extends Max> poll = QUEUE.poll();
                if (poll != null) {
                    System.out.println("虚引用对象被JVM回收了    "+poll);
                }
            }
        }).start();
    }
```
</details>


## 虚引用的用途-管理堆外/直接内存


Java NIO（同步非阻塞的I/O模型）中关键点是零拷贝，在操作系统中接收到网卡的数据要拷贝到buffer中，Java如果想用就要拷贝到堆内存中。

为了实现零拷贝，在JVM中，使用虚引用来管理堆外内存，发现要回收时虚引用时将其放入队列中，然后在队列处理过程中在HotSpot级别释放堆外内存。

<details>
    <summary>虚引用的用途-管理堆外内存/直接内存</summary>
    
```
    
    //分配在JVM外部的内存
    ByteBuffer byteBuffer = ByteBuffer.allocateDirect(1024);   
       
```
</details>

## 弱引用重点讲解 ThreadLocal在spring事务中的应用


<details>
    <summary>弱引用重点讲解 ThreadLocal在spring事务中的应用</summary>
    
```
import java.util.concurrent.TimeUnit;

public class TestThreadLocal {
    //ThreadLocal是个容器，里面可以装Person对象
    static ThreadLocal<Person> t1 = new ThreadLocal<>();

    public static void main(String[] args) {

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(t1.get());
        }).start();


        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            t1.set(new Person());
        }).start();

    }
}
class Person{
    String name = "ZhangSan";
}
```
</details>



一个线程中放入Person对象，另一个线程中却取不到该对象，可以看ThreadLocal源码





[^1]:https://www.bilibili.com/video/BV1bf4y1Q7Gn?p=3

[^2]:https://blog.csdn.net/Light_makeup/article/details/107301647