title: Effective-Java阅读笔记X
date: 2020-1-16 14:29:12
categories: 2020年1月
tags: [Java]

---

第四章阅读：类和接口

19.要么设计继承并提供文档说明，要么禁用继承
除非知道真正需要子类，否则最好通过将类声明为 final，或者确保没有可访问的构造器来禁止类被继承。
20.接口优于抽象类
一个接口通常是定义允许多个实现的类型的最佳方式。 如果导出一个重要的接口，应该强烈考虑提供一个骨架的实现类。在可能的情况下，应该通过接口上的默认方法提供骨架实现，以便接口的所有实现者都可以使用它。即对接口的限制通常要求骨架实现类采用抽象类的形式。
<!-- more -->

# 要么设计继承并提供文档说明，要么禁用继承

　　条目 18 中提醒你注意继承没有设计和文档说明的「外来」类的子类化的危险。 那么对于专门为了继承而设计并且具有良好文档说明的类而言，这又意味着什么呢？

　　首先，这个类必须准确地描述重写每个方法带来的影响。 换句话说，该类必须文档说明可重写方法的自用性（self-use）。 对于每个 public 或者 protected 的方法，文档必须指明方法调用哪些可重写方法，以何种顺序调用的，以及每次调用的结果又是如何影响后续处理。 （重写方法，这里是指非 final 修饰的方法，无论是公开还是保护的。）更一般地说，一个类必须文档说明任何可能调用可重写方法的情况。 例如，后台线程或者静态初始化代码块可能会调用这样的方法。

　　调用可重写方法的方法在文档注释结束时包含对这些调用的描述。 这些描述在规范中特定部分，标记为「Implementation Requirements」，由 Javadoc 标签 @implSpec 生成。 这段话介绍该方法的内部工作原理。 下面是从 java.util.AbstractCollection 类的规范中拷贝的例子：

    public boolean remove(Object o)

    Removes a single instance of the specified element from this collection, if it is present (optional operation). More formally, removes an element e such that Objects.equals(o, e), if this collection contains one or more such elements. Returns true if this collection contained the specified element (or equivalently, if this collection changed as a result of the call).

    Implementation Requirements: This implementation iterates over the collection looking for the specified element. If it finds the element, it removes the element from the collection using the iterator’s remove method. Note that this implementation throws an UnsupportedOperationException if the iterator returned by this collection’s iterator method does not implement the remove method and this collection contains the specified object.

　　从该集合中删除指定元素的单个实例（如果存在，optional 实例操作）。 更广义地说，如果这个集合包含一个或多个这样的元素 e，就删除其中的一个满足 Objects.equals(o, e) 的元素 e。 如果此集合包含指定的元素（或者等同于此集合因调用而发生了更改），则返回 true。

　　实现要求： 这个实现迭代遍历集合查找指定元素。 如果找到元素，则使用迭代器的 remove 方法从集合中删除元素。 请注意，如果此集合的 iterator 方法返回的迭代器未实现 remove 方法，并且此集合包含指定的对象，则该实现将引发 UnsupportedOperationException 异常。

　　这个文档清楚地说明，重写 iterator 方法将会影响 remove 方法的行为。 它还描述了 iterator 方法返回的 Iterator 行为将如何影响 remove 方法的行为。 与条目 18 中的情况相反，在这种情况下，程序员继承 HashSet 并不能说明重写 add 方法是否会影响 addAll 方法的行为。

　　关于程序文档有句格言：好的 API 文档应该描述一个给定的方法做了什么工作，而不是描述它是如何做到的。那么，上面这种做法是否违背了这句格言呢？是的，它确实违背了！这正是继承破坏了封装性所带来的不幸后果。所以，为了设计一个类的文档，以便它能够被安全地子类化，你必须描述清楚那些有可能未定义的实现细节。

　　@implSpec 标签是在 Java 8 中添加的，并且在 Java 9 中被大量使用。这个标签应该默认启用，但是从 Java 9 开始，除非通过命令行开关-tag "apiNote:a:API Note:"，否则 Javadoc 工具仍然会忽略它。

　　为了继承而进行的设计不仅仅涉及自用模式的文档设计。为了使程序员能够编写出更加有效的子类，而无须承受不必要的痛苦，类必须以精心挑选的 protected 方法的形式，提供适当的钩子（hook），以便进入其内部工作中。或者在罕见的情况下，提供受保护的属性。 例如，考虑 java.util.AbstractList 中的 removeRange 方法：


    protected void removeRange(int fromIndex, int toIndex)

    Removes from this list all of the elements whose index is between fromIndex, inclusive, and toIndex, exclusive. Shifts any succeeding elements to the left (reduces their index). This call shortens the list by (toIndex - fromIndex) elements. (If toIndex == fromIndex, this operation has no effect.)

    This method is called by the clear operation on this list and its sublists. Overriding this method to take advantage of the internals of the list implementation can substantially improve the performance of the clear operation on this list and its sublists.

    Implementation Requirements: This implementation gets a list iterator positioned before fromIndex and repeatedly calls ListIterator.nextfollowed by ListIterator.remove, until the entire range has been removed. Note: If ListIterator.remove requires linear time, this implementation requires quadratic time.

    Parameters:
        fromIndex    index of first element to be removed.
        toIndex      index after last element to be removed.

　　从此列表中删除索引介于 fromIndex（包含）和 inclusive（不含）之间的所有元素。 将任何后续元素向左移（减少索引）。 这个调用通过（toIndex - fromIndex）元素来缩短列表。 （如果 toIndex == fromIndex，则此操作无效。）

　　这个方法是通过列表及其子类的 clear 操作来调用的。重写这个方法利用列表内部实现的优势，可以大大提高列表和子类的 clear 操作性能。

　　实现要求： 这个实现获取一个列表迭代器，它位于 fromIndex 之前，并重复调用 ListIterator.remove 和 ListIterator.next 方法，直到整个范围被删除。 注意：如果 ListIterator.remove 需要线性时间，则此实现需要平方级时间。

参数：
　　fromIndex 要移除的第一个元素的索引
　　toIndex 要移除的最后一个元素之后的索引

　　这个方法对 List 实现的最终用户来说是没有意义的。 它仅仅是为了使子类很容易提供一个快速 clear 方法。 在没有 removeRange 方法的情况下，当在子列表上调用 clear 方法，子类将不得不使用平方级的时间，否则，或从头重写整个 subList 机制——这不是一件容易的事情！

　　那么当你设计一个继承类的时候，你如何决定暴露哪些的受保护的成员呢？ 不幸的是，没有灵丹妙药。 所能做的最好的就是努力思考，做出最好的测试，然后通过编写子类来进行测试。 应该尽可能少地暴露受保护的成员，因为每个成员都表示对实现细节的承诺。 另一方面，你不能暴露太少，因为失去了保护的成员会导致一个类几乎不能用于继承。

　　***测试为继承而设计的类的唯一方法是编写子类***。 如果你忽略了一个关键的受保护的成员，试图编写一个子类将会使得遗漏痛苦地变得明显。 相反，如果编写的几个子类，而且没有一个使用受保护的成员，那么应该将其设为私有。 经验表明，三个子类通常足以测试一个可继承的类。 这些子类应该由父类作者以外的人编写。

　　当你为继承设计一个可能被广泛使用的类的时候，要意识到你永远承诺你文档说明的自用模式以及隐含在其保护的方法和属性中的实现决定。 这些承诺可能会使后续版本中改善类的性能或功能变得困难或不可能。 因此， 在发布它之前，你必须通过编写子类来测试你的类。

　　另外，请注意，继承所需的特殊文档混乱了正常的文档，这是为创建类的实例并在其上调用方法的程序员设计的。 在撰写本文时，几乎没有工具将普通的 API 文档从和仅仅针对子类实现的信息，分离出来。

　　还有一些类必须遵守允许继承的限制。 ***构造方法绝不能直接或间接调用可重写的方法***。 如果违反这个规则，将导致程序失败。 父类构造方法在子类构造方法之前运行，所以在子类构造方法运行之前，子类中的重写方法被调用。 如果重写方法依赖于子类构造方法执行的任何初始化，则此方法将不会按预期运行。 为了具体说明，这是一个违反这个规则的类：

    public class Super {
        // Broken - constructor invokes an overridable method
        public Super() {
            overrideMe();
        }
        public void overrideMe() {
        }
    }

以下是一个重写 overrideMe 方法的子类，Super 类的唯一构造方法会错误地调用它：

    public final class Sub extends Super {
        // Blank final, set by constructor
        private final Instant instant;

        Sub() {
            instant = Instant.now();
        }

        // Overriding method invoked by superclass constructor
        @Override
        public void overrideMe() {
            System.out.println(instant);
        }

        public static void main(String[] args) {
            Sub sub = new Sub();
            sub.overrideMe();
        }
    }

　　你可能期望这个程序打印两次 instant 实例，但是它第一次打印出 null，因为在 Sub 构造方法有机会初始化 instant 属性之前，overrideMe 被 Super 构造方法调用。 请注意，这个程序观察两个不同状态的 final 属性！ 还要注意的是，如果 overrideMe 方法调用了 instant 实例中任何方法，那么当父类构造方法调用 overrideMe 时，它将抛出一个 NullPointerException 异常。 这个程序不会抛出 NullPointerException 的唯一原因是 println 方法容忍 null 参数。

　　请注意，从构造方法中调用私有方法，其中任何一个方法都不可重写的，那么 final 方法和静态方法是安全的。

　　Cloneable 和 Serializable 接口在设计继承时会带来特殊的困难。 对于为继承而设计的类来说，实现这些接口通常不是一个好主意，因为这会给继承类的程序员带来很大的负担。 然而，可以采取特殊的行动来允许子类实现这些接口，而不需要强制这样做。 这些操作在条目 13 和条目 86 中有描述。

　　如果你决定在为继承而设计的类中实现 Cloneable 或 Serializable 接口，那么应该知道，由于 clone 和 readObject 方法与构造方法相似，所以也有类似的限制： clone 和 readObject ***都不会直接或间接调用可重写的方法***。 在 readObject 的情况下，重写方法将在子类的状态被反序列化之前运行。 在 clone 的情况下，重写方法将在子类的 clone 方法有机会修复克隆的状态之前运行。 在任何一种情况下，都可能会出现程序故障。 在 clone 的情况下，故障可能会损坏原始对象以及被克隆对象本身。 例如，如果重写方法假定它正在修改对象的深层结构的拷贝，但是尚未创建拷贝，则可能发生这种情况。

　　最后，如果你决定在为继承设计的类中实现 Serializable 接口，并且该类有一个 readResolve 或 writeReplace 方法，则必须使 readResolve 或 writeReplace 方法设置为受保护而不是私有。 如果这些方法是私有的，它们将被子类无声地忽略。 这是另一种情况，把实现细节成为类的 API 的一部分，以允许继承。

　　到目前为止，***设计一个继承类需要很大的努力，并且对这个类有很大的限制。*** 这不是一个轻率的决定。 有些情况显然是正确的，比如抽象类，包括接口的骨架实现（skeletal implementations）（详见第 20 条）。 还有其他的情况显然是错误的，比如不可变的类（详见第 17 条）。

　　但是普通的具体类呢？ 传统上，它们既不是 final 的，也不是为了子类化而设计和文档说明的，但是这种情况是危险的。每次修改这样的类，则继承此类的子类将被破坏。 这不仅仅是一个理论问题。 在修改非 final 的具体类的内部之后，接收与子类相关的错误报告并不少见，这些类没有为继承而设计和文档说明。

　　***解决这个问题的最好办法是，在没有想要安全地子类化的设计和文档说明的类中禁止子类化。*** 有两种方法禁止子类化。 两者中较容易的是声明类为 final。 另一种方法是使所有的构造方法都是私有的或包级私有的，并且添加公共静态工厂来代替构造方法。 这个方案在内部提供了使用子类的灵活性，在条目 17 中讨论过。两种方法都是可以接受的。

　　这个建议可能有些争议，因为许多程序员已经习惯于继承普通的具体类来增加功能，例如通知和同步等功能，或限制原有类的功能。 如果一个类实现了捕获其本质的一些接口，比如 Set，List 或 Map，那么不应该为了禁止子类化而感到愧疚。 在条目 18 中描述的包装类模式为增强功能提供了继承的优越选择。

　　如果一个具体的类没有实现一个标准的接口，那么你可能会通过禁止继承来给一些程序员带来不便。 如果你觉得你必须允许从这样的类继承，一个合理的方法是确保类从不调用任何可重写的方法，并文档说明这个事实。 换句话说，完全消除类的自用（self-use）的可重写的方法。 这样做，你将创建一个合理安全的子类。 重写一个方法不会影响任何其他方法的行为。

　　你可以机械地消除类的自我使用的重写方法，而不会改变其行为。 将每个可重写的方法的主体移动到一个私有的“帮助器方法”，并让每个可重写的方法调用其私有的帮助器方法。 然后用直接调用可重写方法的专用帮助器方法来替换每个自用的可重写方法。

　　简而言之，专门为了继承而设计类是一件很辛苦的工作。你必须建立文档说明其所有的自用模式，并且一旦建立了文档，在这个类的整个生命周期中都必须遵守。如果没有做到，子类就会依赖父类的实现细节，如果父类的实现发生了变化，它就有可能遭到破坏。为了允许其他人能编写出高效的子类，还你必须导出一个或者多个受保护的方法。***除非知道真正需要子类，否则最好通过将类声明为 final，或者确保没有可访问的构造器来禁止类被继承。***

# 接口优于抽象类

　　Java 有两种机制来定义允许多个实现的类型：接口和抽象类。 由于在 Java 8 [JLS 9.4.3] 中引入了接口的默认方法（default methods ），因此这两种机制都允许为某些实例方法提供实现。 一个主要的区别是要实现由抽象类定义的类型，类必须是抽象类的子类。 因为 Java 只允许单一继承，所以对抽象类的这种限制严格限制了它们作为类型定义的使用。 任何定义所有必需方法并服从通用约定的类都可以实现一个接口，而不管类在类层次结构中的位置。

　　现有的类可以很容易地进行改进来实现一个新的接口。 你只需添加所需的方法（如果尚不存在的话），并向类声明中添加一个 implements 子句。 例如，当 Comparable, Iterable， 和 Autocloseable 接口添加到 Java 平台时，很多现有类需要实现它们来加以改进。 一般来说，现有的类不能改进以继承一个新的抽象类。 如果你想让两个类继承相同的抽象类，你必须把它放在类型层级结构中的上面位置，它是两个类的祖先。 不幸的是，这会对类型层级结构造成很大的附带损害，迫使新的抽象类的所有后代对它进行子类化，无论这些后代类是否合适。

　　接口是定义混合类型（mixin）的理想选择。 一般来说，mixin 是一个类，除了它的“主类型”之外，还可以声明它提供了一些可选的行为。 例如，Comparable 是一个类型接口，它允许一个类声明它的实例相对于其他可相互比较的对象是有序的。 这样的接口被称为类型，因为它允许可选功能被“混合”到类型的主要功能。 抽象类不能用于定义混合类，这是因为它们不能被加载到现有的类中：一个类不能有多个父类，并且在类层次结构中没有合理的位置来插入一个类型。

　　接口允许构建非层级类型的框架。 类型层级对于组织某些事物来说是很好的，但是其他的事物并不是整齐地落入严格的层级结构中。 例如，假设我们有一个代表歌手的接口，和另一个代表作曲家的接口：

    public interface Singer {
        AudioClip sing(Song s);
    }

    public interface Songwriter {
        Song compose(int chartPosition);
    }

　　在现实生活中，一些歌手也是作曲家。 因为我们使用接口而不是抽象类来定义这些类型，所以单个类实现歌手和作曲家两个接口是完全允许的。 事实上，我们可以定义一个继承歌手和作曲家的第三个接口，并添加适合于这个组合的新方法：

    public interface SingerSongwriter extends Singer, Songwriter {
        AudioClip strum();
        void actSensitive();
    }

　　你并不总是需要这种灵活性，但是当你这样做的时候，接口是一个救星。 另一种方法是对于每个受支持的属性组合，包含一个单独的类的臃肿类层级结构。 如果类型系统中有 n 个属性，则可能需要支持 2n 种可能的组合。 这就是所谓的组合爆炸（combinatorial explosion）。 臃肿的类层级结构可能会导致具有许多方法的臃肿类，这些方法仅在参数类型上有所不同，因为类层级结构中没有类型来捕获通用行为。

　　接口通过包装类模式确保安全的，强大的功能增强成为可能（详见第 18 条）。 如果使用抽象类来定义类型，那么就让程序员想要添加功能，只能继承。 生成的类比包装类更弱，更脆弱。

　　当其他接口方法有明显的接口方法实现时，可以考虑向程序员提供默认形式的方法实现帮助。 有关此技术的示例，请参阅第 104 页的 removeIf 方法。如果提供默认方法，请确保使用@implSpec Javadoc 标记（条目 19）将它们文档说明为继承。

　　使用默认方法可以提供实现帮助多多少少是有些限制的。 尽管许多接口指定了 Object 类中方法（如 equals 和 hashCode）的行为，但不允许为它们提供默认方法。 此外，接口不允许包含实例属性或非公共静态成员（私有静态方法除外）。 最后，不能将默认方法添加到不受控制的接口中。

　　但是，你可以通过提供一个抽象的骨架实现类（abstract skeletal implementation class）来与接口一起使用，将接口和抽象类的优点结合起来。 接口定义了类型，可能提供了一些默认的方法，而骨架实现类在原始接口方法的顶层实现了剩余的非原始接口方法。 继承骨架实现需要大部分的工作来实现一个接口。 这就是模板方法设计模式[Gamma95]。

　　按照惯例，骨架实现类被称为 AbstractInterface，其中 Interface 是它们实现的接口的名称。 例如，集合框架（ Collections Framework）提供了一个框架实现以配合每个主要集合接口：AbstractCollection，AbstractSet，AbstractList 和 AbstractMap。 可以说，将它们称为 SkeletalCollection，SkeletalSet，SkeletalList 和 SkeletalMap 是有道理的，但是现在已经确立了抽象约定。 如果设计得当，骨架实现（无论是单独的抽象类还是仅由接口上的默认方法组成）可以使程序员非常容易地提供他们自己的接口实现。 例如，下面是一个静态工厂方法，在 AbstractList 的顶层包含一个完整的功能齐全的 List 实现：

    // Concrete implementation built atop skeletal implementation
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);
        // The diamond operator is only legal here in Java 9 and later
        // If you're using an earlier release, specify <Integer>
        return new AbstractList<>() {
            @Override
            public Integer get(int i) {
                return a[i];  // Autoboxing ([Item 6](https://www.safaribooksonline.com/library/view/effective-java-third/9780134686097/ch2.xhtml#lev6))
            }

            @Override
            public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // Auto-unboxing
                return oldVal;  // Autoboxing
            }

            @Override
            public int size() {
                return a.length;
            }
        };

    }

　　当你考虑一个 List 实现为你做的所有事情时，这个例子是一个骨架实现的强大的演示。 顺便说一句，这个例子是一个适配器（Adapter）[Gamma95]，它允许一个 int 数组被看作 Integer 实例列表。 由于 int 值和整数实例（装箱和拆箱）之间的来回转换，其性能并不是非常好。 请注意，实现采用匿名类的形式（详见第 24 条）。

　　骨架实现类的优点在于，它们提供抽象类的所有实现的帮助，而不会强加抽象类作为类型定义时的严格约束。对于具有骨架实现类的接口的大多数实现者来说，继承这个类是显而易见的选择，但它不是必需的。如果一个类不能继承骨架的实现，这个类可以直接实现接口。该类仍然受益于接口本身的任何默认方法。此外，骨架实现类仍然可以协助接口的实现。实现接口的类可以将接口方法的调用转发给继承骨架实现的私有内部类的包含实例。这种被称为模拟多重继承的技术与条目 18 讨论的包装类模式密切相关。它提供了多重继承的许多好处，同时避免了缺陷。

　　编写一个骨架的实现是一个相对简单的过程，虽然有些乏味。 首先，研究接口，并确定哪些方法是基本的，其他方法可以根据它们来实现。 这些基本方法是你的骨架实现类中的抽象方法。 接下来，为所有可以直接在基本方法之上实现的方法提供接口中的默认方法，回想一下，你可能不会为诸如 Object 类中 equals 和 hashCode 等方法提供默认方法。 如果基本方法和默认方法涵盖了接口，那么就完成了，并且不需要骨架实现类。 否则，编写一个声明实现接口的类，并实现所有剩下的接口方法。 为了适合于该任务，此类可能包含任何的非公共属性和方法。

　　作为一个简单的例子，考虑一下 Map.Entry 接口。 显而易见的基本方法是 getKey，getValue 和（可选的）setValue。 接口指定了 equals 和 hashCode 的行为，并且在基本方面方面有一个 toString 的明显的实现。 由于不允许为 Object 类方法提供默认实现，因此所有实现均放置在骨架实现类中：

    // Skeletal implementation class
    public abstract class AbstractMapEntry<K,V>
            implements Map.Entry<K,V> {

        // Entries in a modifiable map must override this method
        @Override public V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        // Implements the general contract of Map.Entry.equals
        @Override
        public boolean equals(Object o) {
            if (o == this)
                return true;
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry) o;
            return Objects.equals(e.getKey(),  getKey())
                && Objects.equals(e.getValue(), getValue());
        }

        // Implements the general contract of Map.Entry.hashCode
        @Override
        public int hashCode() {
            return Objects.hashCode(getKey())
                 ^ Objects.hashCode(getValue());
        }

        @Override
        public String toString() {
            return getKey() + "=" + getValue();
        }

    }

　　请注意，这个骨架实现不能在 Map.Entry 接口中实现，也不能作为子接口实现，因为默认方法不允许重写诸如 equals，hashCode 和 toString 等 Object 类方法。

　　由于骨架实现类是为了继承而设计的，所以你应该遵循条目 19 中的所有设计和文档说明。为了简洁起见，前面的例子中省略了文档注释，但是好的文档在骨架实现中是绝对必要的，无论它是否包含 一个接口或一个单独的抽象类的默认方法。

　　与骨架实现有稍许不同的是简单实现，以 AbstractMap.SimpleEntry 为例。 一个简单的实现就像一个骨架实现，它实现了一个接口，并且是为了继承而设计的，但是它的不同之处在于它不是抽象的：它是最简单的工作实现。 你可以按照情况使用它，也可以根据情况进行子类化。

　　总而言之，一个接口通常是定义允许多个实现的类型的最佳方式。 如果你导出一个重要的接口，应该强烈考虑提供一个骨架的实现类。 在可能的情况下，应该通过接口上的默认方法提供骨架实现，以便接口的所有实现者都可以使用它。 也就是说，对接口的限制通常要求骨架实现类采用抽象类的形式。

# 参考资料
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/19.%20%E8%A6%81%E4%B9%88%E8%AE%BE%E8%AE%A1%E7%BB%A7%E6%89%BF%E5%B9%B6%E6%8F%90%E4%BE%9B%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E%EF%BC%8C%E8%A6%81%E4%B9%88%E7%A6%81%E7%94%A8%E7%BB%A7%E6%89%BF
