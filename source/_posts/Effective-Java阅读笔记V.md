title: Effective-Java阅读笔记V
date: 2020-1-6 09:46:12
categories: 2020年1月
tags: [Java]

---

第三章阅读：Methods Common to All Objects
09.使用 try-with-resources 语句替代 try-finally 语句
10.重写 equals 方法时遵守通用约定

除非必须：在很多情况下，不要重写 equals 方法，从 Object 继承的实现完全是你想要的。 如果你确实重写了 equals 方法，那么一定要比较这个类的所有重要属性，并且以保护前面 equals 约定里五个规定的方式去比较：自反性，对称性，传递性，一致性，非空性。

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

写在前面的读后感：这一节十分冗长，作者举例说明了尽量不要重写equals方法，如果一定要重写，那么要按照约定遵循自反性，对称性，传递性，一致性和非空性。

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
***非空性***：
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

    @Override
    public boolean equals(Object o) {
        if (o == null || o.getClass() != getClass())
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }

　只有当对象具有相同的实现类时，才会产生相同的效果。这看起来可能不是那么糟糕，但是结果是不可接受的:一个 Point 类子类的实例仍然是一个 Point 的实例，它仍然需要作为一个 Point 来运行，但是如果你采用这个方法，就会失败！假设我们要写一个方法来判断一个 Point 对象是否在 unitCircle 集合中。我们可以这样做：

    private static final Set<Point> unitCircle = Set.of(
            new Point( 1,  0), new Point( 0,  1),
            new Point(-1,  0), new Point( 0, -1));

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
    }

　虽然这可能不是实现功能的最快方法，但它可以正常工作。假设以一种不添加值组件的简单方式继承 Point 类，比如让它的构造方法跟踪记录创建了多少实例：

    public class CounterPoint extends Point {
        private static final AtomicInteger counter =
                new AtomicInteger();

        public CounterPoint(int x, int y) {
            super(x, y);
            counter.incrementAndGet();
        }

        public static int numberCreated() {
            return counter.get();
        }
    }

里氏替代原则（Liskov substitution principle）指出，任何类型的重要属性都应该适用于所有的子类型，因此任何为这种类型编写的方法都应该在其子类上同样适用。 这是我们之前声明的一个正式陈述，即 Point 的子类（如 CounterPoint）仍然是一个 Point，必须作为一个 Point 类来看待。 但是，假设我们将一个 CounterPoint 对象传递给 onUnitCircle 方法。 如果 Point 类使用基于 getClass 的 equals 方法，则无论 CounterPoint 实例的 x 和 y 坐标如何，onUnitCircle 方法都将返回 false。 这是因为大多数集合（包括 onUnitCircle 方法使用的 HashSet）都使用 equals 方法来测试是否包含元素，并且 CounterPoint 实例并不等于任何 Point 实例。 但是，如果在 Point 上使用了适当的基于 instanceof 的 equals 方法，则在使用 CounterPoint 实例呈现时，同样的 onUnitCircle 方法可以正常工作。

　　虽然没有令人满意的方法来继承一个可实例化的类并添加一个值组件，但是有一个很好的变通方法：按照条目 18 的建议，“优先使用组合而不是继承”。取代继承 Point 类的 ColorPoint 类，可以在 ColorPoint 类中定义一个私有 Point 属性，和一个公共的视图（view）（详见第 6 条）方法，用来返回具有相同位置的 ColorPoint 对象。

    // Adds a value component without violating the equals contract
    public class ColorPoint {
        private final Point point;
        private final Color color;

        public ColorPoint(int x, int y, Color color) {
            point = new Point(x, y);
            this.color = Objects.requireNonNull(color);
        }

        /**
         * Returns the point-view of this color point.
         */
        public Point asPoint() {
            return point;
        }

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof ColorPoint))
                return false;
            ColorPoint cp = (ColorPoint) o;
            return cp.point.equals(point) && cp.color.equals(color);
        }

        ...    // Remainder omitted
    }

Java 平台类库中有一些类可以继承可实例化的类并添加一个值组件。 例如，java.sql.Timestamp 继承了 java.util.Date 并添加了一个 nanoseconds 字段。 Timestamp 的等价 equals 确实违反了对称性，并且如果 Timestamp 和 Date 对象在同一个集合中使用，或者以其他方式混合使用，则可能导致不稳定的行为。 Timestamp 类有一个免责声明，告诫程序员不要混用 Timestamp 和 Date。 虽然只要将它们分开使用就不会遇到麻烦，但没有什么可以阻止你将它们混合在一起，并且由此产生的错误可能很难调试。 Timestamp 类的这种行为是一个错误，不应该被仿效。

你可以将值组件添加到抽象类的子类中，而不会违反 equals 约定。这对于通过遵循第 23 个条目中“优先考虑类层级（class hierarchies）来代替标记类（tagged classes）”中的建议而获得的类层级，是非常重要的。例如，可以有一个没有值组件的抽象类 Shape，子类 Circle 有一个 radius 属性，另一个子类 Rectangle 包含 length 和 width 属性 。 只要不直接创建父类实例，就不会出现前面所示的问题。

　　一致性（Consistent）——equals 约定的第四个要求是，如果两个对象是相等的，除非一个（或两个）对象被修改了， 那么它们必须始终保持相等。 换句话说，可变对象可以在不同时期可以与不同的对象相等，而不可变对象则不会。 当你写一个类时，要认真思考它是否应该设计为不可变的（详见第 17 条）。 如果你认为应该这样做，那么确保你的 equals 方法强制执行这样的限制：相等的对象永远相等，不相等的对象永远都不会相等。

　　不管一个类是不是不可变的，都不要写一个依赖于不可靠资源的 equals 方法。 如果违反这一禁令，满足一致性要求是非常困难的。 例如，java.net.URL 类中的 equals 方法依赖于与 URL 关联的主机的 IP 地址的比较。 将主机名转换为 IP 地址可能需要访问网络，并且不能保证随着时间的推移会产生相同的结果。 这可能会导致 URL 类的 equals 方法违反 equals 约定，并在实践中造成问题。 URL 类的 equals 方法的行为是一个很大的错误，不应该被效仿。 不幸的是，由于兼容性的要求，它不能改变。 为了避免这种问题，equals 方法应该只对内存驻留对象执行确定性计算。

　　非空性（Non-nullity）——最后 equals 约定的要求没有官方的名称，所以我冒昧地称之为“非空性”。意思是说说所有的对象都必须不等于 null。虽然很难想象在调用 o.equals(null) 的响应中意外地返回 true，但不难想象不小心抛出 NullPointerException 异常的情况。通用的约定禁止抛出这样的异常。许多类中的 equals 方法都会明确阻止对象为 null 的情况：

    @Override
    public boolean equals(Object o) {
        if (o == null)
            return false;
        ...
    }

　这个判断是不必要的。 为了测试它的参数是否相等，equals 方法必须首先将其参数转换为合适类型，以便调用访问器或允许访问的属性。 在执行类型转换之前，该方法必须使用 instanceof 运算符来检查其参数是否是正确的类型：

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof MyType))
            return false;
        MyType mt = (MyType) o;
        ...
    }

如果此类型检查漏掉，并且 equals 方法传递了错误类型的参数，那么 equals 方法将抛出 ClassCastException 异常，这违反了 equals 约定。 但是，如果第一个操作数为 null，则指定 instanceof 运算符返回 false，而不管第二个操作数中出现何种类型[JLS，15.20.2]。 因此，如果传入 null，类型检查将返回 false，因此不需要 明确的 null 检查。

划重点：
　　*** 综合起来，以下是编写高质量 equals 方法的配方（recipe）：***

1.使用 == 运算符检查参数是否为该对象的引用。如果是，返回 true。这只是一种性能优化，但是如果这种比较可能很昂贵的话，那就值得去做。

2.使用 instanceof 运算符来检查参数是否具有正确的类型。 如果不是，则返回 false。 通常，正确的类型是 equals 方法所在的那个类。 有时候，改类实现了一些接口。 如果类实现了一个接口，该接口可以改进 equals 约定以允许实现接口的类进行比较，那么使用接口。 集合接口（如 Set，List，Map 和 Map.Entry）具有此特性。

3.参数转换为正确的类型。因为转换操作在 instanceof 中已经处理过，所以它肯定会成功。

4.对于类中的每个「重要」的属性，请检查该参数属性是否与该对象对应的属性相匹配。如果所有这些测试成功，返回 true，否则返回 false。如果步骤 2 中的类型是一个接口，那么必须通过接口方法访问参数的属性;如果类型是类，则可以直接访问属性，这取决于属性的访问权限。

　　对于类型为非 float 或 double 的基本类型，使用 == 运算符进行比较；对于对象引用属性，递归地调用 equals 方法；对于 float 基本类型的属性，使用静态 Float.compare(float, float) 方法；对于 double 基本类型的属性，使用 Double.compare(double, double) 方法。由于存在 Float.NaN，-0.0f 和类似的 double 类型的值，所以需要对 float 和 double 属性进行特殊的处理；有关详细信息，请参阅 JLS 15.21.1 或 Float.equals 方法的详细文档。 虽然你可以使用静态方法 Float.equals 和 Double.equals 方法对 float 和 double 基本类型的属性进行比较，这会导致每次比较时发生自动装箱，引发非常差的性能。 对于数组属性，将这些准则应用于每个元素。 如果数组属性中的每个元素都很重要，请使用其中一个重载的 Arrays.equals 方法。

　　某些对象引用的属性可能合法地包含 null。 为避免出现 NullPointerException 异常，请使用静态方法 Objects.equals(Object, Object) 检查这些属性是否相等。

　　对于一些类，例如上的 CaseInsensitiveString 类，属性比较相对于简单的相等性测试要复杂得多。在这种情况下，你想要保存属性的一个规范形式（canonical form），这样 equals 方法就可以基于这个规范形式去做开销很小的精确比较，来取代开销很大的非标准比较。这种方式其实最适合不可变类（详见第 17 条）。一旦对象发生改变，一定要确保把对应的规范形式更新到最新。

　　equals 方法的性能可能受到属性比较顺序的影响。 为了获得最佳性能，你应该首先比较最可能不同的属性，开销比较小的属性，或者最好是两者都满足（derived fields）。 你不要比较不属于对象逻辑状态的属性，例如用于同步操作的 lock 属性。 不需要比较可以从“重要属性”计算出来的派生属性，但是这样做可以提高 equals 方法的性能。 如果派生属性相当于对整个对象的摘要描述，比较这个属性将节省在比较失败时再去比较实际数据的开销。 例如，假设有一个 Polygon 类，并缓存该区域。 如果两个多边形的面积不相等，则不必费心比较它们的边和顶点。

　　当你完成编写完 equals 方法时，问你自己三个问题：它是对称的吗?它是传递吗?它是一致的吗?除此而外，编写单元测试加以排查，除非使用 AutoValue 框架（第 49 页）来生成 equals 方法，在这种情况下可以安全地省略测试。如果持有的属性失败，找出原因，并相应地修改 equals 方法。当然，equals 方法也必须满足其他两个属性 (自反性和非空性)，但这两个属性通常都会满足。

　　在下面这个简单的 PhoneNumber 类中展示了根据之前的配方构建的 equals 方法：

    public final class PhoneNumber {

        private final short areaCode, prefix, lineNum;

        public PhoneNumber(int areaCode, int prefix, int lineNum) {
            this.areaCode = rangeCheck(areaCode, 999, "area code");
            this.prefix = rangeCheck(prefix, 999, "prefix");
            this.lineNum = rangeCheck(lineNum, 9999, "line num");
        }

        private static short rangeCheck(int val, int max, String arg) {
            if (val < 0 || val > max)
                throw new IllegalArgumentException(arg + ": " + val);

            return (short) val;
        }

        @Override
        public boolean equals(Object o) {
            if (o == this)
                return true;
            if (!(o instanceof PhoneNumber))
                return false;

            PhoneNumber pn = (PhoneNumber) o;

            return pn.lineNum == lineNum && pn.prefix == prefix
                    && pn.areaCode == areaCode;
        }

        ... // Remainder omitted
    }

以下是一些最后提醒：

当重写 equals 方法时，同时也要重写 hashCode 方法（详见第 11 条）。
不要让 equals 方法试图太聪明。 如果只是简单地测试用于相等的属性，那么要遵守 equals 约定并不困难。如果你在寻找相等方面过于激进，那么很容易陷入麻烦。一般来说，考虑到任何形式的别名通常是一个坏主意。例如，File 类不应该试图将引用的符号链接等同于同一文件对象。幸好 File 类并没这么做。
在 equal 时方法声明中，不要将参数 Object 替换成其他类型。 对于程序员来说，编写一个看起来像这样的 equals 方法并不少见，然后花上几个小时苦苦思索为什么它不能正常工作：

    // Broken - parameter type must be Object!
    public boolean equals(MyClass o) {   
        …
    }
　问题在于这个方法并没有重写 Object.equals 方法，它的参数是 Object 类型的，这样写只是重载了 equals 方法（详见第 52 条）。 即使除了正常的方法之外，提供这种“强类型”的 equals 方法也是不可接受的，因为它可能会导致子类中的 Override 注解产生误报，提供不安全的错觉。 在这里，使用 Override 注解会阻止你犯这个错误 （详见第 40 条）。这个 equals 方法不会编译，错误消息会告诉你到底错在哪里：

编写和测试 equals（和 hashCode）方法很繁琐，生的代码也很普通。替代手动编写和测试这些方法的优雅的手段是，使用谷歌 AutoValue 开源框架，该框架自动为你生成这些方法，只需在类上添加一个注解即可。在大多数情况下，AutoValue 框架生成的方法与你自己编写的方法本质上是相同的。

　　很多 IDE（例如 Eclipse，NetBeans，IntelliJ IDEA 等）也有生成 equals 和 hashCode 方法的功能，但是生成的源代码比使用 AutoValue 框架的代码更冗长、可读性更差，不会自动跟踪类中的更改，因此需要进行测试。这就是说，使用 IDE 工具生成 equals(和 hashCode) 方法通常比手动编写它们更可取，因为 IDE 工具不会犯粗心大意的错误，而人类则会。

　　总之，除非必须：在很多情况下，不要重写 equals 方法，从 Object 继承的实现完全是你想要的。 如果你确实重写了 equals 方法，那么一定要比较这个类的所有重要属性，并且以保护前面 equals 约定里五个规定的方式去比较。

# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/10.%20%E9%87%8D%E5%86%99equals%E6%96%B9%E6%B3%95%E6%97%B6%E9%81%B5%E5%AE%88%E9%80%9A%E7%94%A8%E7%BA%A6%E5%AE%9A
