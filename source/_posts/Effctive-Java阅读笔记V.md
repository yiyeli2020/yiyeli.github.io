title: Effctive-Java阅读笔记V
date: 2020-1-6 09:46:12
categories: 2020年1月
tags: [Java]

---

第三章阅读：创建和销毁对象
09.使用 try-with-resources 语句替代 try-finally 语句
10.重写 equals 方法时遵守通用约定

<!-- more -->
# 使用 try-with-resources 语句替代 try-finally 语句

从以往来看，try-finally 语句是保证资源正确关闭的最佳方式，即使是在程序抛出异常或返回的情况下：

    // try-finally - No longer the best way to close resources!
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
这可能看起来并不坏，但是当添加第二个资源时，情况会变得更糟：

    // try-finally is ugly when used with more than one resource!
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
即使是用 try-finally 语句关闭资源的正确代码，如前面两个代码示例所示，也有一个微妙的缺陷。 try-with-resources 块和 finally 块中的代码都可以抛出异常。 例如，在前面的 firstLineOfFile 方法中，由于底层物理设备发生故障，对 readLine 方法的调用可能会引发异常，并且由于相同的原因，调用 close 方法可能会失败。 在这种情况下，第二个异常完全冲掉了第一个异常。 在异常堆栈跟踪中没有第一个异常的记录，这可能使实际系统中的调试非常复杂——通常这是你想要诊断问题的第一个异常。 虽然可以编写代码来抑制第二个异常，但是实际上没有人这样做，因为它太冗长了。

　　当 Java 7 引入了 try-with-resources 语句时，所有这些问题一下子都得到了解决。要使用这个构造，资源必须实现 AutoCloseable 接口，该接口由一个返回为 void 的 close 组成。Java 类库和第三方类库中的许多类和接口现在都实现或继承了 AutoCloseable 接口。如果你编写的类表示必须关闭的资源，那么这个类也应该实现 AutoCloseable 接口。

　　以下是我们的第一个使用 try-with-resources 的示例：

    // try-with-resources - the the best way to close resources!
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
           return br.readLine();
        }
    }
　以下是我们的第二个使用 try-with-resources 的示例：

    // try-with-resources on multiple resources - short and sweet
    static void copy(String src, String dst) throws IOException {
        try (InputStream   in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }

不仅 try-with-resources 版本比原始版本更精简，更好的可读性，而且它们提供了更好的诊断。 考虑 firstLineOfFile 方法。 如果调用 readLine 和（不可见）close 方法都抛出异常，则后一个异常将被抑制（suppressed），而不是前者。 事实上，为了保留你真正想看到的异常，可能会抑制多个异常。 这些抑制的异常没有被抛弃， 而是打印在堆栈跟踪中，并标注为被抑制了。 你也可以使用 getSuppressed 方法以编程方式访问它们，该方法在 Java 7 中已添加到的 Throwable 中。

　　可以在 try-with-resources 语句中添加 catch 子句，就像在常规的 try-finally 语句中一样。这允许你处理异常，而不会在另一层嵌套中污染代码。作为一个稍微有些做作的例子，这里有一个版本的 firstLineOfFile 方法，它不会抛出异常，但是如果它不能打开或读取文件，则返回默认值：

    // try-with-resources with a catch clause
    static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(
               new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }

结论很明确：在处理必须关闭的资源时，使用 try-with-resources 语句替代 try-finally 语句。 生成的代码更简洁，更清晰，并且生成的异常更有用。 try-with-resources 语句在编写必须关闭资源的代码时会更容易，也不会出错，而使用 try-finally 语句实际上是不可能的。

# 重写 equals 方法时遵守通用约定

　虽然 Object 是一个具体的类，但它主要是为继承而设计的。它的所有非 final 方法（equals、hashCode、toString、clone 和 finalize）都有清晰的通用约定（ general contracts），因为它们被设计为被子类重写。任何类要重写这些方法时，都有义务去遵从它们的通用约定；如果不这样做，将会阻止其他依赖于约定的类 (例如 HashMap 和 HashSet) 与此类一起正常工作。

重写 equals 方法看起来很简单，但是有很多方式会导致重写出错，其结果可能是可怕的。避免此问题的最简单方法是不覆盖 equals 方法，在这种情况下，类的每个实例只与自身相等。如果满足以下任一下条件，则说明是正确的做法：

每个类的实例都是固有唯一的。 对于像 Thread 这样代表活动实体而不是值的类来说，这是正确的。 Object 提供的 equals 实现对这些类完全是正确的行为。

类不需要提供一个「逻辑相等（logical equality）」的测试功能。例如 java.util.regex.Pattern 可以重写 equals 方法检查两个是否代表完全相同的正则表达式 Pattern 实例，但是设计者并不认为客户需要或希望使用此功能。在这种情况下，从 Object 继承的 equals 实现是最合适的。

父类已经重写了 equals 方法，则父类行为完全适合于该子类。例如，大多数 Set 从 AbstractSet 继承了 equals 实现、List 从 AbstractList 继承了 equals 实现，Map 从 AbstractMap 的 Map 继承了 equals 实现。

类是私有的或包级私有的，可以确定它的 equals 方法永远不会被调用。如果你非常厌恶风险，可以重写 equals 方法，以确保不会被意外调用：

    @Override
    public boolean equals(Object o) {
        throw new AssertionError(); // Method is never called
    }
什么时候需要重写 equals 方法呢？如果一个类包含一个逻辑相等（logical equality）的概念，此概念有别于对象标识（object identity），而且父类还没有重写过 equals 方法。这通常用在值类（value classes）的情况。值类只是一个表示值的类，例如 Integer 或 String 类。程序员使用 equals 方法比较值对象的引用，期望发现它们在逻辑上是否相等，而不是引用相同的对象。重写 equals 方法不仅可以满足程序员的期望，它还支持重写过 equals 的实例作为 Map 的键（key），或者 Set 里的元素，以满足预期和期望的行为。

　　一种不需要 equals 方法重写的值类是使用实例控制（instance control）（详见第 1 条）的类，以确保每个值至多存在一个对象。 枚举类型（详见第 34 条）属于这个类别。 对于这些类，逻辑相等与对象标识是一样的，所以 Object 的 equals 方法作用逻辑 equals 方法。

　　当你重写 equals 方法时，必须遵守它的通用约定。Object 的规范如下： equals 方法实现了一个等价关系（equivalence relation）。它有以下这些属性:

***自反性***： 对于任何非空引用 x，x.equals(x) 必须返回 true。
***对称性***： 对于任何非空引用 x 和 y，如果且仅当 y.equals(x) 返回 true 时 x.equals(y) 必须返回 true。
***传递性***： 对于任何非空引用 x、y、z，如果 x.equals(y) 返回 true，y.equals(z) 返回 true，则 x.equals(z) 必须返回 true。
***一致性***： 对于任何非空引用 x 和 y，如果在 equals 比较中使用的信息没有修改，则 x.equals(y) 的多次调用必须始终返回 true 或始终返回 false。
对于任何非空引用 x，x.equals(null) 必须返回 false。

　　除非你喜欢数学，否则这看起来有点吓人，但不要忽略它！如果一旦违反了它，很可能会发现你的程序运行异常或崩溃，并且很难确定失败的根源。套用约翰·多恩（John Donne）的说法，没有哪个类是孤立存在的。一个类的实例常常被传递给另一个类的实例。许多类，包括所有的集合类，都依赖于传递给它们遵守 equals 约定的对象。

　　既然已经意识到违反 equals 约定的危险，让我们详细地讨论一下这个约定。好消息是，表面上看，这并不是很复杂。一旦你理解了，就不难遵守这一约定。

　　那么什么是等价关系？ 笼统地说，它是一个运算符，它将一组元素划分为彼此元素相等的子集。 这些子集被称为等价类（equivalence classes）。 为了使 equals 方法有用，每个等价类中的所有元素必须从用户的角度来说是可以互换（interchangeable）的。 现在让我们依次看下这个五个要求：

　　自反性（Reflexivity）——第一个要求只是说一个对象必须与自身相等。 很难想象无意中违反了这个规定。 如果你违反了它，然后把类的实例添加到一个集合中，那么 contains 方法可能会说集合中没有包含刚添加的实例。

　　对称性（Symmetry）——第二个要求是，任何两个对象必须在是否相等的问题上达成一致。与第一个要求不同的是，我们不难想象在无意中违反了这一要求。例如，考虑下面的类，它实现了不区分大小写的字符串。字符串被 toString 保存，但在 equals 比较中被忽略：

    import java.util.Objects;

    public final class CaseInsensitiveString {
        private final String s;

        public CaseInsensitiveString(String s){
            this.s = Objects.requireNonNull(s);
        }

        // Broken - violates symmetry!
        @Override
        public boolean equals(Object o){
            if(o instanceof CaseInsensitiveString){
                return s.equalsIgnoreCase(
                        ((CaseInsensitiveString)o).s);
            }
            if(o instanceof String){// One-way interoperability!
                return s.equalsIgnoreCase((String)o);
            }
            return false;
        }
    }


上面CaseInsensitiveString（大小写不敏感）类中的 equals 试图与正常的字符串进行操作，假设我们有一个不区分大小写的字符串和一个正常的字符串：

    CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
    String s = "polish";

    System.out.println(cis.equals(s)); // true
    System.out.println(s.equals(cis)); // false

正如所料，cis.equals(s) 返回 true。 问题是，尽管 CaseInsensitiveString 类中的 equals 方法知道正常字符串，但 String 类中的 equals 方法却忽略了不区分大小写的字符串。 因此，s.equals(cis) 返回 false，明显违反对称性。 假设把一个不区分大小写的字符串放入一个集合中：

    List<CaseInsensitiveString> list = new ArrayList<>();
    list.add(cis);

list.contains(s) 返回了什么？谁知道呢？在当前的 OpenJDK 实现中，它会返回 false，但这只是一个实现构件。在另一个实现中，它可以很容易地返回 true 或抛出运行时异常。一旦违反了 equals 约定，就不知道其他对象在面对你的对象时会如何表现了。

　　要消除这个问题，只需删除 equals 方法中与 String 类相互操作的恶意尝试。这样做之后，可以将该方法重构为单个返回语句:

    @Override
    public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString &&
                ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }

传递性（Transitivity）—— equals 约定的第三个要求是，如果第一个对象等于第二个对象，第二个对象等于第三个对象，那么第一个对象必须等于第三个对象。同样，也不难想象，无意中违反了这一要求。考虑子类的情况， 将新值组件（value component）添加到其父类中。换句话说，子类添加了一个信息，它影响了 equals 方法比较。让我们从一个简单不可变的二维整数类型 Point 类开始：

    public class Point {
        private final int x;
        private final int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof Point))
                return false;
            Point p = (Point) o;
            return p.x == x && p.y == y;
        }

        ...  // Remainder omitted
    }

假设想继承这个类，将表示颜色的 Color 类添加到 Point 类中：

    public class ColorPoint extends Point {
        private final Color color;

        public ColorPoint(int x, int y, Color color) {
            super(x, y);
            this.color = color;
        }

        ...  // Remainder omitted
    }
equals 方法应该是什么样子？如果完全忽略，则实现是从 Point 类上继承的，颜色信息在 equals 方法比较中被忽略。虽然这并不违反 equals 约定，但这显然是不可接受的。假设你写了一个 equals 方法，它只在它的参数是另一个具有相同位置和颜色的 ColorPoint 实例时返回 true：

    // Broken - violates symmetry!
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }

当你比较 Point 对象和 ColorPoint 对象时，可以会得到不同的结果，反之亦然。前者的比较忽略了颜色属性，而后者的比较会一直返回 false，因为参数的类型是错误的。为了让问题更加具体，我们创建一个 Point 对象和 ColorPoint 对象：

    Point p = new Point(1, 2);
    ColorPoint cp = new ColorPoint(1, 2, Color.RED);

　　p.equals(cp) 返回 true，但是 cp.equals(p) 返回 false。你可能想使用 ColorPoint.equals 通过混合比较的方式来解决这个问题。

    @Override
    public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;

    // If o is a normal Point, do a color-blind comparison
    if (!(o instanceof ColorPoint))
        return o.equals(this);

    // o is a ColorPoint; do a full comparison
    return super.equals(o) && ((ColorPoint) o).color == color;
    }

这种方法确实提供了对称性，但是丧失了传递性：

    ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
    Point p2 = new Point(1, 2);
    ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

现在，p1.equals(p2) 和 p2.equals(p3) 返回了 true，但是 p1.equals(p3) 却返回了 false，很明显违背了传递性的要求。前两个比较都是不考虑颜色信息的，而第三个比较时却包含颜色信息。

　　此外，这种方法可能导致无限递归：假设有两个 Point 的子类，比如 ColorPoint 和 SmellPoint，每个都有这种 equals 方法。 然后调用 myColorPoint.equals(mySmellPoint) 将抛出一个 StackOverflowError 异常。

　　那么解决方案是什么？ 事实证明，这是面向对象语言中关于等价关系的一个基本问题。 除非您愿意放弃面向对象抽象的好处，否则无法继承可实例化的类，并在保留 equals 约定的同时添加一个值组件。

　　你可能听说过，可以继承一个可实例化的类并添加一个值组件，同时通过在 equals 方法中使用一个 getClass 测试代替 instanceof 测试来保留 equals 约定：


# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/10.%20%E9%87%8D%E5%86%99equals%E6%96%B9%E6%B3%95%E6%97%B6%E9%81%B5%E5%AE%88%E9%80%9A%E7%94%A8%E7%BA%A6%E5%AE%9A
