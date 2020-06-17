---
title: Java之Queue、Deque、ArrayDeque

date: 2020-6-15 20:16:12

categories: 2020年6月

tags: [Java]

---

对Java集合中的Queue、Deque、ArrayDeque源码进行分析。


<!-- more -->

Java里有一个叫做Stack的类，却没有叫做Queue的类（它是个接口名字）。

当需要使用栈时，Java已不推荐使用Stack，而是推荐使用更高效的ArrayDeque；既然Queue只是一个接口，当需要使用队列时也就首选ArrayDeque了（次选是LinkedList）[^1]。

# 队列Queue

Queue是具有队列特性的接口[^2]

Queue具有先进先出的特点

Queue所有新元素都插入队列的末尾，移除元素都移除队列的头部

```
public interface Queue<E> extends Collection<E> {
    //往队列插入元素，如果出现异常会抛出异常
    boolean add(E e);
    //往队列插入元素，如果出现异常则返回false
    boolean offer(E e);
    //移除队列元素，如果出现异常会抛出异常
    E remove();
    //移除队列元素，如果出现异常则返回null
    E poll();
    //获取队列头部元素，如果出现异常会抛出异常
    E element();
    //获取队列头部元素，如果出现异常则返回null
    E peek();
}
```
用表格形式来表现：
    
    操作	    抛出异常	    返回特殊值
    插入	    add()	        offer()
    删除	    remove()	    poll()
    查询	    element()	    peek()

# 双端队列Deque(Double Ended Queue)

Deque是一个双端队列

Deque继承自Queue,Deque也是Queue，Deque也能当Queue用，没有太多额外开销。所以jdk没有单独实现Queue。

Deque具有先进先出或后进先出的特点

Deque支持所有元素在头部和尾部进行插入、删除、获取


```
public interface Deque<E> extends Queue<E> {
    void addFirst(E e);//插入头部，异常会报错
    boolean offerFirst(E e);//插入头部，异常返回false
    E getFirst();//获取头部，异常会报错
    E peekFirst();//获取头部，异常不报错
    E removeFirst();//移除头部，异常会报错
    E pollFirst();//移除头部，异常不报错
    
    void addLast(E e);//插入尾部，异常会报错
    boolean offerLast(E e);//插入尾部，异常返回false
    E getLast();//获取尾部，异常会报错
    E peekLast();//获取尾部，异常不报错
    E removeLast();//移除尾部，异常会报错
    E pollLast();//移除尾部，异常不报错
}
```

# ArrayDeque

实现于Deque，拥有队列或者栈特性的接口

实现于Cloneable，拥有克隆对象的特性

实现于Serializable，拥有序列化的能力

Deque有两种实现类：

1. LinkedList。也就是链表，java的链表同时实现了Deque。

2. ArrayDeque。Deque的数组实现,官方更推荐使用AarryDeque用作栈和队列。


```
public class ArrayDeque<E> extends AbstractCollection<E>
                       implements Deque<E>, Cloneable, Serializable{}
```

从名字可以看出ArrayDeque底层通过数组实现，为了满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，即循环数组（circular array），也就是说数组的任何一点都可能被看作起点或者终点。ArrayDeque是非线程安全的（not thread-safe），当多个线程同时使用的时候，需要程序员手动同步；另外，该容器不允许放入null元素。


先来总结下ArrayDeque的实现思路。[^3]

首先，ArrayDeque内部是拥有一个内部数组用于存储数据。

其次，假设采用简单的方案，即队列数组按顺序在数组里排开，那么：

由于ArrayDeque的两端都能增删数据，那么把数据插入到队列头部也就是数组头部，会造成O(N)的时间复杂度。

假设只在队尾加入而只从队头删除，队头就会空出越来越多的空间。

那么该怎么实现？也很简单。将物理上的连续数组回绕，形成逻辑上的一个 环形结构。即a[size - 1]的下一个位置是a[0].

之后，使用头尾指针标识队列头尾，在队列头尾增删元素，反映在头尾指针上就是这两个指针绕着环赛跑。

这个是大体思路，具体的还有一些细节，后面代码里分析：

- head和tail的具体概念是如何界定？
- 如果判断队满和队空？
- 数组满了怎么办？

## 属性

先来看内部属性。elements域就是存储数据的原生数组。

head和tail分别分别为头尾指针。

```
    transient Object[] elements; // non-private to simplify nested class access

    transient int head;

    transient int tail;
```
## 构造函数

```
    public ArrayDeque() {
        elements = new Object[16];
    }

    public ArrayDeque(int numElements) {
        allocateElements(numElements);
    }

    private void allocateElements(int numElements) {
        elements = new Object[calculateSize(numElements)];
    }
```

1. 如果没有指定内部数组的初始大小，默认为16.

2. 如果指定了内部数组的初始大小，则通过calculateSize函数二次计算出大小。

来看calculateSize函数：

```
 private static final int MIN_INITIAL_CAPACITY = 8;

    private static int calculateSize(int numElements) {
        int initialCapacity = MIN_INITIAL_CAPACITY;
        // Find the best power of two to hold elements.
        // Tests "<=" because arrays aren't kept full.
        if (numElements >= initialCapacity) {
            initialCapacity = numElements;
            initialCapacity |= (initialCapacity >>>  1);
            initialCapacity |= (initialCapacity >>>  2);
            initialCapacity |= (initialCapacity >>>  4);
            initialCapacity |= (initialCapacity >>>  8);
            initialCapacity |= (initialCapacity >>> 16);
            initialCapacity++;

            if (initialCapacity < 0)   // Too many elements, must back off
                initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
        }
        return initialCapacity;
    }
```

- 如果小于8，那么大小就为8.
- 如果大于等于8，则按照2的幂对齐。

在初始化中，数组要求的大小必须为2^n，所以有这么一个算法，如果当前的大小大于默认规定的大小时，就会去计算出新的大小，那么这个计算过程是怎么样的呢？我们举一个例子进行分析:

如果initialCapacity为10的时候，那么二进制为 1010

经过initialCapacity |= (initialCapacity >>>  1)时，那么二进制为 1010 | 0101 = 1111

经过initialCapacity |= (initialCapacity >>>  2)时，那么二进制为 1111 | 0011 = 1111

后面计算的结果都是1111，可以理解为将二进制的低位数都补上1，这样出来的结果都是2^n-1

最后initialCapacity++，2^n-1+1出来的结果就是2^n

## 入队
看两个入队方法：
```
    public void addFirst(E e) {
        if (e == null)
            throw new NullPointerException();
        elements[head = (head - 1) & (elements.length - 1)] = e;
        if (head == tail)
            doubleCapacity();
    }

    public void addLast(E e) {
        if (e == null)
            throw new NullPointerException();
        elements[tail] = e;
        if ( (tail = (tail + 1) & (elements.length - 1)) == head)
            doubleCapacity();
    }
```
addFirst是从队头插入，addLast是从队尾插入。

从该代码能够分析出head和tail指针的含义：

head指针指向的是队头元素的位置，除非队列为空。
tail指针指向的是队尾元素后一格的位置，即尾后指针。
因此：

如果队列没有满，tail指向的是空位置，head指向的是队头元素，永远不可能一样。

但是当队列满时，tail回绕会追上head，当tail等于head时，表示队列满了。

理清楚了这一点，上面的代码也就十分容易理解了：

对应位置插入位置，移动指针。

当tail和head相等时，扩容。

最后，这句：

```
(head - 1) & (elements.length - 1)
```

假如被余数是2的幂次方，那么模运算就能够优化成按位与运算。

也即相当于：
```
(head - 1) % elements.length
```

## 出队
```
    public E pollFirst() {
        int h = head;
        @SuppressWarnings("unchecked")
        E result = (E) elements[h];
        // Element is null if deque empty
        if (result == null)
            return null;
        elements[h] = null;     // Must null out slot
        head = (h + 1) & (elements.length - 1);
        return result;
    }

    public E pollLast() {
        int t = (tail - 1) & (elements.length - 1);
        @SuppressWarnings("unchecked")
        E result = (E) elements[t];
        if (result == null)
            return null;
        elements[t] = null;
        tail = t;
        return result;
    }
```
出队的代码很显然，不多解释。

## 扩容
```
    private void doubleCapacity() {
        assert head == tail;
        int p = head;
        int n = elements.length;
        int r = n - p; // number of elements to the right of p
        int newCapacity = n << 1;
        // 扩容后的大小小于0（溢出），也即队列最大应该是2的30次方
        if (newCapacity < 0)
            throw new IllegalStateException("Sorry, deque too big");
        Object[] a = new Object[newCapacity];
        System.arraycopy(elements, p, a, 0, r);
        System.arraycopy(elements, 0, a, r, p);
        elements = a;
        head = 0;
        tail = n;
    }
```
扩容的实现为按 两倍 扩容原数组，将原数倍拷贝过去。

其中值得注意的是对数组大小溢出的处理。

## 迭代器
容器的实现中，所有修改过容器结构的操作都需要修改modCount字段。

这样迭代器迭代过程中，通过前后比对该字段来判断容器是否被动过，及时抛出异常终止迭代以免造成不可预测的问题。

不过，在ArrayDeque的插入方法中并没有修改modeCount字段。从ArrayDeque的迭代器的实现中可以看出来：
```
    private class DeqIterator implements Iterator<E> {
        /**
         * Index of element to be returned by subsequent call to next.
         */
        private int cursor = head;

        /**
         * Tail recorded at construction (also in remove), to stop
         * iterator and also to check for comodification.
         */
        private int fence = tail;
    }
```
原来，ArrayDeque直接使用了head和tail头尾指针，就能判断出迭代过程中是否发生了变化。



[^1]:https://www.cnblogs.com/CarpenterLee/p/5468803.html

[^2]:https://blog.csdn.net/qq_30379689/article/details/80558771

[^3]:https://segmentfault.com/a/1190000016330001