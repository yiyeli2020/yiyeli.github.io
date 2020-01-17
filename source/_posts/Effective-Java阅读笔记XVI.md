title: Effective-Java阅读笔记XVI
date: 2020-1-17 01:33:12
categories: 2020年1月
tags: [Java]

---

第五章阅读： 泛型

31.使用限定通配符来增加 API 的灵活性
记住基本规则： producer-extends, consumer-super（PECS）。 还要记住，所有 Comparable 和 Comparator 都是消费者。如果一个参数化类型代表一个 T 生产者，使用 <? extends T>；如果它代表 T 消费者，则使用 <? super T>。
32.合理地结合泛型和可变参数
可变参数和泛型不能很好地交互，因为可变参数机制是在数组上面构建的脆弱的抽象，并且数组具有与泛型不同的类型规则。 虽然泛型可变参数不是类型安全的，但它们是合法的。 如果选择使用泛型（或参数化）可变参数编写方法，请首先确保该方法是类型安全的，然后使用 @SafeVarargs 注解对其进行标注

<!-- more -->

# 使用限定通配符来增加 API 的灵活性
　　如条目 28 所述，参数化类型是不变的。换句话说，对于任何两个不同类型的 Type1 和 Type，List<Type1> 既不是 List<Type2> 子类型也不是其父类型。尽管 List<String> 不是 List<Object> 的子类型是违反直觉的，但它确实是有道理的。 可以将任何对象放入 List<Object> 中，但是只能将字符串放入 List<String> 中。 由于 List<String> 不能做 List<Object> 所能做的所有事情，所以它不是一个子类型（条目 10 中的里氏替代原则）。

　　相对于提供的不可变的类型，有时你需要比此更多的灵活性。 考虑条目 29 中的 Stack 类。下面是它的公共 API：

    public class Stack<E> {

        public Stack();

        public void push(E e);

        public E pop();

        public boolean isEmpty();

    }
　　假设我们想要添加一个方法来获取一系列元素，并将它们全部推送到栈上。 以下是第一种尝试：

    // pushAll method without wildcard type - deficient!
    public void pushAll(Iterable<E> src) {
        for (E e : src)
            push(e);
    }
　　这种方法可以干净地编译，但不完全令人满意。 如果可遍历的 src 元素类型与栈的元素类型完全匹配，那么它工作正常。 但是，假设有一个 Stack<Number>，并调用 push(intVal)，其中 intVal 的类型是 Integer。 这是因为 Integer 是 Number 的子类型。 从逻辑上看，这似乎也应该起作用：

    Stack<Number> numberStack = new Stack<>();
    Iterable<Integer> integers = ... ;
    numberStack.pushAll(integers);
　　但是，如果你尝试了，会得到这个错误消息，因为参数化类型是不变的：

    StackTest.java:7: error: incompatible types: Iterable<Integer>
    cannot be converted to Iterable<Number>
            numberStack.pushAll(integers);
                                ^
　　幸运的是，有对应的解决方法。 该语言提供了一种特殊的参数化类型来调用一个限定通配符类型来处理这种情况。 pushAll 的输入参数的类型不应该是「E 的 Iterable 接口」，而应该是「E 的某个子类型的 Iterable 接口」，并且有一个通配符类型，这意味着：Iterable<? extends E>。 （关键字 extends 的使用有点误导：回忆条目 29 中，子类型被定义为每个类型都是它自己的子类型，即使它本身没有继承。）让我们修改 pushAll 来使用这个类型：

    // Wildcard type for a parameter that serves as an E producer
    public void pushAll(Iterable<? extends E> src) {
        for (E e : src)
            push(e);
    }
　　有了这个改变，Stack 类不仅可以干净地编译，而且客户端代码也不会用原始的 pushAll 声明编译。 因为 Stack 和它的客户端干净地编译，你知道一切都是类型安全的。

　　现在假设你想写一个 popAll 方法，与 pushAll 方法相对应。 popAll 方法从栈中弹出每个元素并将元素添加到给定的集合中。 以下是第一次尝试编写 popAll 方法的过程：

    // popAll method without wildcard type - deficient!
    public void popAll(Collection<E> dst) {
        while (!isEmpty())
            dst.add(pop());
    }

　　同样，如果目标集合的元素类型与栈的元素类型完全匹配，则干净编译并且工作正常。 但是，这又不完全令人满意。 假设你有一个 Stac<Number> 和 Object 类型的变量。 如果从栈中弹出一个元素并将其存储在该变量中，它将编译并运行而不会出错。 所以你也不能这样做吗？

    Stack<Number> numberStack = new Stack<Number>();

    Collection<Object> objects = ... ;

    numberStack.popAll(objects);

　　如果尝试将此客户端代码与之前显示的 popAll 版本进行编译，则会得到与我们的第一版 pushAll 非常类似的错误：Collection<Object> 不是 Collection<Number> 的子类型。 通配符类型再一次提供了一条出路。 popAll 的输入参数的类型不应该是「E 的集合」，而应该是「E 的某个父类型的集合」（其中父类型被定义为 E 是它自己的父类型[JLS，4.10]）。 再次，有一个通配符类型，正是这个意思：Collection<? super E>。 让我们修改 popAll 来使用它：

    // Wildcard type for parameter that serves as an E consumer
    public void popAll(Collection<? super E> dst) {
        while (!isEmpty())
            dst.add(pop());
    }

　　通过这个改动，Stack 类和客户端代码都可以干净地编译。
　　这个结论很清楚。***为了获得最大的灵活性，对代表生产者或消费者的输入参数使用通配符类型。*** 如果一个输入参数既是一个生产者又是一个消费者，那么通配符类型对你没有好处：你需要一个精确的类型匹配，这就是没有任何通配符的情况。

　　这里有一个助记符来帮助你记住使用哪种通配符类型： PECS 代表： producer-extends，consumer-super。

　　换句话说，如果一个参数化类型代表一个 T 生产者，使用 <? extends T>；如果它代表 T 消费者，则使用 <? super T>。 在我们的 Stack 示例中，pushAll 方法的 src 参数生成栈使用的 E 实例，因此 src 的合适类型为 Iterable<? extends E>；popAll 方法的 dst 参数消费 Stack 中的 E 实例，因此 dst 的合适类型是 Collection <? super E>。 PECS 助记符抓住了使用通配符类型的基本原则。 Naftalin 和 Wadler 称之为获取和放置原则（Get and Put Principle）[Naftalin07,2.4]。

　　记住这个助记符之后，让我们来看看本章中以前项目的一些方法和构造方法声明。 条目 28 中的 Chooser 类构造方法有这样的声明：

    public Chooser(Collection<T> choices)

　　这个构造方法只使用集合选择来生产类型 T 的值（并将它们存储起来以备后用），所以它的声明应该使用一个 extends T 的通配符类型。下面是得到的构造方法声明：

    // Wildcard type for parameter that serves as an T producer

    public Chooser(Collection<? extends T> choices)

　　这种改变在实践中会有什么不同吗？ 是的，会有不同。 假你有一个 List<Integer>，并且想把它传递给 Chooser<Number> 的构造方法。 这不会与原始声明一起编译，但是它只会将限定通配符类型添加到声明中。

　　现在看看条目 30 中的 union 方法。下是声明：

    public static <E> Set<E> union(Set<E> s1, Set<E> s2)

　　两个参数 s1 和 s2 都是 E 的生产者，所以 PECS 助记符告诉我们该声明应该如下：

    public static <E> Set<E> union(Set<? extends E> s1,  Set<? extends E> s2)

　　请注意，返回类型仍然是 Set<E>。 不要使用限定通配符类型作为返回类型。除了会为用户提供额外的灵活性，还强制他们在客户端代码中使用通配符类型。 通过修改后的声明，此代码将清晰地编译：

    Set<Integer>  integers =  Set.of(1, 3, 5);

    Set<Double>   doubles  =  Set.of(2.0, 4.0, 6.0);

    Set<Number>   numbers  =  union(integers, doubles);

　　如果使用得当，类的用户几乎不会看到通配符类型。 他们使方法接受他们应该接受的参数，拒绝他们应该拒绝的参数。 如果一个类的用户必须考虑通配符类型，那么它的 API 可能有问题。
　　在 Java 8 之前，类型推断规则不够聪明，无法处理先前的代码片段，这要求编译器使用上下文指定的返回类型（或目标类型）来推断 E 的类型。union 方法调用的目标类型如前所示是 Set<Number>。 如果尝试在早期版本的 Java 中编译片段（以及适合的 Set.of 工厂替代版本），将会看到如此长的错综复杂的错误消息：


    Union.java:14: error: incompatible types
            Set<Number> numbers = union(integers, doubles);
                                       ^
      required: Set<Number>
      found:    Set<INT#1>
      where INT#1,INT#2 are intersection types:
        INT#1 extends Number,Comparable<? extends INT#2>
        INT#2 extends Number,Comparable<?>
　　幸运的是有办法来处理这种错误。 如果编译器不能推断出正确的类型，你可以随时告诉它使用什么类型的显式类型参数[JLS，15.12]。 甚至在 Java 8 中引入目标类型之前，这不是你必须经常做的事情，这很好，因为显式类型参数不是很漂亮。 通过添加显式类型参数，如下所示，代码片段在 Java 8 之前的版本中进行了干净编译：

    // Explicit type parameter - required prior to Java 8
    Set<Number> numbers = Union.<Number>union(integers, doubles);

　　接下来让我们把注意力转向条目 30 中的 max 方法。这里是原始声明：

    public static <T extends Comparable<T>> T max(List<T> list)

　为了从原来到修改后的声明，我们两次应用了 PECS。首先直接的应用是参数列表。 它生成 T 实例，所以将类型从 List<T> 更改为 List<? extends T>。 棘手的应用是类型参数 T。这是我们第一次看到通配符应用于类型参数。 最初，T 被指定为继承 Comparable<T>，但 Comparable 的 T 消费 T 实例（并生成指示顺序关系的整数）。 因此，参数化类型 Comparable<T> 被替换为限定通配符类型 Comparable<? super T>。 Comparable 实例总是消费者，所以通常应该使用 Comparable<? super T> 优于 Comparable<T>。 Comparator 也是如此。因此，通常应该使用 Comparator<? super T> 优于 Comparator<T>。
　　修改后的 max 声明可能是本书中最复杂的方法声明。 增加的复杂性是否真的起作用了吗？ 同样，它的确如此。 这是一个列表的简单例子，它被原始声明排除，但在被修改后的版本里是允许的：

    List<ScheduledFuture<?>> scheduledFutures = ... ;
　　无法将原始方法声明应用于此列表的原因是 ScheduledFuture 不实现 Comparable<ScheduledFuture>。 相反，它是 Delayed 的子接口，它继承了 Comparable<Delayed>。 换句话说，一个 ScheduledFuture 实例不仅仅和其他的 ScheduledFuture 实例相比较： 它可以与任何 Delayed 实例比较，并且足以导致原始的声明拒绝它。 更普遍地说，通配符要求来支持没有直接实现 Comparable（或 Comparator）的类型，但继承了一个类型。

　　还有一个关于通配符相关的话题。 类型参数和通配符之间具有双重性，许多方法可以用一个或另一个声明。 例如，下面是两个可能的声明，用于交换列表中两个索引项目的静态方法。 第一个使用无限制类型参数（条目 30），第二个使用无限制通配符：

    // Two possible declarations for the swap method
    public static <E> void swap(List<E> list, int i, int j);
    public static void swap(List<?> list, int i, int j);

　　这两个声明中的哪一个更可取，为什么？ 在公共 API 中，第二个更好，因为它更简单。 你传入一个列表（任何列表），该方法交换索引的元素。 没有类型参数需要担心。 通常， ***如果类型参数在方法声明中只出现一次，请将其替换为通配符。*** 如果它是一个无限制的类型参数，请将其替换为无限制的通配符; 如果它是一个限定类型参数，则用限定通配符替换它。

　　第二个 swap 方法声明有一个问题。 这个简单的实现不会编译：

    public static void swap(List<?> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }

　　试图编译它会产生这个不太有用的错误信息：

    Swap.java:5: error: incompatible types: Object cannot be
    converted to CAP#1
            list.set(i, list.set(j, list.get(i)));
                                            ^
      where CAP#1 is a fresh type-variable:
        CAP#1 extends Object from capture of ?

　　看起来我们不能把一个元素放回到我们刚刚拿出来的列表中。 问题是列表的类型是 List<?>，并且不能将除 null 外的任何值放入 List<?> 中。 幸运的是，有一种方法可以在不使用不安全的转换或原始类型的情况下实现此方法。 这个想法是写一个私有辅助方法来捕捉通配符类型。 辅助方法必须是泛型方法才能捕获类型。 以下是它的定义：

    public static void swap(List<?> list, int i, int j) {
        swapHelper(list, i, j);
    }

    // Private helper method for wildcard capture
    private static <E> void swapHelper(List<E> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }

　　swapHelper 方法知道该列表是一个 List<E>。 因此，它知道从这个列表中获得的任何值都是 E 类型，并且可以安全地将任何类型的 E 值放入列表中。 这个稍微复杂的 swap 的实现可以干净地编译。 它允许我们导出基于通配符的漂亮声明，同时利用内部更复杂的泛型方法。 swap 方法的客户端不需要面对更复杂的 swapHelper 声明，但他们从中受益。 辅助方法具有我们认为对公共方法来说过于复杂的签名。

　　总之，在你的 API 中使用通配符类型，虽然棘手，但使得 API 更加灵活。 如果编写一个将被广泛使用的类库，正确使用通配符类型应该被认为是强制性的。 记住基本规则： producer-extends, consumer-super（PECS）。 还要记住，所有 Comparable 和 Comparator 都是消费者。

# 合理地结合泛型和可变参数

　　在 Java 5 中，可变参数方法（详见第 53 条）和泛型都被添加到平台中，所以你可能希望它们能够正常交互; 可悲的是，他们并没有。 可变参数的目的是允许客户端将一个可变数量的参数传递给一个方法，但这是一个脆弱的抽象（leaky abstraction）：当你调用一个可变参数方法时，会创建一个数组来保存可变参数；那个应该是实现细节的数组是可见的。 因此，当可变参数具有泛型或参数化类型时，会导致编译器警告混淆。

　　回顾条目 28，非具体化（non-reifiable）的类型是其运行时表示比其编译时表示具有更少信息的类型，并且几乎所有泛型和参数化类型都是不可具体化的。 如果某个方法声明其可变参数为非具体化的类型，则编译器将在该声明上生成警告。 如果在推断类型不可确定的可变参数参数上调用该方法，那么编译器也会在调用中生成警告。 警告看起来像这样：

    warning: [unchecked] Possible heap pollution from
        parameterized vararg type List<String>

　　当参数化类型的变量引用不属于该类型的对象时会发生堆污染（Heap pollution）[JLS，4.12.2]。 它会导致编译器的自动生成的强制转换失败，违反了泛型类型系统的基本保证。

　　例如，请考虑以下方法，该方法是第 127 页上的代码片段的一个不太明显的变体：

    // Mixing generics and varargs can violate type safety!
    static void dangerous(List<String>... stringLists) {
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList;             // Heap pollution
        String s = stringLists[0].get(0); // ClassCastException
    }

　　此方法没有可见的强制转换，但在调用一个或多个参数时抛出 ClassCastException 异常。 它的最后一行有一个由编译器生成的隐形转换。 这种转换失败，表明类型安全性已经被破坏，并且将值保存在泛型可变参数数组参数中是不安全的。

　　这个例子引发了一个有趣的问题：为什么声明一个带有泛型可变参数的方法是合法的，当明确创建一个泛型数组是非法的时候呢？ 换句话说，为什么前面显示的方法只生成一个警告，而 127 页上的代码片段会生成一个错误？ 答案是，具有泛型或参数化类型的可变参数参数的方法在实践中可能非常有用，因此语言设计人员选择忍受这种不一致。 事实上，Java 类库导出了几个这样的方法，包括 Arrays.asList(T... a)，Collections.addAll(Collection<? super T> c, T... elements)，EnumSet.of(E first, E... rest)。 与前面显示的危险方法不同，这些类库方法是类型安全的。

　　在 Java 7 中，@SafeVarargs 注解已添加到平台，以允许具有泛型可变参数的方法的作者自动禁止客户端警告。 实质上，@SafeVarargs 注解构成了作者对类型安全的方法的承诺。 为了交换这个承诺，编译器同意不要警告用户调用可能不安全的方法。

　　除非它实际上是安全的，否则注意不要使用 @SafeVarargs 注解标注一个方法。 那么需要做些什么来确保这一点呢？ 回想一下，调用方法时会创建一个泛型数组，以容纳可变参数。 如果方法没有在数组中存储任何东西（它会覆盖参数）并且不允许对数组的引用进行转义（这会使不受信任的代码访问数组），那么它是安全的。 换句话说，如果可变参数数组仅用于从调用者向方法传递可变数量的参数——毕竟这是可变参数的目的——那么该方法是安全的。

　　值得注意的是，你可以违反类型安全性，即使不会在可变参数数组中存储任何内容。 考虑下面的泛型可变参数方法，它返回一个包含参数的数组。 乍一看，它可能看起来像一个方便的小工具：

    // UNSAFE - Exposes a reference to its generic parameter array!
    static <T> T[] toArray(T... args) {
        return args;
    }

　　这个方法只是返回它的可变参数数组。 该方法可能看起来并不危险，但它是该数组的类型由传递给方法的参数的编译时类型决定，编译器可能没有足够的信息来做出正确的判断。 由于此方法返回其可变参数数组，它可以将堆污染传播到调用栈上。
　　为了具体说明，请考虑下面的泛型方法，它接受三个类型 T 的参数，并返回一个包含两个参数的数组，随机选择：

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
          case 0: return toArray(a, b);
          case 1: return toArray(a, c);
          case 2: return toArray(b, c);
        }
        throw new AssertionError(); // Can't get here
    }

　　这个方法本身不是危险的，除了调用具有泛型可变参数的 toArray 方法之外，不会产生警告。

　　编译此方法时，编译器会生成代码以创建一个将两个 T 实例传递给 toArray 的可变参数数组。 这段代码分配了一个 Object[] 类型的数组，它是保证保存这些实例的最具体的类型，而不管在调用位置传递给 pickTwo 的对象是什么类型。 toArray 方法只是简单地将这个数组返回给 pickTwo，然后 pickTwo 将它返回给调用者，所以 pickTwo 总是返回一个 Object[] 类型的数组。

    public static void main(String[] args) {
        String[] attributes = pickTwo("Good", "Fast", "Cheap");
    }

　　这种方法没有任何问题，因此它编译时不会产生任何警告。 但是当运行它时，抛出一个 ClassCastException 异常，尽管不包含可见的转换。 你没有看到的是，编译器已经生成了一个隐藏的强制转换为由 pickTwo 返回的值的 String[] 类型，以便它可以存储在属性中。 转换失败，因为 Object[] 不是 String[] 的子类型。 这种故障相当令人不安，因为它从实际导致堆污染（toArray）的方法中移除了两个级别，并且在实际参数存储在其中之后，可变参数数组未被修改。

　　这个例子是为了让人们认识到给另一个方法访问一个泛型的可变参数数组是不安全的，除了两个例外：将数组传递给另一个可变参数方法是安全的，这个方法是用 @SafeVarargs 正确标注的， 将数组传递给一个非可变参数的方法是安全的，该方法仅计算数组内容的一些方法。

　　这里是安全使用泛型可变参数的典型示例。 此方法将任意数量的列表作为参数，并按顺序返回包含所有输入列表元素的单个列表。 由于该方法使用 @SafeVarargs 进行标注，因此在声明或其调用站位置上不会生成任何警告：

    // Safe method with a generic varargs parameter
    @SafeVarargs
    static <T> List<T> flatten(List<? extends T>... lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }

　　决定何时使用 @SafeVarargs 注解的规则很简单：在每种方法上使用 @SafeVarargs，并使用泛型或参数化类型的可变参数，这样用户就不会因不必要的和令人困惑的编译器警告而担忧。 这意味着你不应该写危险或者 toArray 等不安全的可变参数方法。 每次编译器警告你可能会受到来自你控制的方法中泛型可变参数的堆污染时，请检查该方法是否安全。 提醒一下，在下列情况下，泛型可变参数方法是安全的：

## 它不会在可变参数数组中存储任何东西
## 它不会使数组（或克隆）对不可信代码可见。
如果违反这些禁令中的任何一项，请修复。
　　请注意，SafeVarargs 注解只对不能被重写的方法是合法的，因为不可能保证每个可能的重写方法都是安全的。 在 Java 8 中，注解仅在静态方法和 final 实例方法上合法; 在 Java 9 中，它在私有实例方法中也变为合法。

　　使用 SafeVarargs 注解的替代方法是采用条目 28 的建议，并用 List 参数替换可变参数（这是一个变相的数组）。 下面是应用于我们的 flatten 方法时，这种方法的样子。 请注意，只有参数声明被更改了：

    // List as a typesafe alternative to a generic varargs parameter
    static <T> List<T> flatten(List<List<? extends T>> lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }
　　然后可以将此方法与静态工厂方法 List.of 结合使用，以允许可变数量的参数。 请注意，这种方法依赖于 List.of 声明使用 @SafeVarargs 注解：

    audience = flatten(List.of(friends, romans, countrymen));

　　这种方法的优点是编译器可以证明这种方法是类型安全的。 不必使用 @SafeVarargs 注解来证明其安全性，也不用担心在确定安全性时可能会犯错。 主要缺点是客户端代码有点冗长，运行可能会慢一些。

　　这个技巧也可以用在不可能写一个安全的可变参数方法的情况下，就像第 147 页的 toArray 方法那样。它的列表模拟是 List.of 方法，所以我们甚至不必编写它；Java 类库作者已经为我们完成了这项工作。 pickTwo 方法然后变成这样：

　　main 方变成这样：

    public static void main(String[] args) {
        List<String> attributes = pickTwo("Good", "Fast", "Cheap");
    }

　　生成的代码是类型安全的，因为它只使用泛型，不是数组。

　　总而言之，可变参数和泛型不能很好地交互，因为可变参数机制是在数组上面构建的脆弱的抽象，并且数组具有与泛型不同的类型规则。 虽然泛型可变参数不是类型安全的，但它们是合法的。 如果选择使用泛型（或参数化）可变参数编写方法，请首先确保该方法是类型安全的，然后使用 @SafeVarargs 注解对其进行标注，以免造成使用不愉快。
# 参考资料
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/31.%20%E4%BD%BF%E7%94%A8%E9%99%90%E5%AE%9A%E9%80%9A%E9%85%8D%E7%AC%A6%E6%9D%A5%E5%A2%9E%E5%8A%A0API%E7%9A%84%E7%81%B5%E6%B4%BB%E6%80%A7
