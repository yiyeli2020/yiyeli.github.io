title: Effective-Java阅读笔记VI
date: 2020-1-14 14:56:12
categories: 2020年1月
tags: [Java]

---

第三章阅读：Methods Common to All Objects
11.重写 equals 方法时同时也要重写 hashcode 方法
每次重写 equals 方法时都必须重写 hashCode 方法，否则程序将无法正常运行。hashCode 方法必须遵从 Object 类指定的常规约定，并且必须执行合理的工作，将不相等的哈希码分配给不相等的实例。
12.始终重写 toString 方法
除非父类已经这样做了，否则在每个实例化的类中重写 Object 的 toString 实现。 它使得类更加舒适地使用和协助调试。 toString 方法应该以一种美观的格式返回对象的简明有用的描述。

<!-- more -->

# 重写 equals 方法时同时也要重写 hashcode 方法

在每个类中，在重写 equals 方法的时侯，一定要重写 hashcode 方法。 如果不这样做，你的类违反了 hashCode 的通用约定，这会阻止它在 HashMap 和 HashSet 这样的集合中正常工作。根据 Object 规范，以下时具体约定。

1.当在一个应用程序执行过程中，如果在 equals 方法比较中没有修改任何信息，在一个对象上重复调用 hashCode 方法时，它必须始终返回相同的值。从一个应用程序到另一个应用程序的每一次执行返回的值可以是不一致的。
2.如果两个对象根据 equals(Object) 方法比较是相等的，那么在两个对象上调用 hashCode 就必须产生的结果是相同的整数。
3.如果两个对象根据 equals(Object) 方法比较并不相等，则不要求在每个对象上调用 hashCode 都必须产生不同的结果。 但是，程序员应该意识到，为不相等的对象生成不同的结果可能会提高散列表（hash tables）的性能。
　　当无法重写 hashCode 时，所违反第二个关键条款是：相等的对象必须具有相等的哈希码（ hash codes）。 根据类的 equals 方法，两个不同的实例可能在逻辑上是相同的，但是对于 Object 类的 hashCode 方法，它们只是两个没有什么共同之处的对象。因此， Object 类的 hashCode 方法返回两个看似随机的数字，而不是按约定要求的两个相等的数字。

举例说明，假设你使用条目 10 中的 PhoneNumber 类的实例做为 HashMap 的键（key）：

    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 867, 5309), "Jenny");

你可能期望 m.get(new PhoneNumber(707, 867, 5309)) 方法返回 Jenny 字符串，但实际上，返回了 null。注意，这里涉及到两个 PhoneNumber 实例：一个实例插入到 HashMap 中，另一个作为判断相等的实例用来检索。PhoneNumber 类没有重写 hashCode 方法导致两个相等的实例返回了不同的哈希码，违反了 hashCode 约定。put 方法把 PhoneNumber 实例保存在了一个哈希桶（ hash bucket）中，但 get 方法却是从不同的哈希桶中去查找，即使恰好两个实例放在同一个哈希桶中，get 方法几乎肯定也会返回 null。因为 HashMap 做了优化，缓存了与每一项（entry）相关的哈希码，如果哈希码不匹配，则不会检查对象是否相等了。

　　解决这个问题很简单，只需要为 PhoneNumber 类重写一个合适的 hashCode 方法。hashCode 方法是什么样的？写一个不规范的方法的是很简单的。以下示例，虽然永远是合法的，但绝对不能这样使用：

    // The worst possible legal hashCode implementation - never use!
    @Override public int hashCode() { return 42; }

这是合法的，因为它确保了相等的对象具有相同的哈希码。这很糟糕，因为它确保了每个对象都有相同的哈希码。因此，每个对象哈希到同一个桶中，哈希表退化为链表。应该在线性时间内运行的程序，运行时间变成了平方级别。对于数据很大的哈希表而言，会影响到能够正常工作。

　　一个好的 hash 方法趋向于为不相等的实例生成不相等的哈希码。这也正是 hashCode 约定中第三条的表达。理想情况下，hash 方法为集合中不相等的实例均匀地分配 int 范围内的哈希码。实现这种理想情况可能是困难的。 幸运的是，要获得一个合理的近似的方式并不难。 以下是一个简单的配方：

1.声明一个 int 类型的变量 result，并将其初始化为对象中第一个重要属性 c 的哈希码，如下面步骤 2.a 中所计算的那样。（回顾条目 10，重要的属性是影响比较相等的领域。）

2.对于对象中剩余的重要属性 f，请执行以下操作：
a. 比较属性 f 与属性 c 的 int 类型的哈希码：

    -- i. 如果这个属性是基本类型的，使用 Type.hashCode(f) 方法计算，其中 Type 类是对应属性 f 基本类型的包装类。
    -- ii. 如果该属性是一个对象引用，并且该类的 equals 方法通过递归调用 equals 来比较该属性，并递归地调用 hashCode 方法。 如果需要更复杂的比较，则计算此字段的“范式（“canonical representation）”，并在范式上调用 hashCode。 如果该字段的值为空，则使用 0（也可以使用其他常数，但通常来使用 0 表示）。
     -- iii. 如果属性 f 是一个数组，把它看作每个重要的元素都是一个独立的属性。 也就是说，通过递归地应用这些规则计算每个重要元素的哈希码，并且将每个步骤 2.b 的值合并。 如果数组没有重要的元素，则使用一个常量，最好不要为 0。如果所有元素都很重要，则使用 Arrays.hashCode 方法。

b. 将步骤 2.a 中属性 c 计算出的哈希码合并为如下结果：result = 31 * result + c;

3.返回 result 值。

当你写完 hashCode 方法后，问自己是否相等的实例有相同的哈希码。 编写单元测试来验证你的直觉（除非你使用 AutoValue 框架来生成你的 equals 和 hashCode 方法，在这种情况下，你可以放心地忽略这些测试）。 如果相同的实例有不相等的哈希码，找出原因并解决问题。

　　可以从哈希码计算中排除派生属性（derived fields）。换句话说，如果一个属性的值可以根据参与计算的其他属性值计算出来，那么可以忽略这样的属性。您必须排除在 equals 比较中没有使用的任何属性，否则可能会违反 hashCode 约定的第二条。

　　步骤 2.b 中的乘法计算结果取决于属性的顺序，如果类中具有多个相似属性，则产生更好的散列函数。 例如，如果乘法计算从一个 String 散列函数中被省略，则所有的字符将具有相同的散列码。 之所以选择 31，因为它是一个奇数的素数。 如果它是偶数，并且乘法溢出，信息将会丢失，因为乘以 2 相当于移位。 使用素数的好处不太明显，但习惯上都是这么做的。 31 的一个很好的特性，是在一些体系结构中乘法可以被替换为移位和减法以获得更好的性能：31 * i ==（i << 5） - i。 现代 JVM 可以自动进行这种优化。

　　让我们把上述办法应用到 PhoneNumber 类中：

    // Typical hashCode method
    @Override
    public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }

因为这个方法返回一个简单的确定性计算的结果，它的唯一的输入是 PhoneNumber 实例中的三个重要的属性，所以显然相等的 PhoneNumber 实例具有相同的哈希码。 实际上，这个方法是 PhoneNumber 的一个非常好的 hashCode 实现，与 Java 平台类库中的实现一样。 它很简单，速度相当快，并且合理地将不相同的电话号码分散到不同的哈希桶中。

　　虽然在这个项目的方法产生相当好的哈希函数，但并不是最先进的。 它们的质量与 Java 平台类库的值类型中找到的哈希函数相当，对于大多数用途来说都是足够的。 如果真的需要哈希函数而不太可能产生碰撞，请参阅 Guava 框架的的com.google.common.hash.Hashing [Guava] 方法。

　　Objects 类有一个静态方法，它接受任意数量的对象并为它们返回一个哈希码。 这个名为 hash 的方法可以让你编写一行 hashCode 方法，其质量与根据这个项目中的上面编写的方法相当。 不幸的是，它们的运行速度更慢，因为它们需要创建数组以传递可变数量的参数，以及如果任何参数是基本类型，则进行装箱和取消装箱。 这种哈希函数的风格建议仅在性能不重要的情况下使用。 以下是使用这种技术编写的 PhoneNumber 的哈希函数：

    // One-line hashCode method - mediocre performance
    @Override
    public int hashCode() {
       return Objects.hash(lineNum, prefix, areaCode);
    }

如果一个类是不可变的，并且计算哈希码的代价很大，那么可以考虑在对象中缓存哈希码，而不是在每次请求时重新计算哈希码。 如果你认为这种类型的大多数对象将被用作哈希键，那么应该在创建实例时计算哈希码。 否则，可以选择在首次调用 hashCode 时延迟初始化（lazily initialize）哈希码。 需要注意确保类在存在延迟初始化属性的情况下保持线程安全（项目 83）。 PhoneNumber 类不适合这种情况，但只是为了展示它是如何完成的。 请注意，属性 hashCode 的初始值（在本例中为 0）不应该是通常创建的实例的哈希码：

    // hashCode method with lazily initialized cached hash code
    private int hashCode; // Automatically initialized to 0

    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }

不要试图从哈希码计算中排除重要的属性来提高性能。 由此产生的哈希函数可能运行得更快，但其质量较差可能会降低哈希表的性能，使其无法使用。 具体来说，哈希函数可能会遇到大量不同的实例，这些实例主要在你忽略的区域中有所不同。 如果发生这种情况，哈希函数将把所有这些实例映射到少许哈希码上，而应该以线性时间运行的程序将会运行平方级的时间。

　　这不仅仅是一个理论问题。 在 Java 2 之前，String 类哈希函数在整个字符串中最多使用 16 个字符，从第一个字符开始，在整个字符串中均匀地选取。 对于大量的带有层次名称的集合（如 URL），此功能正好显示了前面描述的病态行为。

　　不要为 hashCode 返回的值提供详细的规范，因此客户端不能合理地依赖它； 你可以改变它的灵活性。 Java 类库中的许多类（例如 String 和 Integer）都将 hashCode 方法返回的确切值指定为实例值的函数。 这不是一个好主意，而是一个我们不得不忍受的错误：它妨碍了在未来版本中改进哈希函数的能力。 如果未指定细节并在散列函数中发现缺陷，或者发现了更好的哈希函数，则可以在后续版本中对其进行更改。

　　总之，每次重写 equals 方法时都必须重写 hashCode 方法，否则程序将无法正常运行。你的 hashCode 方法必须遵从 Object 类指定的常规约定，并且必须执行合理的工作，将不相等的哈希码分配给不相等的实例。如果使用第 51 页的配方，这很容易实现。如条目 10 所述，AutoValue 框架为手动编写 equals 和 hashCode 方法提供了一个很好的选择，IDE 也提供了一些这样的功能。

# 始终重写 toString 方法
虽然 Object 类提供了 toString 方法的实现，但它返回的字符串通常不是你的类的用户想要看到的。 它由类名后跟一个「at」符号（@）和哈希码的无符号十六进制表示组成，例如 PhoneNumber@163b91。 toString 的通用约定要求，返回的字符串应该是「一个简洁但内容丰富的表示，对人们来说是很容易阅读的」。虽然可以认为 PhoneNumber@163b91 简洁易读，但相比于 707-867-5309，但并不是很丰富 。 toString 通用约定「建议所有的子类重写这个方法」。好的建议，的确如此！

　　虽然它并不像遵守 equals 和 hashCode 约定那样重要 (条目 10 和 11)，但是提供一个良好的 toString 实现使你的类更易于使用，并对使用此类的系统更易于调试。当对象被传递到 println、printf、字符串连接操作符或断言，或者由调试器打印时，toString 方法会自动被调用。即使你从不调用对象上的 toString，其他人也可以。例如，对对象有引用的组件可能包含在日志错误消息中对象的字符串表示。如果未能重写 toString，则消息可能是无用的。

　　如果为 PhoneNumber 提供了一个很好的 toString 方法，那么生成一个有用的诊断消息就像下面这样简单：

    System.out.println("Failed to connect to " + phoneNumber);

程序员将以这种方式生成诊断消息，不管你是否重写 toString，但是除非你这样做，否则这些消息将不会有用。 提供一个很好的 toString 方法的好处不仅包括类的实例，同样有益于包含实例引用的对象，特别是集合。 打印 map 对象时你会看到哪一个，{Jenny=PhoneNumber@163b91} 还是 {Jenny=707-867-5309}?

　　实际上，toString 方法应该返回对象中包含的所有需要关注的信息，如电话号码示例中所示。 如果对象很大或者包含不利于字符串表示的状态，这是不切实际的。 在这种情况下，toString 应该返回一个摘要，如 Manhattan residential phone directory (1487536 listings) 或线程[main，5，main]。 理想情况下，字符串应该是不言自明的（线程示例并没有遵守这点）。 如果未能将所有对象的值得关注的信息包含在字符串表示中，则会导致一个特别烦人的处罚：测试失败报告如下所示：

    Assertion failure: expected {abc, 123}, but was {abc, 123}.

实现 toString 方法时，必须做出的一个重要决定是：在文档中指定返回值的格式。 建议你对值类进行此操作，例如电话号码或矩阵类。 指定格式的好处是它可以作为标准的，明确的，可读的对象表示。 这种表示形式可以用于输入、输出以及持久化可读性的数据对象，如 CSV 文件。 如果指定了格式，通常提供一个匹配的静态工厂或构造方法，是个好主意，所以程序员可以轻松地在对象和字符串表示之间来回转换。 Java 平台类库中的许多值类都采用了这种方法，包括 BigInteger，BigDecimal 和大部分基本类型包装类。

　　指定 toString 返回值的格式的缺点是，假设你的类被广泛使用，一旦指定了格式，就会终身使用。程序员将编写代码来解析表达式，生成它，并将其嵌入到持久数据中。如果在将来的版本中更改了格式的表示，那么会破坏他们的代码和数据，并且还会抱怨。但通过选择不指定格式，就可以保留在后续版本中添加信息或改进格式的灵活性。

　　无论是否决定指定格式，你都应该清楚地在文档中表明你的意图。如果指定了格式，则应该这样做。例如，这里有一个 toString 方法，该方法在条目 11 中使用 PhoneNumber 类：

    /**
     * Returns the string representation of this phone number.
     * The string consists of twelve characters whose format is
     * "XXX-YYY-ZZZZ", where XXX is the area code, YYY is the
     * prefix, and ZZZZ is the line number. Each of the capital
     * letters represents a single decimal digit.
     *
     * If any of the three parts of this phone number is too small
     * to fill up its field, the field is padded with leading zeros.
     * For example, if the value of the line number is 123, the last
     * four characters of the string representation will be "0123".
     */
    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }

如果你决定不指定格式，那么文档注释应该是这样的：

    /**
     * Returns a brief description of this potion. The exact details
     * of the representation are unspecified and subject to change,
     * but the following may be regarded as typical:
     *
     * "[Potion #9: type=love, smell=turpentine, look=india ink]"
     */
    @Override
    public String toString() { ... }

在阅读了这条注释之后，那些生成依赖于格式细节的代码或持久化数据的程序员，在这种格式发生改变的时候，只能怪他们自己。

　　无论是否指定格式，都可以通过编程方式访问 toString 返回的值中包含的信息。 例如，PhoneNumber 类应该包含 areaCode, prefix, lineNum 这三个属性。 如果不这样做，就会强迫程序员需要这些信息来解析字符串。 除了降低性能和程序员做不必要的工作之外，这个过程很容易出错，如果改变格式就会中断，并导致脆弱的系统。 由于未能提供访问器，即使已指定格式可能会更改，也可以将字符串格式转换为事实上的 API。

　　在静态工具类（详见第 4 条）中编写 toString 方法是没有意义的。 你也不应该在大多数枚举类型（条目 34）中写一个 toString 方法，因为 Java 为你提供了一个非常好的方法。 但是，你应该在任何抽象类中定义 toString 方法，该类的子类共享一个公共字符串表示形式。 例如，大多数集合实现上的 toString 方法都是从抽象集合类继承的。

　　Google 的开放源代码 AutoValue 工具在条目 10 中讨论过，它为你生成一个 toString 方法，就像大多数 IDE 工具一样。 这些方法非常适合告诉你每个属性的内容，但并不是专门针对类的含义。 因此，例如，为我们的 PhoneNumber 类使用自动生成的 toString 方法是不合适的（因为电话号码具有标准的字符串表示形式），但是对于我们的 Potion 类来说，这是完全可以接受的。 也就是说，自动生成的 toString 方法比从 Object 继承的方法要好得多，它不会告诉你对象的值。

　　回顾一下，除非父类已经这样做了，否则在每个实例化的类中重写 Object 的 toString 实现。 它使得类更加舒适地使用和协助调试。 toString 方法应该以一种美观的格式返回对象的简明有用的描述。

# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/11.%20%E9%87%8D%E5%86%99equals%E6%96%B9%E6%B3%95%E6%97%B6%E5%90%8C%E6%97%B6%E4%B9%9F%E8%A6%81%E9%87%8D%E5%86%99hashcode%E6%96%B9%E6%B3%95
