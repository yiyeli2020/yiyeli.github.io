title: Effctive-Java阅读笔记IX
date: 2020-1-16 11:28:12
categories: 2020年1月
tags: [Java]

---

第四章阅读：类和接口

17.最小化可变性
18.组合优于继承
灵魂三问：
　如果你试图让 B 类继承 A 类时，问自己这个问题：每个 B 都是 A 吗？ 如果你不能如实回答这个问题，那么 B 就不应该继承 A。在决定使用继承来代替组合之前，你应该问自己最后一组问题。对于试图继承的类，它的 API 有没有缺陷呢？ 如果有，你是否愿意将这些缺陷传播到你的类的 API 中？继承传播父类的 API 中的任何缺陷，而组合可以让你设计一个隐藏这些缺陷的新 API。

<!-- more -->

# 最小化可变性

　　不可变类简单来说是其实例不能被修改的类。 包含在每个实例中的所有信息在对象的生命周期中是固定的，因此不会观察到任何变化。 Java 平台类库包含许多不可变的类，包括 String 类、基本类型包装类以及 BigInteger 类和 BigDecimal 类。 有很多很好的理由：不可变类比可变类更易于设计，实现和使用。 他们不容易出错，并且更安全。

　　要使一个类成为不可变类，请遵循以下五条规则：

## 不要提供修改对象状态的方法（也称为 mutators，设值方法）。
## 确保这个类不能被继承。
这可以防止粗心或者恶意的子类假装对象的状态已经改变，从而破坏类的不可变行为。 防止子类化，通常是通过 final 修饰类，但是我们稍后将讨论另一种方法。
##把所有字段设置为 final。
通过系统强制执行的方式，清楚地表达了你的意图。 另外，如果一个新创建的实例的引用在缺乏同步机制的情况下从一个线程传递到另一个线程，就必须保证正确的行为，正如内存模型[JLS，17.5; Goetz06 16] 所述。
## 把所有的字段设置为 private。
这可以防止客户端获得对字段引用的可变对象的访问权限，并直接修改这些对象。 虽然技术上允许不可变类具有包含基本类型数值的公有的 final 字段或对不可变对象的引用，但不建议这样做，因为这样使得在以后的版本中无法再改变内部的表示状态（详见第 15 和 16 条）。
## 确保对任何可变组件的互斥访问。
如果你的类有任何引用可变对象的字段，请确保该类的客户端无法获得对这些对象的引用。 切勿将这样的属性初始化为客户端提供的对象引用，或从访问方法返回属性。 在构造方法，访问方法和 readObject 方法（详见第 88 条）中进行防御性拷贝（详见第 50 条）。
　　以前条目中的许多示例类都是不可变的。 其中一个例子是条目 11 中的 PhoneNumber 类，它具有每个字段的访问方法（accessors），但没有相应的设值方法（mutators）。下面一个稍微复杂一点的例子：

    // Immutable complex number class
    public final class Complex {

        private final double re;
        private final double im;

        public Complex(double re, double im) {
            this.re = re;
            this.im = im;
        }

        public double realPart() {
            return re;
        }

        public double imaginaryPart() {
            return im;
        }

        public Complex plus(Complex c) {
            return new Complex(re + c.re, im + c.im);
        }

        public Complex minus(Complex c) {
            return new Complex(re - c.re, im - c.im);
        }

        public Complex times(Complex c) {
            return new Complex(re * c.re - im * c.im,
                    re * c.im + im * c.re);
        }

        public Complex dividedBy(Complex c) {
            double tmp = c.re * c.re + c.im * c.im;
            return new Complex((re * c.re + im * c.im) / tmp,
                    (im * c.re - re * c.im) / tmp);
        }

        @Override
        public boolean equals(Object o) {
            if (o == this) {
                return true;
            }

            if (!(o instanceof Complex)) {
                return false;
            }

            Complex c = (Complex) o;

            // See page 47 to find out why we use compare instead of ==
            return Double.compare(c.re, re) == 0
                    && Double.compare(c.im, im) == 0;
        }

        @Override
        public int hashCode() {
            return 31 * Double.hashCode(re) + Double.hashCode(im);
        }

        @Override
        public String toString() {
            return "(" + re + " + " + im + "i)";
        }
    }

　　这个类代表了一个复数（包含实部和虚部的数字）。 除了标准的 Object 方法之外，它还为实部和虚部提供访问方法，并提供四个基本的算术运算：加法，减法，乘法和除法。 注意算术运算如何创建并返回一个新的 Complex 实例，而不是修改这个实例。 这种模式被称为函数式方法，因为方法返回将操作数应用于函数的结果，而不修改它们。 与其对应的过程式的（procedural）或命令式的（imperative）的方法相对比，在这种方法中，将一个过程作用在操作数上，导致其状态改变。 请注意，方法名称是介词（如 plus）而不是动词（如 add）。 这强调了方法不会改变对象的值的事实。 BigInteger 和 BigDecimal 类没有遵守这个命名约定，并导致许多使用错误。

　　如果你不熟悉函数式方法，可能会觉得它显得不自然，但它具有不变性，具有许多优点。 不可变对象很简单。 一个不可变的对象可以完全处于一种状态，也就是被创建时的状态。 如果确保所有的构造方法都建立了类不变量，那么就保证这些不变量在任何时候都保持不变，使用此类的程序员无需再做额外的工作。 另一方面，可变对象可以具有任意复杂的状态空间。 如果文档没有提供由设置（mutator）方法执行的状态转换的精确描述，那么可靠地使用可变类可能是困难的或不可能的。

　　***不可变对象本质上是线程安全的；它们不需要同步*** 。 被多个线程同时访问它们时，不会遭到破坏。 这是实现线程安全的最简单方法。 由于没有线程可以观察到另一个线程对不可变对象的影响，所以 ***不可变对象可以被自由地共享*** 。 因此，不可变类应鼓励客户端尽可能重用现有的实例。 一个简单的方法是为常用的值提供公共的静态 final 常量。 例如，Complex 类可能提供这些常量：

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

　　这种方法可以更进一步。 一个不可变的类可以提供静态的工厂（详见第 1 条）来缓存经常被请求的实例，以避免在现有的实例中创建新的实例。 所有基本类型的包装类和 BigInteger 类都是这样做的。 使用这样的静态工厂会使客户端共享实例而不是创建新实例，从而减少内存占用和垃圾回收成本。 在设计新类时，选择静态工厂代替公共构造方法，可以在以后增加缓存的灵活性，而不需要修改客户端。

　　不可变对象可以自由分享的结果是，你永远不需要做出防御性拷贝（defensive copies）（详见第 50 条）。 事实上，永远不需要做任何拷贝，因为这些拷贝永远等于原始对象。 因此，你不需要也不应该在一个不可变的类上提供一个 clone 方法或拷贝构造方法（copy constructor）（详见第 13 条）。 这一点在 Java 平台的早期阶段还不是很好理解，所以 String 类有一个拷贝构造方法，但是它应该尽量很少使用（详见第 6 条）。

　　***不仅可以共享不可变的对象，而且可以共享内部信息*** 。 例如，BigInteger 类在内部使用符号数值表示法。 符号用 int 值表示，数值用 int 数组表示。 negate 方法生成了一个数值相同但符号相反的新 BigInteger 实例。 即使它是可变的，也不需要复制数组；新创建的 BigInteger 指向与原始相同的内部数组。

　　***不可变对象为其他对象提供了很好的构件（building blocks）***，无论是可变的还是不可变的。 如果知道一个复杂组件的内部对象不会发生改变，那么维护复杂对象的不变量就容易多了。这一原则的特例是，不可变对象可以构成 Map 对象的键和 Set 的元素，一旦不可变对象作为 Map 的键或 Set 里的元素，即使破坏了 Map 和 Set 的不可变性，但不用担心它们的值会发生变化。

　　***不可变对象无偿地提供了的原子失败机制（详见第 76 条）***。 它们的状态永远不会改变，所以不可能出现临时的不一致。

　　***不可变类的主要缺点是对于每个不同的值都需要一个单独的对象***。 创建这些对象可能代价很高，特别是是大型的对象下。 例如，假设你有一个百万位的 BigInteger ，你想改变它的低位：

    BigInteger moby = ...;
    moby = moby.flipBit(0);

　　flipBit 方法创建一个新的 BigInteger 实例，也是一百万位长，与原始位置只有一位不同。 该操作需要与 BigInteger 大小成比例的时间和空间。 将其与 java.util.BitSet 对比。 像 BigInteger 一样，BitSet 表示一个任意长度的位序列，但与 BigInteger 不同，BitSet 是可变的。 BitSet 类提供了一种方法，允许你在固定时间内更改百万位实例中单个位的状态：

    BitSet moby = ...;
    moby.flip(0);

　　如果执行一个多步操作，在每一步生成一个新对象，除最终结果之外丢弃所有对象，则性能问题会被放大。这里有两种方式来处理这个问题。第一种办法，先猜测一下会经常用到哪些多步的操作，然后讲它们作为基本类型提供。如果一个多步操作是作为一个基本类型提供的，那么不可变类就不必在每一步创建一个独立的对象。在内部，不可变的类可以是任意灵活的。 例如，BigInteger 有一个包级私有的可变的“伙伴类（companion class）”，它用来加速多步操作，比如模幂运算（modular exponentiation）。出于前面所述的所有原因，使用可变伙伴类比使用 BigInteger 要困难得多。 幸运的是，你不必使用它：BigInteger 类的实现者为你做了很多努力。

　　如果你可以准确预测客户端要在你的不可变类上执行哪些复杂的操作，那么包级私有可变伙伴类的方式可以正常工作。如果不是的话，那么最好的办法就是提供一个公开的可变伙伴类。 这种方法在 Java 平台类库中的主要例子是 String 类，它的可变伙伴类是 StringBuilder（及其过时的前身 StringBuffer 类）。

　　现在你已经知道如何创建一个不可改变类，并且了解不变性的优点和缺点，下面我们来讨论几个设计方案。 回想一下，为了保证不变性，一个类不得允许子类化。 这可以通过使类用 final 修饰，但是还有另外一个更灵活的选择。 而不是使不可变类设置为 final，***可以使其所有的构造方法私有或包级私有，并添加公共静态工厂，而不是公共构造方法***（详见第 1 条）。 为了具体说明这种方法，下面以 Complex 为例，看看如何使用这种方法：

    // Immutable class with static factories instead of constructors
    public class Complex {

        private final double re;
        private final double im;

        private Complex(double re, double im) {
            this.re = re;
            this.im = im;
        }

        public static Complex valueOf(double re, double im) {
            return new Complex(re, im);
        }

        ... // Remainder unchanged
    }

　　这种方法往往是最好的选择。 这是最灵活的，因为它允许使用多个包级私有实现类。 对于驻留在包之外的客户端，不可变类实际上是 final 的，因为不可能继承来自另一个包的类，并且缺少公共或受保护的构造方法。 除了允许多个实现类的灵活性以外，这种方法还可以通过改进静态工厂的对象缓存功能来调整后续版本中类的性能。
　　当 BigInteger 和 BigDecimal 刚被编写出来的时候，“不可变类必须是 final”的说法还没有得到广泛地理解，因此它们的所有方法都可能被重写。不幸的是，为了保持向后兼容性，这一问题无法得以纠正。如果你编写一个安全性取决于来自不受信任的客户端的 BigInteger 或 BigDecimal 参数的不变类时，则必须检查该参数是否为“真实的”BigInteger 或者 BigDecimal，而不应该是不受信任的子类的实例。如果是后者，则必须在假设可能是可变的情况下保护性拷贝（defensively copy）（详见第 50 条）：

    public static BigInteger safeInstance(BigInteger val) {
        return val.getClass() == BigInteger.class ?
                val : new BigInteger(val.toByteArray());
    }

　　在本条目开头关于不可变类的规则说明，没有方法可以修改对象，并且它的所有属性必须是 final 的。事实上，这些规则比实际需要的要强硬一些，其实可以有所放松来提高性能。 事实上，任何方法都不能在对象的状态中产生外部可见的变化。 然而，一些不可变类具有一个或多个非 final 属性，在第一次需要时将开销昂贵的计算结果缓存在这些属性中。 如果再次请求相同的值，则返回缓存的值，从而节省了重新计算的成本。 这个技巧的作用恰恰是因为对象是不可变的，这保证了如果重复的话，计算会得到相同的结果。

　　例如，PhoneNumber 类的 hashCode 方法（详见第 11 条）在第一次调用改方法时计算哈希码，并在再次调用时 对其进行缓存。 这种延迟初始化（详见第 83 条）的一个例子，String 类也使用到了。

　　关于序列化应该加上一个警告。 如果你选择使您的不可变类实现 Serializable 接口，并且它包含一个或多个引用可变对象的属性，则必须提供显式的 readObject 或 readResolve 方法，或者使用 ObjectOutputStream.writeUnshared 和 ObjectInputStream.readUnshared 方法，即默认的序列化形式也是可以接受的。 否则攻击者可能会创建一个可变的类的实例。 这个主题会在条目 88 中会详细介绍。

　　总而言之，坚决不要为每个属性编写一个 get 方法后再编写一个对应的 set 方法。***除非有充分的理由使类成为可变类，否则类应该是不可变的***。 不可变类提供了许多优点，唯一的缺点是在某些情况下可能会出现性能问题。 你应该始终使用较小的值对象（如 PhoneNumber 和 Complex），使其不可变。（Java 平台类库中有几个类，如 java.util.Date 和 java.awt.Point，本应该是不可变的，但实际上并不是）。你应该认真考虑创建更大的值对象，例如 String 和 BigInteger ，设成不可改变的。 只有当你确认有必要实现令人满意的性能（详见第 67 条）时，才应该为不可改变类提供一个公开的可变伙伴类。

　　对于一些类来说，不变性是不切实际的。***如果一个类不能设计为不可变类，那么也要尽可能地限制它的可变性*** 。减少对象可以存在的状态数量，可以更容易地分析对象，以及降低出错的可能性。因此，除非有足够的理由把属性设置为非 final 的情况下，否则应该每个属性都设置为 final 的。把本条目的建议与条目 15 的建议结合起来，你自然的倾向就是：***除非有充分的理由不这样做，否则应该把每个属性声明为私有 final 的***。

　　***构造方法应该创建完全初始化的对象，并建立所有的不变性。*** 除非有令人信服的理由，否则不要提供独立于构造方法或静态工厂的公共初始化方法。 同样，不要提供一个“reinitialize”方法，使对象可以被重用，就好像它是用不同的初始状态构建的。 这样的方法通常以增加的复杂度为代价，仅仅提供很少的性能优势。

　　CountDownLatch 类是这些原理的例证。 它是可变的，但它的状态空间有意保持最小范围内。 创建一个实例，使用它一次，并完成：一旦 countdown 锁的计数器已经达到零，不能再重用它。

　　在这个条目中，应该添加关于 Complex 类的最后一个注释。 这个例子只是为了说明不变性。 这不是一个工业强度复杂的复数实现。 它对复数使用了乘法和除法的标准公式，这些公式不正确会进行不正确的四舍五入，没有为复数的 NaN 和无穷大提供良好的语义[Kahan91，Smith62，Thomas94]。


# 组合优于继承

　　继承是实现代码重用的有效方式，但并不总是最好的工具。使用不当，会导致脆弱的软件。 在包中使用继承是安全的，其中子类和父类的实现都在同一个程序员的控制之下。对应专门为了继承而设计的，并且有文档说明的类来说（详见第 19 条），使用继承也是安全的。 然而，从普通的具体类跨越包级边界继承，是危险的。 提醒一下，本书使用「继承」一词，其含义是实现继承（当一个类扩展另一个类时）。 在本条目中讨论的问题不适用于接口继承（当类实现接口，或者当接口继承另一个接口时）。

　　***与方法调用不同，继承打破了封装[Snyder86]***。 换句话说，一个子类依赖于其父类的实现细节来保证其正确的功能。 父类的实现可能会从发布版本不断变化，如果是这样，子类可能会被破坏，即使它的代码没有任何改变。 因此，一个子类必须与其超类一起更新而变化，除非父类的作者为了继承的目的而专门设计它，并对应有文档的说明。

　　为了具体说明，假设有一个使用 HashSet 的程序。 为了调整程序的性能，需要查询 HashSet ，从创建它之后已经添加了多少个元素（不要和当前的元素数量混淆，当元素被删除时数量也会下降）。 为了提供这个功能，编写了一个 HashSet 变体，它保留了尝试元素插入的数量，并导出了这个插入数量的一个访问方法。 HashSet 类包含两个添加元素的方法，分别是 add 和 addAll，所以我们重写这两个方法：

    // Broken - Inappropriate use of inheritance!
    public class InstrumentedHashSet<E> extends HashSet<E> {
        // The number of attempted element insertions
        private int addCount = 0;

        public InstrumentedHashSet() {
        }

        public InstrumentedHashSet(int initCap, float loadFactor) {
            super(initCap, loadFactor);
        }
        @Override public boolean add(E e) {
            addCount++;
            return super.add(e);
        }
        @Override public boolean addAll(Collection<? extends E> c) {
            addCount += c.size();
            return super.addAll(c);
        }
        public int getAddCount() {
            return addCount;
        }
    }

这个类看起来很合理，但是不能正常工作。 假设创建一个实例并使用 addAll 方法添加三个元素。 顺便提一句，请注意，下面代码使用在 Java 9 中添加的静态工厂方法 List.of 来创建一个列表；如果使用的是早期版本，请改为使用 Arrays.asList：

    InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
    s.addAll(List.of("Snap", "Crackle", "Pop"));

　　我们期望 getAddCount 方法返回的结果是 3，但实际上返回了 6。哪里出来问题？在 HashSet 内部，addAll 方法是基于它的 add 方法来实现的，即使 HashSet 文档中没有指名其实现细节，倒也是合理的。InstrumentedHashSet 中的 addAll 方法首先给 addCount 属性设置为 3，然后使用 super.addAll 方法调用了 HashSet 的 addAll 实现。然后反过来又调用在 InstrumentedHashSet 类中重写的 add 方法，每个元素调用一次。这三次调用又分别给 addCount 加 1，所以，一共增加了 6：通过 addAll 方法每个增加的元素都被计算了两次。

　　我们可以通过消除 addAll 方法的重写来“修复”子类。 尽管生成的类可以正常工作，但是它依赖于它的正确方法，因为 HashSet 的 addAll 方法是在其 add 方法之上实现的。 这个“自我使用（self-use）”是一个实现细节，并不保证在 Java 平台的所有实现中都可以适用，并且可以随发布版本而变化。 因此，产生的 InstrumentedHashSet 类是脆弱的。

　　稍微好一点的做法是，重写 addAll 方法遍历指定集合，为每个元素调用 add 方法一次。 不管 HashSet 的 addAll 方法是否在其 add 方法上实现，都会保证正确的结果，因为 HashSet 的 addAll 实现将不再被调用。然而，这种技术并不能解决所有的问题。 这相当于重新实现了父类方法，这样的方法可能不能确定到底是否《时》自用（self-use）的，实现起来也是困难的，耗时的，容易出错的，并且可能会降低性能。 此外，这种方式并不能总是奏效，因为子类无法访问一些私有属性，所以有些方法就无法实现。

　　导致子类脆弱的一个相关原因是，它们的父类在后续的发布版本中可以添加新的方法。假设一个程序的安全性依赖于这样一个事实：所有被插入到集中的元素都满足一个先决条件。可以通过对集合进行子类化，然后并重写所有添加元素的方法，以确保在添加每个元素之前满足这个先决条件，来确保这一问题。如果在后续的版本中，父类没有新增添加元素的方法，那么这样做没有问题。但是，一旦父类增加了这样的新方法，则很有可能由于调用了未被重写的新方法，将非法的元素添加到子类的实例中。这不是个纯粹的理论问题。在把 Hashtable 和 Vector 类加入到 Collections 框架中的时候，就修复了几个类似性质的安全漏洞。

　　这两个问题都源于重写方法。 如果仅仅添加新的方法并且不要重写现有的方法，可能会认为继承一个类是安全的。 虽然这种扩展更为安全，但这并非没有风险。 如果父类在后续版本中添加了一个新的方法，并且你不幸给了子类一个具有相同签名和不同返回类型的方法，那么你的子类编译失败[JLS，8.4.8.3]。 如果已经为子类提供了一个与新的父类方法具有相同签名和返回类型的方法，那么你现在正在重写它，因此将遇到前面所述的问题。 此外，你的方法是否会履行新的父类方法的约定，这是值得怀疑的，因为在你编写子类方法时，这个约定还没有写出来。

　　幸运的是，有一种方法可以避免上述所有的问题。***不要继承一个现有的类，而应该给你的新类增加一个私有属性，该属性是 现有类的实例引用，这种设计被称为组合（composition）***，因为现有的类成为新类的组成部分。***新类中的每个实例方法调用现有类的包含实例上的相应方法并返回结果。这被称为转发（forwarding），而新类中的方法被称为转发方法***。由此产生的类将坚如磐石，不依赖于现有类的实现细节。即使将新的方法添加到现有的类中，也不会对新类产生影响。为了具体说用，下面代码使用组合和转发方法替代 InstrumentedHashSet 类。请注意，实现分为两部分，类本身和一个可重用的转发类，其中包含所有的转发方法，没有别的方法：

    // Reusable forwarding class
    import java.util.Collection;
    import java.util.Iterator;
    import java.util.Set;

    public class ForwardingSet<E> implements Set<E> {

        private final Set<E> s;

        public ForwardingSet(Set<E> s) {
            this.s = s;
        }

        public void clear() {
            s.clear();
        }

        public boolean contains(Object o) {
            return s.contains(o);
        }

        public boolean isEmpty() {
            return s.isEmpty();
        }

        public int size() {
            return s.size();
        }

        public Iterator<E> iterator() {
            return s.iterator();
        }

        public boolean add(E e) {
            return s.add(e);
        }

        public boolean remove(Object o) {
            return s.remove(o);
        }

        public boolean containsAll(Collection<?> c) {
            return s.containsAll(c);
        }

        public boolean addAll(Collection<? extends E> c) {
            return s.addAll(c);
        }

        public boolean removeAll(Collection<?> c) {
            return s.removeAll(c);
        }

        public boolean retainAll(Collection<?> c) {
            return s.retainAll(c);
        }

        public Object[] toArray() {
            return s.toArray();
        }

        public <T> T[] toArray(T[] a) {
            return s.toArray(a);
        }

        @Override
        public boolean equals(Object o) {
            return s.equals(o);
        }

        @Override
        public int hashCode() {
            return s.hashCode();
        }

        @Override
        public String toString() {
            return s.toString();
        }
    }


    // Wrapper class - uses composition in place of inheritance
    import java.util.Collection;
    import java.util.Set;

    public class InstrumentedSet<E> extends ForwardingSet<E> {

        private int addCount = 0;

        public InstrumentedSet(Set<E> s) {
            super(s);
        }

        @Override public boolean add(E e) {
            addCount++;
            return super.add(e);
        }

        @Override public boolean addAll(Collection<? extends E> c) {
            addCount += c.size();
            return super.addAll(c);
        }

        public int getAddCount() {
            return addCount;
        }
    }

InstrumentedSet 类的设计是通过存在的 Set 接口来实现的，该接口包含 HashSet 类的功能特性。除了功能强大，这个设计是非常灵活的。InstrumentedSet 类实现了 Set 接口，并有一个构造方法，其参数也是 Set 类型的。本质上，这个类把 Set 转换为另一个类型 Set， 同时添加了计数的功能。与基于继承的方法不同，该方法仅适用于单个具体类，并且父类中每个需要支持构造方法，提供单独的构造方法，所以可以使用包装类来包装任何 Set 实现，并且可以与任何预先存在的构造方法结合使用：

    Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
    Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));

　　InstrumentedSet 类甚至可以用于临时替换没有计数功能下使用的集合实例：

    static void walk(Set<Dog> dogs) {
        InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
        ... // Within this method use iDogs instead of dogs
    }

　　InstrumentedSet 类被称为包装类，因为每个 InstrumentedSet 实例都包含（“包装”）另一个 Set 实例。 这也被称为装饰器模式[Gamma95]，因为 InstrumentedSet 类通过添加计数功能来“装饰”一个集合。 有时组合和转发的结合被不精确地地称为委托（delegation）。 从技术上讲，除非包装对象把自身传递给被包装对象，否则不是委托[Lieberman86; Gamma95]。

　　包装类的缺点很少。 一个警告是包装类不适合在回调框架（callback frameworks）中使用，其中对象将自我引用传递给其他对象以用于后续调用（「回调」）。 因为一个被包装的对象不知道它外面的包装对象，所以它传递一个指向自身的引用（this），回调时并不记得外面的包装对象。 这被称为 SELF 问题[Lieberman86]。 有些人担心转发方法调用的性能影响，以及包装对象对内存占用。 两者在实践中都没有太大的影响。 编写转发方法有些繁琐，但是只需为每个接口编写一次可重用的转发类，并且提供转发类。 例如，Guava 为所有的 Collection 接口提供转发类[Guava]。

　　只有在子类真的是父类的子类型的情况下，继承才是合适的。 换句话说，只有在两个类之间存在「is-a」关系的情况下，B 类才能继承 A 类。 如果你试图让 B 类继承 A 类时，问自己这个问题：每个 B 都是 A 吗？ 如果你不能如实回答这个问题，那么 B 就不应该继承 A。如果答案是否定的，那么 B 通常包含一个 A 的私有实例，并且暴露一个不同的 API ：A 不是 B 的重要部分 ，只是其实现细节。

　　在 Java 平台类库中有一些明显的违反这个原则的情况。 例如，stacks 实例并不是 vector 实例，所以 Stack 类不应该继承 Vector 类。 同样，一个属性列表不是一个哈希表，所以 Properties 不应该继承 Hashtable 类。 在这两种情况下，组合方式更可取。

　　如果在合适组合的地方使用继承，则会不必要地公开实现细节。由此产生的 API 将与原始实现联系在一起，永远限制类的性能。更严重的是，通过暴露其内部，客户端可以直接访问它们。至少，它可能导致混淆语义。例如，属性 p 指向 Properties 实例，那么 p.getProperty(key) 和 p.get(key) 就有可能返回不同的结果：前者考虑了默认的属性表，而后者是继承 Hashtable 的，它则没有考虑默认属性列表。最严重的是，客户端可以通过直接修改超父类来破坏子类的不变性。在 Properties 类，设计者希望只有字符串被允许作为键和值，但直接访问底层的 Hashtable 允许违反这个不变性。一旦违反，就不能再使用属性 API 的其他部分（load 和 store 方法）。在发现这个问题的时候，纠正这个问题为时已晚，因为客户端依赖于使用非字符串键和值了。

　　在决定使用继承来代替组合之前，你应该问自己最后一组问题。对于试图继承的类，它的 API 有没有缺陷呢？ 如果有，你是否愿意将这些缺陷传播到你的类的 API 中？继承传播父类的 API 中的任何缺陷，而组合可以让你设计一个隐藏这些缺陷的新 API。

　　总之，继承是强大的，但它是有问题的，因为它违反封装。 只有在子类和父类之间存在真正的子类型关系时才适用。 即使如此，如果子类与父类不在同一个包中，并且父类不是为继承而设计的，继承可能会导致脆弱性。 为了避免这种脆弱性，使用合成和转发代替继承，特别是如果存在一个合适的接口来实现包装类。 包装类不仅比子类更健壮，而且更强大。


# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/17.%20%E6%9C%80%E5%B0%8F%E5%8C%96%E5%8F%AF%E5%8F%98%E6%80%A7
