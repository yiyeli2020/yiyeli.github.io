title: Effctive-Java阅读笔记III
date: 2019-12-31 15:06:12
categories: 2019年12月
tags: [Design Pattern，Java]

---

第二章阅读：创建和销毁对象
05.依赖注入优于硬连接资源（hardwiring resources）
06.避免创建不必要的对象

<!-- more -->

# 依赖注入优于硬连接资源（hardwiring resources）

　许多类依赖于一个或多个底层资源。例如，拼写检查器依赖于字典。将此类类实现为静态工具类并不少见 （详见第 4 条）:

    // Inappropriate use of static utility - inflexible & untestable!
    public class SpellChecker {
        private static final Lexicon dictionary = ...;

        private SpellChecker() {} // Noninstantiable

        public static boolean isValid(String word) { ... }
        public static List<String> suggestions(String typo) { ... }
    }

同样地，将它们实现为单例的做法也并不少见（详见第 3 条）：

    // Inappropriate use of singleton - inflexible & untestable!
    public class SpellChecker {
        private final Lexicon dictionary = ...;

        private SpellChecker(...) {}
        public static INSTANCE = new SpellChecker(...);

        public boolean isValid(String word) { ... }
        public List<String> suggestions(String typo) { ... }
    }

这两种方法都不令人满意，因为他们都是假设只有一本字典可用。实际上，每种语言都有自己的字典，特殊的字典被用于特殊的词汇表。另外，可能还需要用特殊的词典进行测试。想当然地认为一本字典就足够了，这是一厢情愿的想法。

　　可以通过使 dictionary 属性设置为非final，final关键字修饰的成员变量是常量，修饰的类不能被继承。修饰的成员方法是不能被子类重写的。并添加一个方法来更改现有拼写检查器中的字典，从而让SpellChecker 支持多个字典，但是这样的设置显得非常笨拙、容易出错、并且无法并行工作。静态工具类和单例类不适合与需要引用底层资源的类。

　　这里所需要的是能够支持类的多个实例 （在我们的示例中是指 SpellChecker），每个实例都使用客户端所期望的资源（在我们的例子中是 dictionary）。满足这一需求的简单模式是，在创建新实例时将资源传递到构造器中。 这是依赖项注入（dependency injection）的一种形式：字典是拼写检查器的一个依赖项，当它创建时被注入到拼写检查器中。

    // Dependency injection provides flexibility and testability
    public class SpellChecker {
        private final Lexicon dictionary;

        public SpellChecker(Lexicon dictionary) {
            this.dictionary = Objects.requireNonNull(dictionary);
        }

        public boolean isValid(String word) { ... }
        public List<String> suggestions(String typo) { ... }
    }

　　依赖注入模式非常简单，许多程序员使用它多年而不知道它有一个名字。 虽然我们的拼写检查器的例子只有一个资源（字典），但是依赖项注入可以使用任意数量的资源和任意依赖图。 它保持了不变性（详见第 17 条），因此多个客户端可以共享依赖对象（假设客户需要相同的底层资源）。 依赖注入同样适用于构造器、静态工厂（详见第 1 条）和 builder 模式（详见第 2 条）。

　　该模式的一个有用的变体是将资源工厂传递给构造器。 工厂是可以重复调用以创建类型实例的对象。 这种工厂体现了工厂方法模式（Factory Method pattern）[Gamma95]。 Java 8 中引入的 Supplier<T> 接口非常适合代表工厂。 在输入上采用 Supplier<T> 的方法通常应该使用有界的通配符类型（bounded wildcard type）（详见第 31 条）约束工厂的类型参数，以便客户端能够传入一个工厂，来创建指定类型的任意子类型。例如，下面是一个生产马赛克的方法，它利用客户端提供的工厂来生产每一片马赛克：

    Mosaic create(Supplier<? extends Tile> tileFactory) { ... }

　　尽管依赖注入极大地提高了灵活性和可测试性，但它可能使大型项目变得混乱，这些项目通常包含数千个依赖项。使用依赖注入框架（如 Dagger [Dagger]、Guice [Guice] 或 Spring [Spring]）可以消除这些混乱。这些框架的使用超出了本书的范围，但是请注意，为手动依赖注入而设计的 API 非常适合这些框架的使用。

　　总而言之，不要用单例和静态工具类来实现依赖一个或多个底层资源的类，且该资源的行为会影响到该类的行为；也不要直接用这个类来创建这些资源。而应该将这些资源或者工厂传给构造器（或者静态工厂，或者构建器），通过它们来创建类。这个实践就被称作依赖注人，它极大地提升了类的灵活性、可重用性和可测试性。

# 避免创建不必要的对象

　通过使用静态工厂方法（详见第 1 条）和构造器，可以避免创建不需要的对象。例如，工厂方法 Boolean.valueOf(String) 比构造方法 Boolean(String) 更可取，后者在 Java 9 中被弃用。构造方法每次调用时都必须创建一个新对象，而工厂方法永远不需要这样做，在实践中也不需要。除了重用不可变对象，如果知道它们不会被修改，还可以重用可变对象。
一些对象的创建比其他对象的创建要昂贵得多。 如果要重复使用这样一个「昂贵的对象」，建议将其缓存起来以便重复使用。 不幸的是，当创建这样一个对象时并不总是很直观明显的。 假设你想写一个方法来确定一个字符串是否是一个有效的罗马数字。 以下是使用正则表达式完成此操作时最简单方法：

    // Performance can be greatly improved!
    static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
这个实现的问题在于它依赖于 String.matches 方法。 虽然 String.matches 是检查字符串是否与正则表达式匹配的最简单方法，但它不适合在性能临界的情况下重复使用。 问题是它在内部为正则表达式创建一个 Pattern 实例，并且只使用它一次，之后它就有资格进行垃圾收集。 创建 Pattern 实例是昂贵的，因为它需要将正则表达式编译成有限状态机（finite state machine）。

　　为了提高性能，作为类初始化的一部分，将正则表达式显式编译为一个 Pattern 实例（不可变），缓存它，并在 isRomanNumeral 方法的每个调用中重复使用相同的实例：

    // Reusing expensive object for improved performance
    public class RomanNumerals {
        private static final Pattern ROMAN = Pattern.compile(
                "^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

        static boolean isRomanNumeral(String s) {
            return ROMAN.matcher(s).matches();
        }
    }
如果经常调用，isRomanNumeral 的改进版本的性能会显著提升。

　　如果包含 isRomanNumeral 方法的改进版本的类被初始化，但该方法从未被调用，则 ROMAN 属性则没必要初始化。 在第一次调用 isRomanNumeral 方法时，可以通过延迟初始化（ lazily initializing）属性（详见第 83 条）来排除初始化，但一般不建议这样做。 延迟初始化常常会导致实现复杂化，而性能没有可衡量的改进（详见第 67 条）。

　　当一个对象是不可变的时，很明显它可以被安全地重用，但是在其他情况下，它远没有那么明显，甚至是违反直觉的。考虑适配器（adapters）的情况[Gamma95]，也称为视图（views）。一个适配器是一个对象，它委托一个支持对象（backing object），提供一个可替代的接口。由于适配器没有超出其支持对象的状态，因此不需要为给定对象创建多个给定适配器的实例。

例如，Map 接口的 keySet 方法返回 Map 对象的 Set 视图，包含 Map 中的所有 key。 天真地说，似乎每次调用 keySet 都必须创建一个新的 Set 实例，但是对给定 Map 对象的 keySet 的每次调用都返回相同的 Set 实例。 尽管返回的 Set 实例通常是可变的，但是所有返回的对象在功能上都是相同的：当其中一个返回的对象发生变化时，所有其他对象也都变化，因为它们全部由相同的 Map 实例支持。 虽然创建 keySet 视图对象的多个实例基本上是无害的，但这是没有必要的，也没有任何好处。

　　另一种创建不必要的对象的方法是自动装箱（auto boxing），它允许程序员混用基本类型和包装的基本类型，根据需要自动装箱和拆箱。 自动装箱模糊不清，但不会消除基本类型和装箱基本类型之间的区别。 有微妙的语义区别和不那么细微的性能差异（详见第 61 条）。 考虑下面的方法，它计算所有正整数的总和。 要做到这一点，程序必须使用 long 类型，因为 int 类型不足以保存所有正整数的总和：

    // Hideously slow! Can you spot the object creation?
    private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }

这个程序的结果是正确的，但由于写错了一个字符，运行的结果要比实际慢很多。变量 sum 被声明成了 Long 而不是 long，这意味着程序构造了大约 231 不必要的 Long 实例（大约每次往 Long 类型的 sum 变量中增加一个 long 类型构造的实例），把 sum 变量的类型由 Long 改为 long。这个教训很明显：优先使用基本类型而不是装箱的基本类型，也要注意无意识的自动装箱。

　　这个条目不应该被误解为暗示对象创建是昂贵的，应该避免创建对象。 相反，使用构造方法创建和回收小的对象是非常廉价，构造方法只会做很少的显示工作，尤其是在现代 JVM 实现上。 创建额外的对象以增强程序的清晰度，简单性或功能性通常是件好事。

　　相反，除非池中的对象非常重量级，否则通过维护自己的对象池来避免对象创建是一个坏主意。对象池的典型例子就是数据库连接。建立连接的成本非常高，因此重用这些对象是有意义的。但是，一般来说，维护自己的对象池会使代码混乱，增加内存占用，并损害性能。现代 JVM 实现具有高度优化的垃圾收集器，它们在轻量级对象上轻松胜过此类对象池。

　　这个条目的对应点是针对条目 50 的防御性复制（defensive copying）。 目前的条目说：「当你应该重用一个现有的对象时，不要创建一个新的对象」，而条目 50 说：「不要重复使用现有的对象，当你应该创建一个新的对象时。」请注意，重用防御性复制所要求的对象所付出的代价，要远远大于不必要地创建重复的对象。 未能在需要的情况下防御性复制会导致潜在的错误和安全漏洞；而不必要地创建对象只会影响程序的风格和性能。

#参考资料
[1] https://sjsdfg.github.io/effective-java-3rd-chinese/#/notes/05.%20%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5%E4%BC%98%E4%BA%8E%E7%A1%AC%E8%BF%9E%E6%8E%A5%E8%B5%84%E6%BA%90(hardwiring%20resources)
