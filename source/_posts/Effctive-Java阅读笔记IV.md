title: Effctive-Java阅读笔记IV
date: 2019-12-31 15:06:12
categories: 2019年12月
tags: [Java]

---

第二章阅读：创建和销毁对象
07.消除过期的对象引用
08.避免使用 Finalizer 和 Cleaner 机制
除了作为一个安全网或者终止非关键的本地资源，不要使用 Cleaner 机制，或者是在 Java 9 发布之前的 finalizers 机制。
<!-- more -->

# 消除过期的对象引用
如果你从使用手动内存管理的语言（如 C 或 C++）切换到像 Java 这样的带有垃圾收集机制的语言，那么作为程序员的工作就会变得容易多了，因为你的对象在使用完毕以后就自动回收了。当你第一次体验它的时候，它就像魔法一样。这很容易让人觉得你不需要考虑内存管理，但这并不完全正确。

    // Can you spot the "memory leak"?
    public class Stack {
        private Object[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;

        public Stack() {
            elements = new Object[DEFAULT_INITIAL_CAPACITY];
        }

        public void push(Object e) {
            ensureCapacity();
            elements[size++] = e;
        }

        public Object pop() {
            if (size == 0)
                throw new EmptyStackException();
            return elements[--size];
        }

        /**
         * Ensure space for at least one more element, roughly
         * doubling the capacity each time the array needs to grow.
         ** /
        private void ensureCapacity() {
            if (elements.length == size)
                elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

这个程序没有什么明显的错误（但是对于泛型版本，请参阅条目 29）。 你可以对它进行详尽的测试，它都会成功地通过每一项测试，但有一个潜在的问题。 笼统地说，程序有一个「内存泄漏」，由于垃圾回收器的活动的增加，或内存占用的增加，静默地表现为性能下降。 在极端的情况下，这样的内存泄漏可能会导致磁盘分页（disk paging），甚至导致内存溢出（OutOfMemoryError）的失败，但是这样的故障相对较少。

　　那么哪里发生了内存泄漏？ 如果一个栈增长后收缩，那么从栈弹出的对象不会被垃圾收集，即使使用栈的程序不再引用这些对象。 这是因为栈维护对这些对象的过期引用（obsolete references）。 过期引用简单来说就是永远不会解除的引用。 在这种情况下，元素数组「活动部分（active portion）」之外的任何引用都是过期的。 活动部分是由索引下标小于 size 的元素组成。

　　垃圾收集语言中的内存泄漏（更适当地称为无意的对象保留 unintentional object retentions）是隐蔽的。 如果无意中保留了对象引用，那么不仅这个对象排除在垃圾回收之外，而且该对象引用的任何对象也是如此。 即使只有少数对象引用被无意地保留下来，也可以阻止垃圾回收机制对许多对象的回收，这对性能产生很大的影响。

　　这类问题的解决方法很简单：一旦对象引用过期，将它们设置为 null。 在我们的 Stack 类的情景下，只要从栈中弹出，元素的引用就设置为过期。 pop 方法的修正版本如下所示：

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

取消过期引用的另一个好处是，如果它们随后被错误地引用，程序立即抛出 NullPointerException 异常，而不是悄悄地做继续做错误的事情。尽可能快地发现程序中的错误是有好处的。

　　当程序员第一次被这个问题困扰时，他们可能会在程序结束后立即清空所有对象引用。这既不是必要的，也不是可取的；它不必要地搞乱了程序。清空对象引用应该是例外而不是规范。消除过期引用的最好方法是让包含引用的变量超出范围。如果在最近的作用域范围内定义每个变量 （详见第 57 条），这种自然就会出现这种情况。

　　那么什么时候应该清空一个引用呢？Stack 类的哪个方面使它容易受到内存泄漏的影响？简单地说，它管理自己的内存。存储池（storage pool）由 elements 数组的元素组成（对象引用单元，而不是对象本身）。数组中活动部分的元素 (如前面定义的) 被分配，其余的元素都是空闲的。垃圾收集器没有办法知道这些；对于垃圾收集器来说，elements 数组中的所有对象引用都同样有效。只有程序员知道数组的非活动部分不重要。程序员可以向垃圾收集器传达这样一个事实，一旦数组中的元素变成非活动的一部分，就可以手动清空这些元素的引用。

　　一般来说，当一个类自己管理内存时，程序员应该警惕内存泄漏问题。 每当一个元素被释放时，元素中包含的任何对象引用都应该被清除。

　　另一个常见的内存泄漏来源是缓存。 一旦将对象引用放入缓存中，很容易忘记它的存在，并且在它变得无关紧要之后，仍然保留在缓存中。对于这个问题有几种解决方案。如果你正好想实现了一个缓存：只要在缓存之外存在对某个项（entry）的键（key）引用，那么这项就是明确有关联的，就可以用 WeakHashMap 来表示缓存；这些项在过期之后自动删除。记住，只有当缓存中某个项的生命周期是由外部引用到键（key）而不是值（value）决定时，WeakHashMap 才有用。

　　更常见的情况是，缓存项有用的生命周期不太明确，随着时间的推移一些项变得越来越没有价值。在这种情况下，缓存应该偶尔清理掉已经废弃的项。这可以通过一个后台线程 (也许是 ScheduledThreadPoolExecutor) 或将新的项添加到缓存时顺便清理。LinkedHashMap 类使用它的 removeEldestEntry 方法实现了后一种方案。对于更复杂的缓存，可能直接需要使用 java.lang.ref。

　　第三个常见的内存泄漏来源是监听器和其他回调。如果你实现了一个 API，其客户端注册回调，但是没有显式地撤销注册回调，除非采取一些操作，否则它们将会累积。确保回调是垃圾收集的一种方法是只存储弱引用（weak references），例如，仅将它们保存在 WeakHashMap 的键（key）中。

　　因为内存泄漏通常不会表现为明显的故障，所以它们可能会在系统中保持多年。 通常仅在仔细的代码检查或借助堆分析器（heap profiler）的调试工具才会被发现。 因此，学习如何预见这些问题，并防止这些问题发生，是非常值得的。

# 避免使用 Finalizer 和 Cleaner 机制

Finalizer 机制是不可预知的，往往是危险的，而且通常是不必要的。 它们的使用会导致不稳定的行为，糟糕的性能和移植性问题。 Finalizer 机制有一些特殊的用途，我们稍后会在这个条目中介绍，但是通常应该避免它们。 从 Java 9 开始，Finalizer 机制已被弃用，但仍被 Java 类库所使用。 Java 9 中 Cleaner 机制代替了 Finalizer 机制。 Cleaner 机制不如 Finalizer 机制那样危险，但仍然是不可预测，运行缓慢并且通常是不必要的。

　　提醒 C++程序员不要把 Java 中的 Finalizer 或 Cleaner 机制当成的 C++ 析构函数的等价物。 在 C++ 中，析构函数是回收对象相关资源的正常方式，是与构造方法相对应的。 在 Java 中，当一个对象变得不可达时，垃圾收集器回收与对象相关联的存储空间，不需要开发人员做额外的工作。 C++ 析构函数也被用来回收其他非内存资源。 在 Java 中，try-with-resources 或 try-finally 块用于此目的（详见第 9 条）。

　　Finalizer 和 Cleaner 机制的一个缺点是不能保证他们能够及时执行[JLS，12.6]。 在一个对象变得无法访问时，到 Finalizer 和 Cleaner 机制开始运行时，这期间的时间是任意长的。 这意味着你永远不应该 Finalizer 和 Cleaner 机制做任何时间敏感（time-critical）的事情。 例如，依赖于 Finalizer 和 Cleaner 机制来关闭文件是严重的错误，因为打开的文件描述符是有限的资源。 如果由于系统迟迟没有运行 Finalizer 和 Cleaner 机制而导致许多文件被打开，程序可能会失败，因为它不能再打开文件了。

　　及时执行 Finalizer 和 Cleaner 机制是垃圾收集算法的一个功能，这种算法在不同的实现中有很大的不同。程序的行为依赖于 Finalizer 和 Cleaner 机制的及时执行，其行为也可能大不不同。 这样的程序完全可以在你测试的 JVM 上完美运行，然而在你最重要的客户的机器上可能运行就会失败。

　　延迟终结（finalization）不只是一个理论问题。为一个类提供一个 Finalizer 机制可以任意拖延它的实例的回收。一位同事调试了一个长时间运行的 GUI 应用程序，这个应用程序正在被一个 OutOfMemoryError 错误神秘地死掉。分析显示，在它死亡的时候，应用程序的 Finalizer 机制队列上有成千上万的图形对象正在等待被终结和回收。不幸的是，Finalizer 机制线程的运行优先级低于其他应用程序线程，所以对象被回收的速度低于进入队列的速度。语言规范并不保证哪个线程执行 Finalizer 机制，因此除了避免使用 Finalizer 机制之外，没有轻便的方法来防止这类问题。在这方面， Cleaner 机制比 Finalizer 机制要好一些，因为 Java 类的创建者可以控制自己 cleaner 机制的线程，但 cleaner 机制仍然在后台运行，在垃圾回收器的控制下运行，但不能保证及时清理。

　　Java 规范不能保证 Finalizer 和 Cleaner 机制能及时运行；它甚至不能能保证它们是否会运行。当一个程序结束后，一些不可达对象上的 Finalizer 和 Cleaner 机制仍然没有运行。因此，不应该依赖于 Finalizer 和 Cleaner 机制来更新持久化状态。例如，依赖于 Finalizer 和 Cleaner 机制来释放对共享资源（如数据库）的持久锁，这是一个使整个分布式系统陷入停滞的好方法。

　　不要相信 System.gc 和 System.runFinalization 方法。 他们可能会增加 Finalizer 和 Cleaner 机制被执行的几率，但不能保证一定会执行。 曾经声称做出这种保证的两个方法：System.runFinalizersOnExit 和它的孪生兄弟 Runtime.runFinalizersOnExit，包含致命的缺陷，并已被弃用了几十年[ThreadStop]。

　　Finalizer 机制的另一个问题是在执行 Finalizer 机制过程中，未捕获的异常会被忽略，并且该对象的 Finalizer 机制也会终止 [JLS, 12.6]。未捕获的异常会使其他对象陷入一种损坏的状态（corrupt state）。如果另一个线程试图使用这样一个损坏的对象，可能会导致任意不确定的行为。通常情况下，未捕获的异常将终止线程并打印堆栈跟踪（ stacktrace），但如果发生在 Finalizer 机制中，则不会发出警告。Cleaner 机制没有这个问题，因为使用 Cleaner 机制的类库可以控制其线程。

　　使用 finalizer 和 cleaner 机制会导致严重的性能损失。 在我的机器上，创建一个简单的 AutoCloseable 对象，使用 try-with-resources 关闭它，并让垃圾回收器回收它的时间大约是 12 纳秒。 使用 finalizer 机制，而时间增加到 550 纳秒。 换句话说，使用 finalizer 机制创建和销毁对象的速度要慢 50 倍。 这主要是因为 finalizer 机制会阻碍有效的垃圾收集。 如果使用它们来清理类的所有实例（在我的机器上的每个实例大约是 500 纳秒），那么 cleaner 机制的速度与 finalizer 机制的速度相当，但是如果仅将它们用作安全网（safety net），则 cleaner 机制要快得多，如下所述。 在这种环境下，创建，清理和销毁一个对象在我的机器上需要大约 66 纳秒，这意味着如果你不使用安全网的话，需要支付 5 倍（而不是 50 倍）的保险。

　　finalizer 机制有一个严重的安全问题：它们会打开你的类来进行 finalizer 机制攻击。finalizer 机制攻击的想法很简单：如果一个异常是从构造方法或它的序列化中抛出的——readObject 和 readResolve 方法 （第 12 章）——恶意子类的 finalizer 机制可以运行在本应该「中途夭折（died on the vine）」的部分构造对象上。finalizer 机制可以在静态字属性记录对对象的引用，防止其被垃圾收集。一旦记录了有缺陷的对象，就可以简单地调用该对象上的任意方法，而这些方法本来就不应该允许存在。从构造方法中抛出异常应该足以防止对象出现；而在 finalizer 机制存在下，则不是。这样的攻击会带来可怕的后果。Final 类不受 finalizer 机制攻击的影响，因为没有人可以编写一个 final 类的恶意子类。为了保护非 final 类不受 finalizer 机制攻击，编写一个 final 的 finalize 方法，它什么都不做。

　　那么，你应该怎样做呢？为对象封装需要结束的资源（如文件或线程），而不是为该类编写 Finalizer 和 Cleaner 机制？让你的类实现 AutoCloseable 接口即可，并要求客户在在不再需要时调用每个实例 close 方法，通常使用 try-with-resources 确保终止，即使面对有异常抛出情况（详见第 9 条）。一个值得一提的细节是实例必须跟踪是否已经关闭：close 方法必须记录在对象里不再有效的属性，其他方法必须检查该属性，如果在对象关闭后调用它们，则抛出 IllegalStateException 异常。

　　那么，Finalizer 和 Cleaner 机制有什么好处呢？它们可能有两个合法用途。一个是作为一个安全网（safety net），以防资源的拥有者忽略了它的 close 方法。虽然不能保证 Finalizer 和 Cleaner 机制会迅速运行 (或者根本就没有运行)，最好是把资源释放晚点出来，也要好过客户端没有这样做。如果你正在考虑编写这样的安全网 Finalizer 机制，请仔细考虑一下这样保护是否值得付出对应的代价。一些 Java 库类，如 FileInputStream、FileOutputStream、ThreadPoolExecutor 和 java.sql.Connection，都有作为安全网的 Finalizer 机制。

　　第二种合理使用 Cleaner 机制的方法与本地对等类（native peers）有关。本地对等类是一个由普通对象委托的本地 (非 Java) 对象。由于本地对等类不是普通的 Java 对象，所以垃圾收集器并不知道它，当它的 Java 对等对象被回收时，本地对等类也不会回收。假设性能是可以接受的，并且本地对等类没有关键的资源，那么 Finalizer 和 Cleaner 机制可能是这项任务的合适的工具。但如果性能是不可接受的，或者本地对等类持有必须迅速回收的资源，那么类应该有一个 close 方法，正如前面所述。

　　Cleaner 机制使用起来有点棘手。下面是演示该功能的一个简单的 Room 类。假设 Room 对象必须在被回收前清理干净。Room 类实现 AutoCloseable 接口；它的自动清理安全网使用的是一个 Cleaner 机制，这仅仅是一个实现细节。与 Finalizer 机制不同，Cleaner 机制不污染一个类的公共 API：

    // An autocloseable class using a cleaner as a safety net
    public class Room implements AutoCloseable {
        private static final Cleaner cleaner = Cleaner.create();

        // Resource that requires cleaning. Must not refer to Room!
        private static class State implements Runnable {
            int numJunkPiles; // Number of junk piles in this room

            State(int numJunkPiles) {
                this.numJunkPiles = numJunkPiles;
            }

            // Invoked by close method or cleaner
            @Override
            public void run() {
                System.out.println("Cleaning room");
                numJunkPiles = 0;
            }
        }

        // The state of this room, shared with our cleanable
        private final State state;

        // Our cleanable. Cleans the room when it’s eligible for gc
        private final Cleaner.Cleanable cleanable;

        public Room(int numJunkPiles) {
            state = new State(numJunkPiles);
            cleanable = cleaner.register(this, state);
        }

        @Override
        public void close() {
            cleanable.clean();
        }
    }
　　静态内部 State 类拥有 Cleaner 机制清理房间所需的资源。 在这里，它仅仅包含 numJunkPiles 属性，它代表混乱房间的数量。 更实际地说，它可能是一个 final 修饰的 long 类型的指向本地对等类的指针。 State 类实现了 Runnable 接口，其 run 方法最多只能调用一次，只能被我们在 Room 构造方法中用 Cleaner 机制注册 State 实例时得到的 Cleanable 调用。 对 run 方法的调用通过以下两种方法触发：通常，通过调用 Room 的 close 方法内调用 Cleanable 的 clean 方法来触发。 如果在 Room 实例有资格进行垃圾回收的时候客户端没有调用 close 方法，那么 Cleaner 机制将（希望）调用 State 的 run 方法。

　　一个 State 实例不引用它的 Room 实例是非常重要的。如果它引用了，则创建了一个循环，阻止了 Room 实例成为垃圾收集的资格（以及自动清除）。因此，State 必须是静态的嵌内部类，因为非静态内部类包含对其宿主类的实例的引用（详见第 24 条）。同样，使用 lambda 表达式也是不明智的，因为它们很容易获取对宿主类对象的引用。

　　就像我们之前说的，Room 的 Cleaner 机制仅仅被用作一个安全网。如果客户将所有 Room 的实例放在 try-with-resource 块中，则永远不需要自动清理。行为良好的客户端如下所示：

    public class Adult {
        public static void main(String[] args) {
            try (Room myRoom = new Room(7)) {
                System.out.println("Goodbye");
            }
        }
    }

　　正如你所预料的，运行 Adult 程序会打印 Goodbye 字符串，随后打印 Cleaning room 字符串。但是如果时不合规矩的程序，它从来不清理它的房间会是什么样的?

    public class Teenager {
        public static void main(String[] args) {
            new Room(99);
            System.out.println("Peace out");
        }
    }

　　你可能期望它打印出 Peace out，然后打印 Cleaning room 字符串，但在我的机器上，它从不打印 Cleaning room 字符串；仅仅是程序退出了。 这是我们之前谈到的不可预见性。 Cleaner 机制的规范说：“System.exit 方法期间的清理行为是特定于实现的。 不保证清理行为是否被调用。”虽然规范没有说明，但对于正常的程序退出也是如此。 在我的机器上，将 System.gc() 方法添加到 Teenager 类的 main 方法足以让程序退出之前打印 Cleaning room，但不能保证在你的机器上会看到相同的行为。

　　总之，除了作为一个安全网或者终止非关键的本地资源，不要使用 Cleaner 机制，或者是在 Java 9 发布之前的 finalizers 机制。即使是这样，也要当心不确定性和性能影响。
# 参考资料：
【1】https://sjsdfg.github.io/effective-java-3rd-chinese/#/notes/08.%20%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8Finalizer%E5%92%8CCleaner%E6%9C%BA%E5%88%B6
