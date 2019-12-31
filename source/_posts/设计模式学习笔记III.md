---
title: 设计模式学习笔记III
date: 2019-12-30 16:55:12
categories: 2019年12月
tags: [Design Pattern，Java]

---

静态静态工厂方法

<!-- more -->

#  序：什么是静态工厂方法
在 Java 中，获得一个类实例最简单的方法就是使用 new 关键字，通过构造函数来实现对象的创建。
就像这样：

    Fragment fragment = new MyFragment();
    // or
    Date date = new Date();

不过在实际的开发中，我们经常还会见到另外一种获取类实例的方法：

    Fragment fragment = MyFragment.newIntance();
    // or
    Calendar calendar = Calendar.getInstance();
    // or
    Integer number = Integer.valueOf("3");

 ***像这样的：不通过 new，而是用一个静态方法来对外提供自身实例的方法，即为我们所说的静态工厂方法(Static factory method)。***

## 知识点：new 究竟做了什么?

简单来说：当我们使用 new 来构造一个新的类实例时，其实是告诉了 JVM 我需要一个新的实例。JVM 就会自动在内存中开辟一片空间，然后调用构造函数来初始化成员变量，最终把引用返回给调用方。
#  Effective Java

在关于 Java 中书籍中，《Effective Java》绝对是最负盛名几本的之一，在此书中，作者总结了几十条改善 Java 程序设计的金玉良言。其中开篇第一条就是『考虑使用静态工厂方法代替构造器』，关于其原因，作者总结了几条，我们先来逐个看一下。

##  静态工厂方法与构造器不同的第一优势在于，它们有名字

由于语言的特性，Java 的构造函数都是跟类名一样的。这导致的一个问题是构造函数的名称不够灵活，经常不能准确地描述返回值，在有多个重载的构造函数时尤甚，如果参数类型、数目又比较相似的话，那更是很容易出错。

比如，如下的一段代码 ：

    Date date0 = new Date();
    Date date1 = new Date(0L);
    Date date2 = new Date("0");
    Date date3 = new Date(1,2,1);
    Date date4 = new Date(1,2,1,1,1);
    Date date5 = new Date(1,2,1,1,1,1);

Date 类有很多重载函数，对于开发者来说，假如不是特别熟悉的话，恐怕是需要犹豫一下，才能找到合适的构造函数的。而对于其他的代码阅读者来说，估计更是需要查看文档，才能明白每个参数的含义了。

（当然，Date 类在目前的 Java 版本中，只保留了一个无参和一个有参的构造函数，其他的都已经标记为 @Deprecated 了）

而如果使用静态工厂方法，就可以给方法起更多有意义的名字，比如前面的 valueOf、newInstance、getInstance 等，对于代码的编写和阅读都能够更清晰。

##  第二个优势，不用每次被调用时都创建新对象
这个很容易理解了，有时候外部调用者只需要拿到一个实例，而不关心是否是新的实例；又或者我们想对外提供一个单例时如果使用工厂方法，就可以很容易的在内部控制，防止创建不必要的对象，减少开销。

在实际的场景中，单例的写法也大都是用静态工厂方法来实现的。

如果你想对单例有更多了解，可以看一下这里：☞《Hi，我们再来聊一聊Java的单例吧》[2]
## 第三个优势，可以返回原返回类型的子类
这条不用多说，设计模式中的基本的原则之一——『里氏替换』原则，就是说子类应该能替换父类。
显然，构造方法只能返回确切的自身类型，而静态工厂方法则能够更加灵活，可以根据需要方便地返回任何它的子类型的实例。

    Class Person {
        public static Person getInstance(){
            return new Person();
            // 这里可以改为 return new Player() / Cooker()
        }
    }
    Class Player extends Person{
    }
    Class Cooker extends Person{
    }

比如上面这段代码，Person 类的静态工厂方法可以返回 Person 的实例，也可以根据需要返回它的子类 Player 或者 Cooker。（当然，这只是为了演示，在实际的项目中，一个类是不应该依赖于它的子类的。但如果这里的 getInstance () 方法位于其他的类中，就更具有的实际操作意义了）

## 静态工厂的第四个优点是返回对象的类可以根据输入参数的不同而不同

EnumSet 类（详见第 36 条）没有公共构造方法，只有静态工厂。 在 OpenJDK 实现中，它们根据底层枚举类型的大小返回两个子类中的一个的实例：大多数枚举类型具有 64 个或更少的元素，静态工厂将返回一个 RegularEnumSet 实例， 底层是long 类型；如果枚举类型具有六十五个或更多元素，则工厂将返回一个 JumboEnumSet 实例，底层是long 类型的数组。

    /**
         * Creates an empty enum set with the specified element type.
         *
         * @param <E> The class of the elements in the set
         * @param elementType the class object of the element type for this enum
         *     set
         * @return An empty enum set of the specified type.
         * @throws NullPointerException if <tt>elementType</tt> is null
         ** /
        public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
            Enum<?>[] universe = getUniverse(elementType);
            if (universe == null)
                throw new ClassCastException(elementType + " not an enum");

            if (universe.length <= 64)
                return new RegularEnumSet<>(elementType, universe);
            else
                return new JumboEnumSet<>(elementType, universe);
        }

　　这两个实现类的存在对于客户端而言是不可见的。 如果 RegularEnumSet 对于小的枚举类型不再具有性能优势，则可以在未来版本中将其淘汰，且不会产生任何不良影响。 同样，如果可以证明添加 EnumSet 的更多的实现可以提高性能，那么在未来的版本可能就会这样做。 客户既不知道也不关心他们从工厂返回的对象的类别; 他们只需要知道它是 EnumSet 的子类。

## 第五个优势是，在编写包含该方法的类时，返回的对象的类不需要存在。

这种灵活的静态工厂方法构成了服务提供者框架的基础，比如 Java 数据库连接 AP（JDBC）。服务提供者框架是提供者实现服务的系统，并且系统使得实现对客户端可用，从而将客户端从实现中分离出来。

书中这段读起来比较晦涩，对服务提供者框架在参考资料[3]中有解释，
　　
![](assets/markdown-img-paste-20191230175836971.png)

服务提供者框架中有三个基本组：服务接口，它表示实现；提供者注册 API，提供者用来注册实现；以及服务访问 API，客户端使用该 API 获取服务的实例。服务访问 API 允许客户指定选择实现的标准。在缺少这样的标准的情况下，API 返回一个默认实现的实例，或者允许客户通过所有可用的实现进行遍历。服务访问 API 是灵活的静态工厂，它构成了服务提供者框架的基础。

  Class.forName("com.mysql.jdbc.Driver");

这样一个语句会实例化一个Driver类（提供服务者实现类），并将这个类的实例注册到DriverManager（服务提供者注册类）。

  DriverManager.getConnection("jdbc:mysql://...","...","...");
这里通过建立连接的URL等信息来获取数据库连接。DriverManager通过传进来的url信息判断出你是要获取那个服务提供者提供的服务。因为前面已经将提供服务者实现类注册到DriverManager了，DriverManager获取到这个服务提供者实现类对象之后，通过调用它的getService（mysql里面是connect方法）方法获取到服务具体实现类对象，返回的却是java.sql.Connection接口对象（因为服务具体实现类实现了Connection接口），这样把服务具体实现类对象隐藏了。提供了很好的扩展性。

　　服务提供者框架的一个可选的第四个组件是一个服务提供者接口，它描述了一个生成服务接口实例的工厂对象。在没有服务提供者接口的情况下，必须对实现进行反射实例化（详见第 65 条）。在 JDBC 的情况下，Connection 扮演服务接口的一部分，DriverManager.registerDriver 提供程序注册 API、DriverManager.getConnection 是服务访问 API，Driver 是服务提供者接口。

服务提供者框架模式有许多变种。 例如，服务访问 API 可以向客户端返回比提供者提供的更丰富的服务接口。 这是桥接模式。 依赖注入框架（详见第 5 条）可以被看作是强大的服务提供者。 从 Java 6 开始，平台包含一个通用的服务提供者框架 java.util.ServiceLoader，所以你不需要，一般也不应该自己编写（详见第 59 条）。 JDBC 不使用 ServiceLoader，因为前者早于后者。

## 另一个优势，在创建带泛型的实例时，能使代码变得简洁
这条主要是针对带泛型类的繁琐声明而说的，需要重复书写两次泛型参数：

Map<String,Date> map = new HashMap<String,Date>();
不过自从 java7 开始，这种方式已经被优化过了 —— 对于一个已知类型的变量进行赋值时，由于泛型参数是可以被推导出，所以可以在创建实例时省略掉泛型参数。

Map<String,Date> map = new HashMap<>();

# 除此之外
以上是《Effective Java》中总结的几条应该使用静态工厂方法代替构造器的原因，除了上面总结的几条之外，静态工厂方法实际上还有更多的优势。

## 可以有多个参数相同但名称不同的工厂方法
构造函数虽然也可以有多个，但是由于函数名已经被固定，所以就要求参数必须有差异时（类型、数量或者顺序）才能够重载了。
举例来说：

    class Child{
        int age = 10;
        int weight = 30;
        public Child(int age, int weight) {
            this.age = age;
            this.weight = weight;
        }
        public Child(int age) {
            this.age = age;
        }
    }
Child 类有 age 和 weight 两个属性，如代码所示，它已经有了两个构造函数：Child(int age, int weight) 和 Child(int age)，这时候如果我们想再添加一个指定 wegiht 但不关心 age 的构造函数，一般是这样：

    public Child( int weight) {
        this.weight = weight;
    }
但要把这个构造函数添加到 Child 类中，我们都知道是行不通的，因为 java 的函数签名是忽略参数名称的，所以 Child(int age) 跟 Child(int weight) 会冲突。

这时候，静态工厂方法就可以登场了。

    class Child{
        int age = 10;
        int weight = 30;
        public static Child newChild(int age, int weight) {
            Child child = new Child();
            child.weight = weight;
            child.age = age;
            return child;
        }
        public static Child newChildWithWeight(int weight) {
            Child child = new Child();
            child.weight = weight;
            return child;
        }
        public static Child newChildWithAge(int age) {
            Child child = new Child();
            child.age = age;
            return child;
        }
    }
其中的 newChildWithWeight 和 newChildWithAge，就是两个参数类型相同的的方法，但是作用不同，如此，就能够满足上面所说的类似Child(int age) 跟 Child(int weight)同时存在的需求。
（另外，这两个函数名字也是自描述的，相对于一成不变的构造函数更能表达自身的含义，这也是上面所说的第一条优势 —— 『它们有名字』）

## 可以减少对外暴露的属性
软件开发中有一条很重要的经验：对外暴露的属性越多，调用者就越容易出错。所以对于类的提供者，一般来说，应该努力减少对外暴露属性，从而降低调用者出错的机会。

考虑一下有如下一个 Player 类：

    // Player : Version 1
    class Player {
        public static final int TYPE_RUNNER = 1;
        public static final int TYPE_SWIMMER = 2;
        public static final int TYPE_RACER = 3;
        protected int type;
        public Player(int type) {
            this.type = type;
        }
    }
Player 对外提供了一个构造方法，让使用者传入一个 type 来表示类型。那么这个类期望的调用方式就是这样的：

    Player player1 = new Player(Player.TYPE_RUNNER);
    Player player2 = new Player(Player.TYPE_SWEIMMER);
但是，我们知道，提供者是无法控制调用方的行为的，实际中调用方式可能是这样的：

    Player player3 = new Player(0);
    Player player4 = new Player(-1);
    Player player5 = new Player(10086);
提供者期望的构造函数传入的值是事先定义好的几个常量之一，但如果不是，就很容易导致程序错误。

—— 要避免这种错误，使用枚举来代替常量值是常见的方法之一，当然如果不想用枚举的话，使用我们今天所说的主角静态工厂方法也是一个很好的办法。

插一句：
实际上，使用枚举也有一些缺点，比如增大了调用方的成本；如果枚举类成员增加，会导致一些需要完备覆盖所有枚举的调用场景出错等。
如果把以上需求用静态工厂方法来实现，代码大致是这样的：


    // Player : Version 2
    class Player {
        public static final int TYPE_RUNNER = 1;
        public static final int TYPE_SWIMMER = 2;
        public static final int TYPE_RACER = 3;
        int type;

        private Player(int type) {
            this.type = type;
        }

        public static Player newRunner() {
            return new Player(TYPE_RUNNER);
        }
        public static Player newSwimmer() {
            return new Player(TYPE_SWIMMER);
        }
        public static Player newRacer() {
            return new Player(TYPE_RACER);
        }
    }
注意其中的构造方法被声明为了 private，这样可以防止它被外部调用，于是调用方在使用 Player 实例的时候，基本上就必须通过 newRunner、newSwimmer、newRacer 这几个静态工厂方法来创建，调用方无须知道也无须指定 type 值 —— 这样就能把 type 的赋值的范围控制住，防止前面所说的异常值的情况。

插一句：
严谨一些的话，通过反射仍能够绕过静态工厂方法直接调用构造函数，甚至直接修改一个已创建的 Player 实例的 type 值，但本文暂时不讨论这种非常规情况。
## 多了一层控制，方便统一修改
我们在开发中一定遇到过很多次这样的场景：在写一个界面时，服务端的数据还没准备好，这时候我们经常就需要自己在客户端编写一个测试的数据，来进行界面的测试，像这样：

    // 创建一个测试数据
    User tester = new User();
    tester.setName("隔壁老王");
    tester.setAge(25);
    tester.setDescription("我住隔壁我姓王！");
    // use tester
    bindUI(tester);
    ……
要写一连串的测试代码，如果需要测试的界面有多个，那么这一连串的代码可能还会被复制多次到项目的多个位置。

这种写法的缺点呢，首先是代码臃肿、混乱；其次是万一上线的时候漏掉了某一处，忘记修改，那就可以说是灾难了……

但是如果你像我一样，习惯了用静态工厂方法代替构造器的话，则会很自然地这么写，先在 User 中定义一个 newTestInstance 方法：

    static class User{
        String name ;
        int age ;
        String description;
        public static User newTestInstance() {
            User tester = new User();
            tester.setName("隔壁老王");
            tester.setAge(25);
            tester.setDescription("我住隔壁我姓王！");
            return tester;
        }
    }
然后调用的地方就可以这样写了：

    // 创建一个测试数据
    User tester = User.newTestInstance();
    // use tester
    bindUI(tester);
是不是瞬间就觉得优雅了很多？！

而且不只是代码简洁优雅，由于所有测试实例的创建都是在这一个地方，所以在需要正式数据的时候，也只需把这个方法随意删除或者修改一下，所有调用者都会编译不通过，彻底杜绝了由于疏忽导致线上还有测试代码的情况。
# 缺点：没有公共或受保护构造方法的类不能被子类化
例如，要想将 Collections 框架中任何遍历的实现类进行子类化，是不可能的。但是这样也会因祸得福，因为它鼓励程序员使用组合（composition）而不是继承（详见第 18 条），并且是不可变类型锁需要的（详见第 17 条）。

# 静态工厂方法的第二个缺点是，程序员很难找到它们

它们不像构造方法那样在 API 文档中明确的标注出来。因此，对于提供了静态方法而不是构造器的类来说，想要查明如何实例化一个类是十分困难的。

通过关注类或者接口的文档中静态方法，并且遵守标准的命名习惯，也可以弥补这一劣势。下面是一些静态工厂方法的常用名称。以下清单这是列出了其中的一小部分：

from —— 类型转换方法，它接受单个参数并返回此类型的相应实例，例如：

    Date d = Date.from(instant);

of —— 聚合方法，接受多个参数并返回该类型的实例，并把他们合并在一起，例如：

    Set faceCards = EnumSet.of(JACK, QUEEN, KING);
valueOf —— from 和 to 更为详细的替代 方式，例如：

    BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
instance 或 getinstance —— 返回一个由其参数 (如果有的话) 描述的实例，但不能说它具有相同的值，例如：

    StackWalker luke = StackWalker.getInstance(options);
create 或 newInstance —— 与 instance 或 getInstance 类似，除此之外该方法保证每次调用返回一个新的实例，例如：

    Object newArray = Array.newInstance(classObject, arrayLen);
getType —— 与 getInstance 类似，但是在工厂方法处于不同的类中的时候使用。getType 中的 Type 是工厂方法返回的对象类型，例如：

    FileStore fs = Files.getFileStore(path);
newType —— 与 newInstance 类似，但是在工厂方法处于不同的类中的时候使用。newType中的 Type 是工厂方法返回的对象类型，例如：

    BufferedReader br = Files.newBufferedReader(path);
type —— getType 和 newType 简洁的替代方式，例如：

    List litany = Collections.list(legacyLitany);

# 总结
静态工厂方法和公共构造方法都有它们的用途，并且了解它们的相对优点是值得的。通常，静态工厂更可取，因此避免在没有考虑静态工厂的情况下直接选择使用公共构造方法

总体来说，『考虑使用静态工厂方法代替构造器』这点，除了有名字、可以用子类等这些语法层面上的优势之外，更多的是在工程学上的意义，它实质上的最主要作用是：能够增大类的提供者对自己所提供的类的控制力。

作为一个开发者，当我们作为调用方，使用别人提供的类时，如果要使用 new 关键字来为其创建一个类实例，如果对类不是特别熟悉，那么一定是要特别慎重的 —— new 实在是太好用了，以致于它经常被滥用，随时随地的 new 是有很大风险的，除了可能导致性能、内存方面的问题外，也经常会使得代码结构变得混乱。

而当我们在作为类的提供方时，无法控制调用者的具体行为，但是我们可以尝试使用一些方法来增大自己对类的控制力，减少调用方犯错误的机会，这也是对代码更负责的具体体现。



# 参考资料
【1】 https://www.diycode.cc/topics/1027
【2】 https://www.jianshu.com/p/eb30a388c5fc
【3】 https://juejin.im/post/5d6a0652f265da03df5f285d
