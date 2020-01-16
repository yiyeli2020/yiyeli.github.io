title: Effctive-Java阅读笔记XV
date: 2020-1-17 00:33:12
categories: 2020年1月
tags: [Java]

---

第五章阅读： 泛型

29.优先考虑泛型
泛型类型比需要在客户端代码中强制转换的类型更安全，更易于使用。
30.优先使用泛型方法
像泛型类型一样，泛型方法比需要客户端对输入参数和返回值进行显式强制转换的方法更安全，更易于使用。

<!-- more -->

# 优先考虑泛型

　　参数化声明并使用 JDK 提供的泛型类型和方法通常不会太困难。 但编写自己的泛型类型有点困难，但值得努力学习。

　　考虑条目 7 中的简单堆栈实现：

    // Object-based collection - a prime candidate for generics
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
            Object result = elements[--size];
            elements[size] = null; // Eliminate obsolete reference
            return result;
        }

        public boolean isEmpty() {
            return size == 0;
        }

        private void ensureCapacity() {
            if (elements.length == size)
                elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

　　这个类应该已经被参数化了，但是由于事实并非如此，我们可以对它进行泛型化。 换句话说，我们可以参数化它，而不会损害原始非参数化版本的客户端。 就目前而言，客户端必须强制转换从堆栈中弹出的对象，而这些强制转换可能会在运行时失败。 泛型化类的第一步是在其声明中添加一个或多个类型参数。 在这种情况下，有一个类型参数，表示堆栈的元素类型，这个类型参数的常规名称是 E（详见第 68 条）。

　　下一步是用相应的类型参数替换所有使用的 Object 类型，然后尝试编译生成的程序：

    // Initial attempt to generify Stack - won't compile!
    public class Stack<E> {
        private E[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;

        public Stack() {
            elements = new E[DEFAULT_INITIAL_CAPACITY];
        }

        public void push(E e) {
            ensureCapacity();
            elements[size++] = e;
        }

        public E pop() {
            if (size == 0)
                throw new EmptyStackException();
            E result = elements[--size];
            elements[size] = null; // Eliminate obsolete reference
            return result;
        }
        ... // no changes in isEmpty or ensureCapacity
    }

　　你通常会得到至少一个错误或警告，这个类也不例外。 幸运的是，这个类只产生一个错误：

    Stack.java:8: generic array creation
            elements = new E[DEFAULT_INITIAL_CAPACITY];
                       ^
　　如条目 28 所述，你不能创建一个不可具体化类型的数组，例如类型 E。每当编写一个由数组支持的泛型时，就会出现此问题。 有两种合理的方法来解决它。 第一种解决方案直接规避了对泛型数组创建的禁用：创建一个 Object 数组并将其转换为泛型数组类型。 现在没有了错误，编译器会发出警告。 这种用法是合法的，但不是（一般）类型安全的：

    Stack.java:8: warning: [unchecked] unchecked cast
    found: Object[], required: E[]
            elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
                           ^
　　编译器可能无法证明你的程序是类型安全的，但你可以。 你必须说服自己，不加限制的类型强制转换不会损害程序的类型安全。 有问题的数组（元素）保存在一个私有属性中，永远不会返回给客户端或传递给任何其他方法。 保存在数组中的唯一元素是那些传递给 push 方法的元素，它们是 E 类型的，所以未经检查的强制转换不会造成任何伤害。

　　一旦证明未经检查的强制转换是安全的，请尽可能缩小范围（条目 27）。 在这种情况下，构造方法只包含未经检查的数组创建，所以在整个构造方法中抑制警告是合适的。 通过添加一个注解来执行此操作，Stack 可以干净地编译，并且可以在没有显式强制转换或担心 ClassCastException 异常的情况下使用它：

    // The elements array will contain only E instances from push(E).
    // This is sufficient to ensure type safety, but the runtime
    // type of the array won't be E[]; it will always be Object[]!
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

　消除 Stack 中的泛型数组创建错误的第二种方法是将属性元素的类型从 E[] 更改为 Object[]。 如果这样做，会得到一个不同的错误：

    Stack.java:19: incompatible types
    found: Object, required: E
            E result = elements[--size];
                               ^
　　可以通过将从数组中检索到的元素转换为 E 来将此错误更改为警告：

    Stack.java:19: warning: [unchecked] unchecked cast
    found: Object, required: E
            E result = (E) elements[--size];
                                   ^
　　因为 E 是不可具体化的类型，编译器无法在运行时检查强制转换。 再一次，你可以很容易地向自己证明，不加限制的转换是安全的，所以可以适当地抑制警告。 根据条目 27 的建议，我们只在包含未经检查的强制转换的分配上抑制警告，而不是在整个 pop 方法上：

    // Appropriate suppression of unchecked warning
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();

        // push requires elements to be of type E, so cast is correct
        @SuppressWarnings("unchecked") E result =
            (E) elements[--size];

        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
　　两种消除泛型数组创建的技术都有其追随者。 第一个更可读：数组被声明为 E[] 类型，清楚地表明它只包含 E 实例。 它也更简洁：在一个典型的泛型类中，你从代码中的许多点读取数组; 第一种技术只需要一次转换（创建数组的地方），而第二种技术每次读取数组元素都需要单独转换。 因此，第一种技术是优选的并且在实践中更常用。 但是，它确实会造成堆污染（heap pollution）（详见第 32 条）：数组的运行时类型与编译时类型不匹配（除非 E 碰巧是 Object）。 这使得一些程序员非常不安，他们选择了第二种技术，尽管在这种情况下堆的污染是无害的。
　　下面的程序演示了泛型 Stack 类的使用。 该程序以相反的顺序打印其命令行参数，并将其转换为大写。 对从堆栈弹出的元素调用 String 的 toUpperCase 方法不需要显式强制转换，而自动生成的强制转换将保证成功：

    // Little program to exercise our generic Stack
    public static void main(String[] args) {
        Stack<String> stack = new Stack<>();
        for (String arg : args)
            stack.push(arg);
        while (!stack.isEmpty())
            System.out.println(stack.pop().toUpperCase());
    }

　　上面的例子似乎与条目 28 相矛盾，条目 28 中鼓励使用列表优先于数组。 在泛型类型中使用列表并不总是可行或可取的。 Java 本身生来并不支持列表，所以一些泛型类型（如 ArrayList）必须在数组上实现。 其他的泛型类型，比如 HashMap，是为了提高性能而实现的。

　　绝大多数泛型类型就像我们的 Stack 示例一样，它们的类型参数没有限制：可以创建一个 Stack<Object>，Stack<int[]>，Stack<List<String>> 或者其他任何对象的 Stack 引用类型。 请注意，不能创建基本类型的堆栈：尝试创建 Stack<int> 或 Stack<double> 将导致编译时错误。 这是 Java 泛型类型系统的一个基本限制。 可以使用基本类型的包装类（详见第 61 条）来解决这个限制。

　　有一些泛型类型限制了它们类型参数的允许值。 例如，考虑 java.util.concurrent.DelayQueue，它的声明如下所示：

    class DelayQueue<E extends Delayed> implements BlockingQueue<E>

　　类型参数列表（<E extends Delayed>）要求实际的类型参数 E 是 java.util.concurrent.Delayed 的子类型。 这使得 DelayQueue 实现及其客户端可以利用 DelayQueue 元素上的 Delayed 方法，而不需要显式的转换或 ClassCastException 异常的风险。 类型参数 E 被称为限定类型参数。 请注意，子类型关系被定义为每个类型都是自己的子类型，因此创建 DelayQueue<Delayed> 是合法的。

　　总之，泛型类型比需要在客户端代码中强制转换的类型更安全，更易于使用。 当你设计新的类型时，确保它们可以在没有这种强制转换的情况下使用。 这通常意味着使类型泛型化。 如果你有任何现有的类型，应该是泛型的但实际上却不是，那么把它们泛型化。 这使这些类型的新用户的使用更容易，而不会破坏现有的客户端（条目 26）。

# 优先使用泛型方法

　　正如类可以是泛型的，方法也可以是泛型的。 对参数化类型进行操作的静态工具方法通常都是泛型的。 集合中的所有“算法”方法（如 binarySearch 和 sort）都是泛型的。
　　编写泛型方法类似于编写泛型类型。 考虑这个方法，它返回两个集合的并集：

    // Uses raw types - unacceptable! [Item 26]
    public static Set union(Set s1, Set s2) {
        Set result = new HashSet(s1);
        result.addAll(s2);
        return result;
    }

　　此方法可以编译但有两个警告：

    Union.java:5: warning: [unchecked] unchecked call to
    HashSet(Collection<? extends E>) as a member of raw type HashSet
            Set result = new HashSet(s1);
                         ^
    Union.java:6: warning: [unchecked] unchecked call to
    addAll(Collection<? extends E>) as a member of raw type Set
            result.addAll(s2);
                         ^
　　要修复这些警告并使方法类型安全，请修改其声明以声明表示三个集合（两个参数和返回值）的元素类型的类型参数，并在整个方法中使用此类型参数。 声明类型参数的类型参数列表位于方法的修饰符和返回类型之间。 在这个例子中，类型参数列表是 <E>，返回类型是 Set<E>。 类型参数的命名约定对于泛型方法和泛型类型是相同的（详见第 29 和 68 条）：

    // Generic method
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;

    }

　　至少对于简单的泛型方法来说，就是这样。 此方法编译时不会生成任何警告，并提供类型安全性和易用性。 这是一个简单的程序来运行该方法。 这个程序不包含强制转换和编译时没有错误或警告：至少对于简单的泛型方法来说，就是这样。 此方法编译时不会生成任何警告，并提供类型安全性和易用性。 这是一个简单的程序来运行该方法。 这个程序不包含强制转换和编译时没有错误或警告：

    // Simple program to exercise generic method
    public static void main(String[] args) {
        Set<String> guys = Set.of("Tom", "Dick", "Harry");
        Set<String> stooges = Set.of("Larry", "Moe", "Curly");
        Set<String> aflCio = union(guys, stooges);
        System.out.println(aflCio);
    }

　　当运行这个程序时，它会打印[Moe, Tom, Harry, Larry, Curly, Dick]（输出中元素的顺序依赖于具体实现。）

　　union 方法的一个限制是所有三个集合（输入参数和返回值）的类型必须完全相同。 通过使用限定通配符类型（ bounded wildcard types）（详见第 31 条），可以使该方法更加灵活。

　　有时，需要创建一个不可改变但适用于许多不同类型的对象。 因为泛型是通过擦除来实现的（详见第 28 条），所以可以使用单个对象进行所有必需的类型参数化，但是需要编写一个静态工厂方法来重复地为每个请求的类型参数化分配对象。 这种称为泛型单例工厂（generic singleton factory）的模式用于方法对象（function objects）（详见第 42 条），比如 Collections.reverseOrder 方法，偶尔也用于 Collections.emptySet 之类的集合。

　　假设你想写一个恒等方法分配器（ identity function dispenser）。 类库提供了 Function.identity 方法，所以没有理由编写你自己的实现（详见第 59 条），但它是有启发性的。 如果每次要求的时候都去创建一个新的恒等方法对象是浪费的，因为它是无状态的。 如果 Java 的泛型被具体化，那么每个类型都需要一个恒等方法，但是由于它们被擦除以后，所以泛型的单例就足够了。 以下是它的实例：

    // Generic singleton factory pattern
    private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }

　　将 IDENTITY_FN 转换为 (UnaryFunction<T>) 会生成一个未经检查的强制转换警告，因为 UnaryOperator<Object> 对于每个 T 都不是一个 UnaryOperator<T>。但是恒等方法是特殊的：它返回未修改的参数，所以我们知道，使用它作为一个 UnaryFunction<T> 是类型安全的，无论 T 的值是多少。因此，我们可以放心地抑制由这个强制生成的未经检查的强制转换警告。 一旦我们完成了这些，代码编译没有错误或警告。
　　下面是一个示例程序，它使用我们的泛型单例作为 UnaryOperator<String> 和 UnaryOperator<Number>。 像往常一样，它不包含强制转化，编译时也没有错误和警告：

    // Sample program to exercise generic singleton
    public static void main(String[] args) {
        String[] strings = { "jute", "hemp", "nylon" };
        UnaryOperator<String> sameString = identityFunction();

        for (String s : strings)
            System.out.println(sameString.apply(s));

        Number[] numbers = { 1, 2.0, 3L };

        UnaryOperator<Number> sameNumber = identityFunction();

        for (Number n : numbers)
            System.out.println(sameNumber.apply(n));
    }

　　虽然相对较少，类型参数受涉及该类型参数本身的某种表达式限制是允许的。 这就是所谓的 ***递归类型限制（recursive type bound）*** 。 递归类型限制的常见用法与 Comparable 接口有关，它定义了一个类型的自然顺序（详见第 14 条）。 这个接口如下所示：

    public interface Comparable<T> {
        int compareTo(T o);
    }

　　类型参数 T 定义了实现 Comparable<T> 的类型的元素可以比较的类型。 在实际中，几乎所有类型都只能与自己类型的元素进行比较。 所以，例如，String 类实现了 Comparable<String>，Integer 类实现了 Comparable<Integer> 等等。
　　许多方法采用实现 Comparable 的元素的集合来对其进行排序，在其中进行搜索，计算其最小值或最大值等。 要做到这一点，要求集合中的每一个元素都可以与其中的每一个元素相比，换言之，这个元素是可以相互比较的。 以下是如何表达这一约束：

    // Using a recursive type bound to express mutual comparability
    public static <E extends Comparable<E>> E max(Collection<E> c);

　　限定的类型 <E extends Comparable <E >> 可以理解为「任何可以与自己比较的类型 E」，这或多或少精确地对应于相互可比性的概念。
　　这里有一个与前面的声明相匹配的方法。它根据其元素的自然顺序来计算集合中的最大值，并编译没有错误或警告：

    // Returns max value in a collection - uses recursive type bound
    public static <E extends Comparable<E>> E max(Collection<E> c) {
      if (c.isEmpty())
        throw new IllegalArgumentException("Empty collection");
      E result = null;
      for (E e : c)
        if (result == null || e.compareTo(result) > 0)
          result = Objects.requireNonNull(e);
      return result;
    }

　　请注意，如果列表为空，则此方法将引发 IllegalArgumentException 异常。 更好的选择是返回一个 Optional<E>（详见第 55 条）。

　　递归类型限制可能变得复杂得多，但幸运的是他们很少这样做。 如果你理解了这个习惯用法，它的通配符变体（详见第 31 条）和模拟的自我类型用法（详见第 2 条），你将能够处理在实践中遇到的大多数递归类型限制。

　　总之，像泛型类型一样，泛型方法比需要客户端对输入参数和返回值进行显式强制转换的方法更安全，更易于使用。 像类型一样，你应该确保你的方法可以不用强制转换，这通常意味着它们是泛型的。 应该泛型化现有的方法，其使用需要强制转换。 这使得新用户的使用更容易，而不会破坏现有的客户端（详见第 26 条）。


# 参考资料
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/29.%20%E4%BC%98%E5%85%88%E8%80%83%E8%99%91%E6%B3%9B%E5%9E%8B
