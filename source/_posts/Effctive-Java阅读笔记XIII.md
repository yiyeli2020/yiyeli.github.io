title: Effctive-Java阅读笔记XIII
date: 2020-1-16 23:14:12
categories: 2020年1月
tags: [Java]

---

第四章阅读：类和接口

25.将源文件限制为单个顶级类
永远不要将多个顶级类或接口放在一个源文件中。 遵循这个规则保证在编译时不能有多个定义。 这又保证了编译生成的类文件以及生成的程序的行为与源文件传递给编译器的顺序无关。

第五章阅读： 泛型
泛型实现了参数化类型，这样编写的组件（通常是集合）可以适用于多种类型。“泛型”这个术语的含义是“适用于很多类型”。编程语言中泛型出现的初衷是通过解耦类或方法与所使用的类型之间的约束，使得类或方法具备最宽泛的表达力。

26.不要使用原始类型
使用原始类型可能导致运行时异常，所以不要使用它们。 原始类型只是为了与引入泛型机制之前的遗留代码进行兼容和互用而提供的。Set<Object> 是一个参数化类型，表示一个可以包含任何类型对象的集合，Set<?> 是一个通配符类型，表示一个只能包含某些未知类型对象的集合，Set 是一个原始类型，它不在泛型类型系统之列。 前两个类型是安全的，最后一个不是。

<!-- more -->

# 将源文件限制为单个顶级类

　　虽然 Java 编译器允许在单个源文件中定义多个顶级类，但这样做没有任何好处，并且存在重大风险。 风险源于在源文件中定义多个顶级类使得为类提供多个定义成为可能。 使用哪个定义会受到源文件传递给编译器的顺序的影响。
　　为了具体说明，请考虑下面源文件，其中只包含一个引用其他两个顶级类（Utensil 和 Dessert 类）的成员的 Main 类：


    public class Main {
        public static void main(String[] args) {
            System.out.println(Utensil.NAME + [Dessert.NAME](http://Dessert.NAME));
        }
    }
　　现在假设在 Utensil.java 的源文件中同时定义了 Utensil 和 Dessert：

    // Two classes defined in one file. Don't ever do this!
    class Utensil {
        static final String NAME = "pan";
    }

    class Dessert {
        static final String NAME = "cake";
    }

    当然，main 方法会打印 pancake。

　　现在假设你不小心创建了另一个名为 Dessert.java 的源文件，它定义了相同的两个类：

    // Two classes defined in one file. Don't ever do this!
    class Utensil {
        static final String NAME = "pot";
    }

    class Dessert {
        static final String NAME = "pie";
    }

　　如果你足够幸运，使用命令 javac Main.java Dessert.java 编译程序，编译将失败，编译器会告诉你，你已经多次定义了类 Utensil 和 Dessert。 这是因为编译器首先编译 Main.java，当它看到对 Utensil 的引用（它在 Dessert 的引用之前）时，它将在 Utensil.java 中查找这个类并找到 Utensil 和 Dessert。 当编译器在命令行上遇到 Dessert.java 时，它也将拉入该文件，导致它遇到 Utensil 和 Dessert 的定义。

　　如果使用命令 javac Main.java 或 javac Main.java Utensil.java 编译程序，它的行为与在编写 Dessert.java 文件（即打印 pancake）之前的行为相同。 但是，如果使用命令 javac Dessert.java Main.java 编译程序，它将打印 potpie。 程序的行为因此受到源文件传递给编译器的顺序的影响，这显然是不可接受的。

　　解决这个问题很简单，将顶层类（如我们的例子中的 Utensil 和 Dessert）分割成单独的源文件。 如果试图将多个顶级类放入单个源文件中，请考虑使用静态成员类（详见第 24 条）作为将类拆分为单独的源文件的替代方法。 如果这些类从属于另一个类，那么将它们变成静态成员类通常是更好的选择，因为它提高了可读性，并且可以通过声明它们为私有（详见第 15 条）来减少类的可访问性。下面是我们的例子看起来如何使用静态成员类：

    // Static member classes instead of multiple top-level classes
    public class Test {
        public static void main(String[] args) {
            System.out.println(Utensil.NAME + [Dessert.NAME](http://Dessert.NAME));
        }

        private static class Utensil {
            static final String NAME = "pan";
        }

        private static class Dessert {
            static final String NAME = "cake";
        }
    }

　　这个教训很清楚：永远不要将多个顶级类或接口放在一个源文件中。 遵循这个规则保证在编译时不能有多个定义。 这又保证了编译生成的类文件以及生成的程序的行为与源文件传递给编译器的顺序无关。
# 不要使用原始类型

　　首先，有几个术语。一个类或接口，它的声明有一个或多个类型参数（type parameters ），被称之为泛型类或泛型接口。 例如，List 接口具有单个类型参数 E，表示其元素类型。 接口的全名是 List<E>（读作「E」的列表），但是人们经常称它为 List。 泛型类和接口统称为泛型类型（generic types）。

　　每个泛型定义了一组参数化类型（parameterized types），它们由类或接口名称组成，后跟一个与泛型类型的形式类型参数[JLS，4.4,4.5] 相对应的实际类型参数的尖括号「<>」列表。 例如，List<String>（读作「字符串列表」）是一个参数化类型，表示其元素类型为 String 的列表。 （String 是与形式类型参数 E 相对应的实际类型参数）。

　　最后，每个泛型定义了一个原始类型（raw type），它是没有任何类型参数的泛型类型的名称[JLS，4.8]。 例如，对应于 List<E> 的原始类型是 List。 原始类型的行为就像所有的泛型类型信息都从类型声明中被清除一样。 它们的存在主要是为了与没有泛型之前的代码相兼容。

　　在泛型被添加到 Java 之前，这是一个典型的集合声明。 从 Java 9 开始，它仍然是合法的，但并不是典型的声明方式了：

    // Raw collection type - don't do this!

    // My stamp collection. Contains only Stamp instances.
    private final Collection stamps = ... ;

　　如果你今天使用这个声明，然后不小心把 coin 实例放入你的 stamp 集合中，错误的插入编译和运行没有错误（尽管编译器发出一个模糊的警告）：

    // Erroneous insertion of coin into stamp collection
    stamps.add(new Coin( ... )); // Emits "unchecked call" warning

　　直到您尝试从 stamp 集合中检索 coin 实例时才会发生错误：

    // Raw iterator type - don't do this!
    for (Iterator i = stamps.iterator(); i.hasNext(); )
        Stamp stamp = (Stamp) i.next(); // Throws ClassCastException
            stamp.cancel();

　　正如本书所提到的，在编译完成之后尽快发现错误是值得的，理想情况是在编译时。 在这种情况下，直到运行时才发现错误，在错误发生后的很长一段时间，以及可能远离包含错误的代码的代码中。 一旦看到 ClassCastException，就必须搜索代码类库，查找将 coin 实例放入 stamp 集合的方法调用。 编译器不能帮助你，因为它不能理解那个说「仅包含 stamp 实例」的注释。
　　对于泛型，类型声明包含的信息，而不是注释：

    // Parameterized collection type - typesafe
    private final Collection<Stamp> stamps = ... ;

　　从这个声明中，编译器知道 stamps 集合应该只包含 Stamp 实例，并保证它是 true，假设你的整个代码类库编译时不发出（或者抑制；参见条目 27）任何警告。 当使用参数化类型声明声明 stamps 时，错误的插入会生成一个编译时错误消息，告诉你到底发生了什么错误：

    Test.java:9: error: incompatible types: Coin cannot be converted
    to Stamp
        c.add(new Coin());
                  ^
　　当从集合中检索元素时，编译器会为你插入不可见的强制转换，并保证它们不会失败（再假设你的所有代码都不会生成或禁止任何编译器警告）。 虽然意外地将 coin 实例插入 stamp 集合的预期可能看起来很牵强，但这个问题是真实的。 例如，很容易想象将 BigInteger 放入一个只包含 BigDecimal 实例的集合中。

　　如前所述，使用原始类型（没有类型参数的泛型）是合法的，但是你不应该这样做。
***如果你使用原始类型，则会丧失泛型的所有安全性和表达上的优势。*** 鉴于你不应该使用它们，为什么语言设计者首先允许原始类型呢？ 答案是为了兼容性。 泛型被添加时，Java 即将进入第二个十年，并且有大量的代码没有使用泛型。 所有这些代码都是合法的，并且与使用泛型的新代码进行交互操作被认为是至关重要的。 将参数化类型的实例传递给为原始类型设计的方法必须是合法的，反之亦然。 这个需求，被称为迁移兼容性，驱使决策支持原始类型，并使用擦除来实现泛型（详见第 28 条）。

　　虽然不应使用诸如 List 之类的原始类型，但可以使用参数化类型来允许插入任意对象（如 List<Object>）。 原始类型 List 和参数化类型 List<Object> 之间有什么区别？ 松散地说，前者已经选择了泛型类型系统，而后者明确地告诉编译器，它能够保存任何类型的对象。 虽然可以将 List<String> 传递给 List 类型的参数，但不能将其传递给 List<Object> 类型的参数。 泛型有子类型的规则，List<String> 是原始类型 List 的子类型，但不是参数化类型 List<Object> 的子类型（条目 28）。 因此，如果使用诸如 List 之类的原始类型，则会丢失类型安全性，但是如果使用参数化类型（例如 List<Object>）则不会。

　　为了具体说明，请考虑以下程序：

    // Fails at runtime - unsafeAdd method uses a raw type (List)!
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0); // Has compiler-generated cast
    }

    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }

　　此程序可以编译，它使用原始类型列表，但会收到警告：

    Test.java:10: warning: [unchecked] unchecked call to add(E) as a
    member of the raw type List
        list.add(o);
                ^

　　实际上，如果运行该程序，则当程序尝试调用 strings.get(0) 的结果（一个 Integer）转换为一个 String 时，会得到 ClassCastException 异常。 这是一个编译器生成的强制转换，因此通常会保证成功，但在这种情况下，我们忽略了编译器警告并付出了代价。

　　如果用 unsafeAdd 声明中的参数化类型 List<Object> 替换原始类型 List，并尝试重新编译该程序，则会发现它不再编译，而是发出错误消息：

    Test.java:5: error: incompatible types: List<String> cannot be
    converted to List<Object>
        unsafeAdd(strings, Integer.valueOf(42));

　　你可能会试图使用原始类型来处理元素类型未知且无关紧要的集合。 例如，假设你想编写一个方法，它需要两个集合并返回它们共同拥有的元素的数量。 如果是泛型新手，那么您可以这样写：

    // Use of raw type for unknown element type - don't do this!
    static int numElementsInCommon(Set s1, Set s2) {
        int result = 0;
        for (Object o1 : s1)
            if (s2.contains(o1))
                result++;
        return result;
    }

　　这种方法可以工作，但它使用原始类型，这是危险的。 安全替代方式是使用无限制通配符类型（unbounded wildcard types）。 如果要使用泛型类型，但不知道或关心实际类型参数是什么，则可以使用问号来代替。 例如，泛型类型 Set<E> 的无限制通配符类型是 Set<?>（读取「某种类型的集合」）。 它是最通用的参数化的 Set 类型，能够保持任何集合。 下面是 numElementsInCommon 方法使用无限制通配符类型声明的情况：

    // Uses unbounded wildcard type - typesafe and flexible
    static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }

　　无限制通配符 Set<?> 与原始类型 Set 之间有什么区别？ 问号真的给你放任何东西吗？ 这不是要点，但通配符类型是安全的，原始类型不是。 你可以将任何元素放入具有原始类型的集合中，轻易破坏集合的类型不变性（如第 119 页上的 unsafeAdd 方法所示）; 你不能把任何元素（除 null 之外）放入一个 Collection<?> 中。 试图这样做会产生一个像这样的编译时错误消息：

    WildCard.java:13: error: incompatible types: String cannot be
    converted to CAP#1
        c.add("verboten");
              ^
      where CAP#1 is a fresh type-variable:
        CAP#1 extends Object from capture of ?

　　不可否认的是，这个错误信息留下了一些需要的东西，但是编译器已经完成了它的工作，不管它的元素类型是什么，都不会破坏集合的类型不变性。 你不仅不能将任何元素（除 null 以外）放入一个 Collection<?> 中，并且根本无法猜测你会得到那种类型的对象。 如果这些限制是不可接受的，可以使用泛型方法（详见第 30 条）或有限制的通配符类型（详见第 31 条）。

　　对于不应该使用原始类型的规则，有一些小例外。 ***你必须在类字面值（class literals）中使用原始类型。***  规范中不允许使用参数化类型（尽管它允许数组类型和基本类型）[JLS，15.8.2]。 换句话说，List.class，String[].class 和 int.class 都是合法的，但 List<String>.class 和 List<?>.class 都是不合法的。

　　规则的第二个例外与 instanceof 操作符有关。 因为泛型类型信息在运行时被擦除，所以在无限制通配符类型以外的参数化类型上使用 instanceof 运算符是非法的。 使用无限制通配符类型代替原始类型，不会对 instanceof 运算符的行为产生任何影响。 在这种情况下，尖括号（<>）和问号（?）就显得多余。 ***以下是使用泛型类型的 instanceof 运算符的首选方法***：

    // Legitimate use of raw type - instanceof operator
    if (o instanceof Set) {       // Raw type
        Set<?> s = (Set<?>) o;    // Wildcard type
        ...
    }

　　请注意，一旦确定 o 对象是一个 Set，则必须将其转换为通配符 Set<?>，而不是原始类型 Set。 这是一个受检查的（checked）转换，所以不会导致编译器警告。

　　总之，使用原始类型可能导致运行时异常，所以不要使用它们。 原始类型只是为了与引入泛型机制之前的遗留代码进行兼容和互用而提供的。 作为一个快速回顾，Set<Object> 是一个参数化类型，表示一个可以包含任何类型对象的集合，Set<?> 是一个通配符类型，表示一个只能包含某些未知类型对象的集合，Set 是一个原始类型，它不在泛型类型系统之列。 前两个类型是安全的，最后一个不是。

　　为了快速参考，下表中总结了本条目（以及本章稍后介绍的一些）中介绍的术语：

    术语	       中文含义	   举例	             所在条目
    Parameterized type	参数化类型	List<String>	条目 26
    Actual type parameter	实际类型参数	String	条目 26
    Generic type	泛型类型	List<E>	条目 26 和 条目 29
    Formal type parameter	形式类型参数	E	条目 26
    Unbounded wildcard type	无限制通配符类型	List<?>	条目 26
    Raw type	原始类型	List	条目 26
    Bounded type parameter	限制类型参数	<E extends Number>	条目 29
    Recursive type bound	递归类型限制	<T extends Comparable<T>>	条目 30
    Bounded wildcard type	限制通配符类型	List<? extends Number>	条目 31
    Generic method	泛型方法	static <E> List<E> asList(E[] a)	条目 30
    Type token	类型令牌	String.class	条目 33

# 参考资料
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/25.%20%E5%B0%86%E6%BA%90%E6%96%87%E4%BB%B6%E9%99%90%E5%88%B6%E4%B8%BA%E5%8D%95%E4%B8%AA%E9%A1%B6%E7%BA%A7%E7%B1%BB?id=_25-%e5%b0%86%e6%ba%90%e6%96%87%e4%bb%b6%e9%99%90%e5%88%b6%e4%b8%ba%e5%8d%95%e4%b8%aa%e9%a1%b6%e7%ba%a7%e7%b1%bb
【2】https://lingcoder.github.io/OnJava8/#/book/20-Generics?id=%e7%ae%80%e5%8d%95%e6%b3%9b%e5%9e%8b
