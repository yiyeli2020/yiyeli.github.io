title: Effective-Java阅读笔记XI
date: 2020-1-16 16:23:12
categories: 2020年1月
tags: [Java]

---

第四章阅读：类和接口

21.为后代设计接口
应该避免使用默认方法向现有的接口添加新的方法，除非这个需要是关键的。在默认方法的情况下，接口的现有实现类可以在没有错误或警告的情况下编译，但在运行时会失败。
22.接口仅用来定义类型
接口只能用于定义类型。 它们不应该只是被用于导出常量。

<!-- more -->

# 为后代设计接口

　　在 Java 8 之前，不可能在不破坏现有实现的情况下为接口添加方法。 如果向接口添加了一个新方法，现有的实现通常会缺少该方法，从而导致编译时错误。 在 Java 8 中，添加了默认方法（default method）构造[JLS 9.4]，目的是允许将方法添加到现有的接口。 但是增加新的方法到现有的接口是充满风险的。

　　默认方法的声明包含一个默认实现，该方法允许实现接口的类直接使用，而不必实现默认方法。 虽然在 Java 中添加默认方法可以将方法添加到现有接口，但不能保证这些方法可以在所有已有的实现中使用。 默认的方法被「注入（injected）」到现有的实现中，没有经过实现类的知道或同意。 在 Java 8 之前，这些实现是用默认的接口编写的，它们的接口永远不会获得任何新的方法。

　　许多新的默认方法被添加到 Java 8 的核心集合接口中，主要是为了方便使用 lambda 表达式（第 6 章）。 Java 类库的默认方法是高质量的通用实现，在大多数情况下，它们工作正常。 但是，编写一个默认方法并不总是可能的，它保留了每个可能的实现的所有不变量。

　　例如，考虑在 Java 8 中添加到 Collection 接口的 removeIf 方法。此方法删除给定布尔方法（或 Predicate 函数式接口）返回 true 的所有元素。默认实现被指定为使用迭代器遍历集合，调用每个元素的谓词，并使用迭代器的 remove 方法删除谓词返回 true 的元素。 据推测，这个声明看起来像这样：默认实现被指定为使用迭代器遍历集合，调用每个元素的 Predicate 函数式接口，并使用迭代器的 remove 方法删除 Predicate 函数式接口返回 true 的元素。 根据推测，这个声明看起来像这样：

    // Default method added to the Collection interface in Java 8
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean result = false;
        for (Iterator<E> it = iterator(); it.hasNext(); ) {
            if (filter.test(it.next())) {
                it.remove();
                result = true;
            }
        }
        return result;
    }

　　这是可能为 removeIf 方法编写的最好的通用实现，但遗憾的是，它在一些实际的 Collection 实现中失败了。 例如，考虑 org.apache.commons.collections4.collection.SynchronizedCollection 方法。 这个类出自 Apache Commons 类库中，与 java.util 包中的静态工厂 Collections.synchronizedCollection 方法返回的类相似。 Apache 版本还提供了使用客户端提供的对象进行锁定的能力，以代替集合。 换句话说，它是一个包装类（条目 18），它们的所有方法在委托给包装集合类之前在一个锁定对象上进行同步。

　　Apache 的 SynchronizedCollection 类仍然在积极维护，但在撰写本文时，并未重写 removeIf 方法。 如果这个类与 Java 8 一起使用，它将继承 removeIf 的默认实现，但实际上不能保持类的基本承诺：自动同步每个方法调用。 默认实现对同步一无所知，并且不能访问包含锁定对象的属性。 如果客户端在另一个线程同时修改集合的情况下调用 SynchronizedCollection 实例上的 removeIf 方法，则可能会导致 ConcurrentModificationException 异常或其他未指定的行为。

　　为了防止在类似的 Java 平台类库实现中发生这种情况，比如 Collections.synchronizedCollection 返回的包级私有的类，JDK 维护者必须重写默认的 removeIf 实现和其他类似的方法来在调用默认实现之前执行必要的同步。 原来不属于 Java 平台的集合实现没有机会与接口更改进行类似的改变，有些还没有这样做。

　　***在默认方法的情况下，接口的现有实现类可以在没有错误或警告的情况下编译，但在运行时会失败。*** 虽然不是非常普遍，但这个问题也不是一个孤立的事件。 在 Java 8 中添加到集合接口的一些方法已知是易受影响的，并且已知一些现有的实现会受到影响。

　　应该避免使用默认方法向现有的接口添加新的方法，除非这个需要是关键的，在这种情况下，你应该仔细考虑，以确定现有的接口实现是否会被默认的方法实现所破坏。然而，默认方法对于在创建接口时提供标准的方法实现非常有用，以减轻实现接口的任务（详见第 20 条）。

　　还值得注意的是，默认方法不是被用来设计，来支持从接口中移除方法或者改变现有方法的签名的目的。在不破坏现有客户端的情况下，这些接口都不可能发生更改。

　　准则是清楚的。 尽管默认方法现在是 Java 平台的一部分，***但是非常悉心地设计接口仍然是非常重要的***。 虽然默认方法可以将方法添加到现有的接口，但这样做有很大的风险。 如果一个接口包含一个小缺陷，可能会永远惹怒用户。 如果一个接口严重缺陷，可能会破坏包含它的 API。

　　因此，在发布之前测试每个新接口是非常重要的。 多个程序员应该以不同的方式实现每个接口。 至少，你应该准备三种不同的实现。 编写多个使用每个新接口的实例来执行各种任务的客户端程序同样重要。 这将大大确保每个接口都能满足其所有的预期用途。 这些步骤将允许你在发布之前发现接口中的缺陷，但仍然可以轻松地修正它们。 虽然在接口被发布后可能会修正一些存在的缺陷，但不要太指望这一点。

# 接口仅用来定义类型

　　当类实现接口时，该接口作为一种类型（type），可以用来引用类的实例。因此，一个类实现了一个接口，因此表明客户端可以如何处理类的实例。为其他目的定义接口是不合适的。

　　一种失败的接口就是所谓的常量接口（constant interface）。 这样的接口不包含任何方法; 它只包含静态 final 属性，每个输出一个常量。 使用这些常量的类实现接口，以避免需要用类名限定常量名。 这里是一个例子：

    // Constant interface antipattern - do not use!
    public interface PhysicalConstants {
        // Avogadro's number (1/mol)
        static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

        // Boltzmann constant (J/K)
        static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

        // Mass of the electron (kg)
        static final double ELECTRON_MASS      = 9.109_383_56e-31;
    }

　　常量接口模式是对接口的糟糕使用。 类在内部使用一些常量，完全属于实现细节。实现一个常量接口会导致这个实现细节泄漏到类的导出 API 中。对类的用户来说，类实现一个常量接口是没有意义的。事实上，它甚至可能使他们感到困惑。更糟糕的是，它代表了一个承诺：如果在将来的版本中修改了类，不再需要使用常量，那么它仍然必须实现接口，以确保二进制兼容性。如果一个非 final 类实现了常量接口，那么它的所有子类的命名空间都会被接口中的常量所污染。

　　Java 平台类库中有多个常量接口，如 java.io.ObjectStreamConstants。 这些接口应该被视为不规范的，不应该被效仿。

　　如果你想导出常量，有几个合理的选择方案。 如果常量与现有的类或接口紧密相关，则应将其添加到该类或接口中。 例如，所有数字基本类型的包装类，如 Integer 和 Double，都会导出 MIN_VALUE 和 MAX_VALUE 常量。 如果常量最好被看作枚举类型的成员，则应该使用枚举类型（详见第 34 条）导出它们。 否则，你应该用一个不可实例化的工具类来导出常量（详见第 4 条）。 下是前面所示的 PhysicalConstants 示例的工具类的版本：

    // Constant utility class
    package com.effectivejava.science;

    public class PhysicalConstants {
      private PhysicalConstants() { }  // Prevents instantiation

      public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
      public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;
      public static final double ELECTRON_MASS    = 9.109_383_56e-31;
    }

　　顺便提一下，请注意在数字文字中使用下划线字符_ 。 从 Java 7 开始，合法的下划线对数字字面量的值没有影响，但是如果使用得当的话可以使它们更容易阅读。 无论是固定的浮点数，如果他们包含五个或更多的连续数字，考虑将下划线添加到数字字面量中。 对于底数为 10 的数字，无论是整型还是浮点型的，都应该用下划线将数字分成三个数字组，表示一千的正负幂。

　　通常，实用工具类要求客户端使用类名来限定常量名，例如 PhysicalConstants.AVOGADROS_NUMBER。 如果大量使用实用工具类导出的常量，则通过使用静态导入来限定具有类名的常量：

    // Use of static import to avoid qualifying constants
    import static com.effectivejava.science.PhysicalConstants.*;

    public class Test {
        double  atoms(double mols) {
            return AVOGADROS_NUMBER * mols;
        }
        ...
        // Many more uses of PhysicalConstants justify static import
    }

　　总之，接口只能用于定义类型。 它们不应该只是被用于导出常量。





# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/21.%20%E4%B8%BA%E5%90%8E%E4%BB%A3%E8%AE%BE%E8%AE%A1%E6%8E%A5%E5%8F%A3
