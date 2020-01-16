title: Effctive-Java阅读笔记VII
date: 2020-1-15 10:48:12
categories: 2020年1月
tags: [Java]

---

第三章阅读：Methods Common to All Objects
13.谨慎地重写 clone 方法
14.考虑实现 Comparable 接口
总而言之，无论何时实现具有合理排序的值类，你都应该让该类实现 Comparable 接口，以便在基于比较的集合中轻松对其实例进行排序，搜索和使用。 比较
compareTo 方法的实现中的字段值时，请避免使用「<」和「>」运算符。 相反，使用包装类中的静态 compare 方法或 Comparator 接口中的构建方法

<!-- more -->

# 谨慎地重写 clone 方法

TODO：这一条没有读懂，留待看过20/65条后再回头来读。

Cloneable 接口的目的是作为一个 mixin 接口 （详见第 20 条），公布这样的类允许克隆。不幸的是，它没有达到这个目的。它的主要缺点是缺少 clone 方法，而 Object 的 clone 方法是受保护的。你不能，不借助反射 （详见第 65 条），仅仅因为它实现了 Cloneable 接口，就调用对象上的 clone 方法。即使是反射调用也可能失败，因为不能保证对象具有可访问的 clone 方法。尽管存在许多缺陷，该机制在合理的范围内使用，所以理解它是值得的。这个条目告诉你如何实现一个行为良好的 clone 方法，在适当的时候讨论这个方法，并提出替代方案。

　　既然 Cloneable 接口不包含任何方法，那它用来做什么？ 它决定了 Object 的受保护的 clone 方法实现的行为：如果一个类实现了 Cloneable 接口，那么 Object 的 clone 方法将返回该对象的逐个属性（field-by-field）拷贝；否则会抛出 CloneNotSupportedException 异常。这是一个非常反常的接口使用，而不应该被效仿。 通常情况下，实现一个接口用来表示可以为客户做什么。但对于 Cloneable 接口，它会修改父类上受保护方法的行为。

　　虽然规范并没有说明，但在实践中，实现 Cloneable 接口的类希望提供一个正常运行的公共 clone 方法。为了实现这一目标，该类及其所有父类必须遵循一个复杂的、不可执行的、稀疏的文档协议。由此产生的机制是脆弱的、危险的和不受语言影响的（extralinguistic）：它创建对象而不需要调用构造方法。

　　clone 方法的通用规范很薄弱的。 以下内容是从 Object 规范中复制出来的：

　　创建并返回此对象的副本。 「复制（copy）」的确切含义可能取决于对象的类。 一般意图是，对于任何对象 x，表达式 x.clone() != x 返回 true，并且 x.clone().getClass() == x.getClass() 也返回 true，但它们不是绝对的要求，但通常情况下，x.clone().equals(x) 返回 true，当然这个要求也不是绝对的。

　　根据约定，这个方法返回的对象应该通过调用 super.clone 方法获得的。 如果一个类和它的所有父类（Object 除外）都遵守这个约定，情况就是如此，x.clone().getClass() == x.getClass()。

　　根据约定，返回的对象应该独立于被克隆的对象。 为了实现这种独立性，在返回对象之前，可能需要修改由 super.clone 返回的对象的一个或多个属性。

　　这种机制与构造方法链（chaining）很相似，只是它没有被强制执行；如果一个类的 clone 方法返回一个通过调用构造方法获得而不是通过调用 super.clone 的实例，那么编译器不会抱怨，但是如果一个类的子类调用了 super.clone，那么返回的对象包含错误的类，从而阻止子类 clone 方法正常执行。如果一个类重写的 clone 方法是有 final 修饰的，那么这个约定可以被安全地忽略，因为子类不需要担心。但是，如果一个 final 类有一个不调用 super.clone 的 clone 方法，那么这个类没有理由实现 Cloneable 接口，因为它不依赖于 Object 的 clone 实现的行为。

　　假设你希望在一个类中实现 Cloneable 接口，它的父类提供了一个行为良好的 clone 方法。首先调用 super.clone。 得到的对象将是原始的完全功能的复制品。 在你的类中声明的任何属性将具有与原始属性相同的值。 如果每个属性包含原始值或对不可变对象的引用，则返回的对象可能正是你所需要的，在这种情况下，不需要进一步的处理。 例如，对于条目 11 中的 PhoneNumber 类，情况就是这样，但是请注意，不可变类永远不应该提供 clone 方法，因为这只会浪费复制。 有了这个警告，以下是 PhoneNumber 类的 clone 方法：

    // Clone method for class with no references to mutable state
    @Override public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  // Can't happen
        }
    }

为了使这个方法起作用，PhoneNumber 的类声明必须被修改，以表明它实现了 Cloneable 接口。 虽然 Object 类的 clone 方法返回 Object 类，但是这个 clone 方法返回 PhoneNumber 类。 这样做是合法和可取的，***因为 Java 支持协变返回类型。 换句话说，重写方法的返回类型可以是重写方法的返回类型的子类。*** 这消除了在客户端转换的需要。 在返回之前，我们必须将 Object 的 super.clone 的结果强制转换为 PhoneNumber，但保证强制转换成功。

　　super.clone 的调用包含在一个 try-catch 块中。 这是因为 Object 声明了它的 clone 方法来抛出 CloneNotSupportedException 异常，这是一个检查时异常。 由于 PhoneNumber 实现了 Cloneable 接口，所以我们知道调用 super.clone 会成功。 这里引用的需要表明 CloneNotSupportedException 应该是未被检查的（详见第 71条）。

　　如果对象包含引用可变对象的属性，则前面显示的简单 clone 实现可能是灾难性的。 例如，考虑条目 7 中的 Stack 类：

    public class Stack {

        private Object[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;

        public Stack() {
            this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
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

        // Ensure space for at least one more element.
        private void ensureCapacity() {
            if (elements.length == size)
                elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

假设你想让这个类可以克隆。 如果 clone 方法仅返回 super.clone() 调用的对象，那么生成的 Stack 实例在其 size 属性中具有正确的值，但 elements 属性引用与原始 Stack 实例相同的数组。 修改原始实例将破坏克隆中的不变量，反之亦然。 你会很快发现你的程序产生了无意义的结果，或者抛出 NullPointerException 异常。

　　这种情况永远不会发生，因为调用 Stack 类中的唯一构造方法。 实际上，clone 方法作为另一种构造方法; 必须确保它不会损坏原始对象，并且可以在克隆上正确建立不变量。 为了使 Stack 上的 clone 方法正常工作，它必须复制 stack 对象的内部。 最简单的方法是对元素数组递归调用 clone 方法：

    // Clone method for class with references to mutable state
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

请注意，我们不必将 elements.clone 的结果转换为 Object[] 数组。 在数组上调用 clone 会返回一个数组，其运行时和编译时类型与被克隆的数组相同。 这是复制数组的首选习语(行话)。 事实上，数组是 clone 机制的唯一有力的用途。

　　还要注意，如果 elements 属性是 final 的，则以前的解决方案将不起作用，因为克隆将被禁止向该属性分配新的值。 这是一个基本的问题：像序列化一样，Cloneable 体系结构与引用可变对象的 final 属性的正常使用不兼容，除非可变对象可以在对象和其克隆之间安全地共享。 为了使一个类可以克隆，可能需要从一些属性中移除 final 修饰符。

　　仅仅递归地调用 clone 方法并不总是足够的。 例如，假设您正在为哈希表编写一个 clone 方法，其内部包含一个哈希桶数组，每个哈希桶都指向「键-值」对链表的第一项。 为了提高性能，该类实现了自己的轻量级单链表，而没有使用 java 内部提供的 java.util.LinkedList：

    public class HashTable implements Cloneable {
        private Entry[] buckets = ...;
        private static class Entry {
            final Object key;
            Object value;
            Entry  next;

            Entry(Object key, Object value, Entry next) {
                this.key   = key;
                this.value = value;
                this.next  = next;  
            }
        }
        ... // Remainder omitted
    }
假设你只是递归地克隆哈希桶数组，就像我们为 Stack 所做的那样：

    // Broken clone method - results in shared mutable state!
    @Override public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

虽然被克隆的对象有自己的哈希桶数组，但是这个数组引用与原始数组相同的链表，这很容易导致克隆对象和原始对象中的不确定性行为。 要解决这个问题，你必须复制包含每个桶的链表。 下面是一种常见的方法：

    // Recursive clone method for class with complex mutable state
    public class HashTable implements Cloneable {
        private Entry[] buckets = ...;

        private static class Entry {
            final Object key;
            Object value;
            Entry  next;

            Entry(Object key, Object value, Entry next) {
                this.key   = key;
                this.value = value;
                this.next  = next;  
            }

            // Recursively copy the linked list headed by this Entry
            Entry deepCopy() {
                return new Entry(key, value,
                    next == null ? null : next.deepCopy());
            }
        }

        @Override public HashTable clone() {
            try {
                HashTable result = (HashTable) super.clone();
                result.buckets = new Entry[buckets.length];
                for (int i = 0; i < buckets.length; i++)
                    if (buckets[i] != null)
                        result.buckets[i] = buckets[i].deepCopy();
                return result;
            } catch (CloneNotSupportedException e) {
                throw new AssertionError();
            }
        }
        ... // Remainder omitted
    }


　私有类 HashTable.Entry 已被扩充以支持「深度复制」方法。 HashTable 上的 clone 方法分配一个合适大小的新哈希桶数组，迭代原来哈希桶数组，深度复制每个非空的哈希桶。 Entry 上的 deepCopy 方法递归地调用它自己以复制由头节点开始的整个链表。 如果哈希桶不是太长，这种技术很聪明并且工作正常。但是，克隆链表不是一个好方法，因为它为列表中的每个元素消耗一个栈帧（stack frame）。 如果列表很长，这很容易导致堆栈溢出。 为了防止这种情况发生，可以用迭代来替换 deepCopy 中的递归：


    // Iteratively copy the linked list headed by this Entry
    Entry deepCopy() {
       Entry result = new Entry(key, value, next);
       for (Entry p = result; p.next != null; p = p.next)
          p.next = new Entry(p.next.key, p.next.value, p.next.next);
       return result;
    }

克隆复杂可变对象的最后一种方法是调用 super.clone，将结果对象中的所有属性设置为其初始状态，然后调用更高级别的方法来重新生成原始对象的状态。 以 HashTable 为例，bucket 属性将被初始化为一个新的 bucket 数组，并且 put(key, value) 方法（未示出）被调用用于被克隆的哈希表中的键值映射。 这种方法通常产生一个简单，合理的优雅 clone 方法，其运行速度不如直接操纵克隆内部的方法快。 虽然这种方法是干净的，但它与整个 Cloneable 体系结构是对立的，因为它会盲目地重写构成体系结构基础的逐个属性对象复制。

　　与构造方法一样，clone 方法绝对不可以在构建过程中，调用一个可以重写的方法（详见第 19 条）。如果 clone 方法调用一个在子类中重写的方法，则在子类有机会在克隆中修复它的状态之前执行该方法，很可能导致克隆和原始对象的损坏。因此，我们在前面讨论的 put(key, value) 方法应该时 final 或 private 修饰的。（如果时 private 修饰，那么大概是一个非 final 公共方法的辅助方法）。

　　Object 类的 clone 方法被声明为抛出 CloneNotSupportedException 异常，但重写方法时不需要。 公共 clone 方法应该省略 throws 子句，因为不抛出检查时异常的方法更容易使用（详见第 71 条）。

　　在为继承设计一个类时（详见第 19 条），通常有两种选择，但无论选择哪一种，都不应该实现 Clonable 接口。你可以选择通过实现正确运行的受保护的 clone 方法来模仿 Object 的行为，该方法声明为抛出 CloneNotSupportedException 异常。 这给了子类实现 Cloneable 接口的自由，就像直接继承 Object 一样。 或者，可以选择不实现工作的 clone 方法，并通过提供以下简并 clone 实现来阻止子类实现它：

    // clone method for extendable class not supporting Cloneable
    @Override
    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }

还有一个值得注意的细节。 如果你编写一个实现了 Cloneable 的线程安全的类，记得它的 clone 方法必须和其他方法一样（详见第 78 条）需要正确的同步。 Object 类的 clone 方法是不同步的，所以即使它的实现是令人满意的，也可能需要编写一个返回 super.clone() 的同步 clone 方法。

　　回顾一下，实现 Cloneable 的所有类应该重写公共 clone 方法，而这个方法的返回类型是类本身。 这个方法应该首先调用 super.clone，然后修复任何需要修复的属性。 通常，这意味着复制任何包含内部「深层结构」的可变对象，并用指向新对象的引用来代替原来指向这些对象的引用。虽然这些内部拷贝通常可以通过递归调用 clone 来实现，但这并不总是最好的方法。 如果类只包含基本类型或对不可变对象的引用，那么很可能是没有属性需要修复的情况。 这个规则也有例外。 例如，表示序列号或其他唯一 ID 的属性即使是基本类型的或不可变的，也需要被修正。

　　这么复杂是否真的有必要？很少。 如果你继承一个已经实现了 Cloneable 接口的类，你别无选择，只能实现一个行为良好的 clone 方法。 否则，通常你最好提供另一种对象复制方法。 对象复制更好的方法是提供一个复制构造方法或复制工厂。 复制构造方法接受参数，其类型为包含此构造方法的类，例如：

    // Copy constructor
    public Yum(Yum yum) { ... };

复制工厂类似于复制构造方法的静态工厂：

    // Copy factory
    public static Yum newInstance(Yum yum) { ... };

复制构造方法及其静态工厂变体与 Cloneable/clone 相比有许多优点：它们不依赖风险很大的语言外的对象创建机制；不要求遵守那些不太明确的惯例；不会与 final 属性的正确使用相冲突; 不会抛出不必要的检查异常; 而且不需要类型转换。

　　此外，复制构造方法或复制工厂可以接受类型为该类实现的接口的参数。 例如，按照惯例，所有通用集合实现都提供了一个构造方法，其参数的类型为 Collection 或 Map。 基于接口的复制构造方法和复制工厂（更适当地称为转换构造方法和转换工厂）允许客户端选择复制的实现类型，而不是强制客户端接受原始实现类型。 例如，假设你有一个 HashSet，并且你想把它复制为一个 TreeSet。 clone 方法不能提供这种功能，但使用转换构造方法很容易：new TreeSet<>(s)。

　　考虑到与 Cloneable 接口相关的所有问题，新的接口不应该继承它，新的可扩展类不应该实现它。 虽然实现 Cloneable 接口对于 final 类没有什么危害，但应该将其视为性能优化的角度，仅在极少数情况下才是合理的（详见第 67 条）。 通常，复制功能最好由构造方法或工厂提供。 这个规则的一个明显的例外是数组，它最好用 clone 方法复制。

# 考虑实现 Comparable 接口

与本章讨论的其他方法不同，compareTo 方法并没有在 Object 类中声明。 相反，它是 Comparable 接口中的唯一方法。 它与 Object 类的 equals 方法在性质上是相似的，除了它允许在简单的相等比较之外的顺序比较，它是泛型的。 通过实现 Comparable 接口，一个类表明它的实例有一个自然顺序（natural ordering）。 对实现 Comparable 接口的对象数组排序非常简单，如下所示：

    Arrays.sort(a);

它很容易查找，计算极端数值，以及维护 Comparable 对象集合的自动排序。例如，在下面的代码中，依赖于 String 类实现了 Comparable 接口，去除命令行参数输入重复的字符串，并按照字母顺序排序：

    public class WordList {

        public static void main(String[] args) {
            Set<String> s = new TreeSet<>();
            Collections.addAll(s, args);
            System.out.println(s);
        }
    }

通过实现 Comparable 接口，可以让你的类与所有依赖此接口的通用算法和集合实现进行互操作。 只需少量的努力就可以获得巨大的能量。 几乎 Java 平台类库中的所有值类以及所有枚举类型（详见第 34 条）都实现了 Comparable 接口。 如果你正在编写具有明显自然顺序（如字母顺序，数字顺序或时间顺序）的值类，则应该实现 Comparable 接口：

    public interface Comparable<T> {
        int compareTo(T t);
    }

compareTo 方法的通用约定与 equals 相似：

　　将此对象与指定的对象按照排序进行比较。 返回值可能为负整数，零或正整数，因为此对象对应小于，等于或大于指定的对象。 如果指定对象的类型与此对象不能进行比较，则引发 ClassCastException 异常。

　　下面的描述中，符号 sgn(expression) 表示数学中的 signum 函数，它根据表达式的值为负数、零、正数，对应返回-1、0 和 1。

实现类必须确保所有 x 和 y 都满足 sgn(x.compareTo(y)) == -sgn(y. compareTo(x))。 （这意味着当且仅当 y.compareTo(x) 抛出异常时，x.compareTo(y) 必须抛出异常。）
实现类还必须确保该关系是可传递的：(x. compareTo(y) > 0 && y.compareTo(z) > 0) 意味着 x.compareTo(z) > 0。
最后，对于所有的 z，实现类必须确保 x.compareTo(y) == 0 意味着 sgn(x.compareTo(z)) == sgn(y.compareTo(z))。
强烈推荐 (x.compareTo(y) == 0) == (x.equals(y))，但不是必需的。 一般来说，任何实现了 Comparable 接口的类违反了这个条件都应该清楚地说明这个事实。 推荐的语言是「注意：这个类有一个自然顺序，与 equals 不一致」。
　　与 equals 方法一样，不要被上述约定的数学特性所退缩。这个约定并不像看起来那么复杂。 与 equals 方法不同，equals 方法在所有对象上施加了全局等价关系，compareTo 不必跨越不同类型的对象：当遇到不同类型的对象时，compareTo 被允许抛出 ClassCastException 异常。 通常，这正是它所做的。 约定确实允许进行不同类型间比较，这种比较通常在由被比较的对象实现的接口中定义。

　　正如一个违反 hashCode 约定的类可能会破坏依赖于哈希的其他类一样，违反 compareTo 约定的类可能会破坏依赖于比较的其他类。 依赖于比较的类，包括排序后的集合 TreeSet 和 TreeMap 类，以及包含搜索和排序算法的实用程序类 Collections 和 Arrays。

　　我们来看看 compareTo 约定的规定。 第一条规定，如果反转两个对象引用之间的比较方向，则会发生预期的事情：如果第一个对象小于第二个对象，那么第二个对象必须大于第一个; 如果第一个对象等于第二个，那么第二个对象必须等于第一个; 如果第一个对象大于第二个，那么第二个必须小于第一个。 第二项约定说，如果一个对象大于第二个对象，而第二个对象大于第三个对象，则第一个对象必须大于第三个对象。 最后一条规定，所有比较相等的对象与任何其他对象相比，都必须得到相同的结果。

　　这三条规定的一个结果是，compareTo 方法所实施的平等测试必须遵守 equals 方法约定所施加的相同限制：自反性，对称性和传递性。 因此，同样需要注意的是：除非你愿意放弃面向对象抽象（详见第 10 条）的好处，否则无法在保留 compareTo 约定的情况下使用新的值组件继承可实例化的类。 同样的解决方法也适用。 如果要将值组件添加到实现 Comparable 的类中，请不要继承它；编写一个包含第一个类实例的不相关的类。 然后提供一个返回包含实例的「视图”方法。 这使你可以在包含类上实现任何 compareTo 方法，同时客户端在需要时，把包含类的实例视同以一个类的实例。

　　compareTo 约定的最后一段是一个强烈的建议，而不是一个真正的要求，只是声明 compareTo 方法施加的相等性测试，通常应该返回与 equals 方法相同的结果。 如果遵守这个约定，则 compareTo 方法施加的顺序被认为与 equals 相一致。 如果违反，顺序关系被认为与 equals 不一致。 其 compareTo 方法施加与 equals 不一致顺序关系的类仍然有效，但包含该类元素的有序集合可能不服从相应集合接口（Collection，Set 或 Map）的一般约定。 这是因为这些接口的通用约定是用 equals 方法定义的，但是排序后的集合使用 compareTo 强加的相等性测试来代替 equals。 如果发生这种情况，虽然不是一场灾难，但仍是一件值得注意的事情。

　　例如，考虑 BigDecimal 类，其 compareTo 方法与 equals 不一致。 如果你创建一个空的 HashSet 实例，然后添加 new BigDecimal("1.0") 和 new BigDecimal("1.00")，则该集合将包含两个元素，因为与 equals 方法进行比较时，添加到集合的两个 BigDecimal 实例是不相等的。 但是，如果使用 TreeSet 而不是 HashSet 执行相同的过程，则该集合将只包含一个元素，因为使用 compareTo 方法进行比较时，两个 BigDecimal 实例是相等的。 （有关详细信息，请参阅 BigDecimal 文档。）

　　编写 compareTo 方法与编写 equals 方法类似，但是有一些关键的区别。 因为 Comparable 接口是参数化的，compareTo 方法是静态类型的，所以你不需要输入检查或者转换它的参数。 如果参数是错误的类型，那么调用将不会编译。 如果参数为 null，则调用应该抛出一个 NullPointerException 异常，并且一旦该方法尝试访问其成员，它就会立即抛出这个异常。

　　在 compareTo 方法中，比较属性的顺序而不是相等。 要比较对象引用属性，请递归调用 compareTo 方法。 如果一个属性没有实现 Comparable，或者你需要一个非标准的顺序，那么使用 Comparator 接口。 可以编写自己的比较器或使用现有的比较器，如在条目 10 中的 CaseInsensitiveString 类的 compareTo 方法中：

    // Single-field Comparable with object reference field
    public final class CaseInsensitiveString
            implements Comparable<CaseInsensitiveString> {
        public int compareTo(CaseInsensitiveString cis) {
            return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
        }
        ... // Remainder omitted
    }

请注意，CaseInsensitiveString 类实现了 Comparable<CaseInsensitiveString> 接口。 这意味着 CaseInsensitiveString 引用只能与另一个 CaseInsensitiveString 引用进行比较。 当声明一个类来实现 Comparable 接口时，这是正常模式。

　　在本书第二版中，曾经推荐如果比较整型基本类型的属性，使用关系运算符「<」和 「>」，对于浮点类型基本类型的属性，使用 Double.compare 和 Float.compare 静态方法。在 Java 7 中，静态比较方法被添加到 Java 的所有包装类中。 在 compareTo 方法中使用关系运算符「<」和「>」是冗长且容易出错的，不再推荐。

　　如果一个类有多个重要的属性，那么比较他们的顺序是至关重要的。 从最重要的属性开始，逐步比较所有的重要属性。 如果比较结果不是零（零表示相等），则表示比较完成; 只是返回结果。 如果最重要的字段是相等的，比较下一个重要的属性，依此类推，直到找到不相等的属性或比较剩余不那么重要的属性。 以下是条目 11 中 PhoneNumber 类的 compareTo 方法，演示了这种方法：

    // Multiple-field `Comparable` with primitive fields
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);
        if (result == 0) {
            result = Short.compare(prefix, pn.prefix);
            if (result == 0)
                result = Short.compare(lineNum, pn.lineNum);
        }
        return result;
    }

在 Java 8 中 Comparator 接口提供了一系列比较器方法，可以使比较器流畅地构建。 这些比较器可以用来实现 compareTo 方法，就像 Comparable 接口所要求的那样。 许多程序员更喜欢这种方法的简洁性，尽管它的性能并不出众：在我的机器上排序 PhoneNumber 实例的数组速度慢了大约 10％。 在使用这种方法时，考虑使用 Java 的静态导入，以便可以通过其简单名称来引用比较器静态方法，以使其清晰简洁。 以下是 PhoneNumber 的 compareTo 方法的使用方法：


    // Comparable with comparator construction methods
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
              .thenComparingInt(pn -> pn.prefix)
              .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }

此实现在类初始化时构建比较器，使用两个比较器构建方法。第一个是 comparingInt 方法。它是一个静态方法，它使用一个 ***键提取器函数式接口*** (key extractor function）作为参数，将对象引用映射为 int 类型的键，并返回一个根据该键排序的实例的比较器。在前面的示例中，comparingInt 方法使用 lambda 表达式，它从 PhoneNumber 中提取区域代码，并返回一个 Comparator<PhoneNumber>，根据它们的区域代码来排序电话号码。注意，lambda 表达式显式指定了其输入参数的类型 (PhoneNumber pn)。事实证明，在这种情况下，Java 的类型推断功能不够强大，无法自行判断类型，因此我们不得不帮助它以使程序编译。

　　如果两个电话号码实例具有相同的区号，则需要进一步细化比较，这正是第二个比较器构建方法，即 thenComparingInt 方法做的。 它是 Comparator 上的一个实例方法，接受一个 int 类型 ***键提取器函数式接口*** key extractor function）作为参数，并返回一个比较器，该比较器首先应用原始比较器，然后使用提取的键来打破连接。 你可以按照喜欢的方式多次调用 thenComparingInt 方法，从而产生一个字典顺序。 在上面的例子中，我们将两个调用叠加到 thenComparingInt，产生一个排序，它的二级键是 prefix，而其三级键是 lineNum。 请注意，我们不必指定传递给 thenComparingInt 的任何一个调用的 ***键提取器函数式接口*** 的参数类型：Java 的类型推断足够聪明，可以自己推断出参数的类型。

　　Comparator 类具有完整的构建方法。对于 long 和 double 基本类型，也有对应的类似于 comparingInt 和 thenComparingInt 的方法，int 版本的方法也可以应用于取值范围小于 int 的类型上，如 short 类型，如 PhoneNumber 实例中所示。对于 double 版本的方法也可以用在 float 类型上。这提供了所有 Java 的基本数字类型的覆盖。

　　也有对象引用类型的比较器构建方法。静态方法 comparing 有两个重载方式。第一个方法使用 ***键提取器函数式接口*** 并按键的自然顺序。第二种方法是 ***键提取器函数式接口*** 和比较器，用于键的排序。thenComparing 方法有三种重载。第一个重载只需要一个比较器，并使用它来提供一个二级排序。第二次重载只需要一个 ***键提取器函数式接口*** ，并使用键的自然顺序作为二级排序。最后的重载方法同时使用一个 ***键提取器函数式接口*** 和一个比较器来用在提取的键上。

　　有时，你可能会看到 compareTo 或 compare 方法依赖于两个值之间的差值，如果第一个值小于第二个值，则为负；如果两个值相等则为零，如果第一个值大于，则为正值。这是一个例子：

    // BROKEN difference-based comparator - violates transitivity!

    static Comparator<Object> hashCodeOrder = new Comparator<>() {
        public int compare(Object o1, Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
    };

不要使用这种技术！它可能会导致整数最大长度溢出和 IEEE 754 浮点运算失真的危险[JLS 15.20.1,15.21.1]。 此外，由此产生的方法不可能比使用上述技术编写的方法快得多。 使用静态 compare 方法：

    // Comparator based on static compare method
    static Comparator<Object> hashCodeOrder = new Comparator<>() {
        public int compare(Object o1, Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
    };

或者使用 Comparator 的构建方法：

    // Comparator based on Comparator construction method
    static Comparator<Object> hashCodeOrder =
            Comparator.comparingInt(o -> o.hashCode());

***总而言之，无论何时实现具有合理排序的值类，你都应该让该类实现 Comparable 接口，以便在基于比较的集合中轻松对其实例进行排序，搜索和使用。 比较
compareTo 方法的实现中的字段值时，请避免使用「<」和「>」运算符。 相反，使用包装类中的静态 compare 方法或 Comparator 接口中的构建方法***

# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/13.%20%E8%B0%A8%E6%85%8E%E5%9C%B0%E9%87%8D%E5%86%99%20clone%20%E6%96%B9%E6%B3%95
