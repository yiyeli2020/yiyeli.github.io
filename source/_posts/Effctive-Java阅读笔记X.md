title: Effctive-Java阅读笔记VII
date: 2020-1-16 14:29:12
categories: 2020年1月
tags: [Java]

---

第四章阅读：类和接口

19.要么设计继承并提供文档说明，要么禁用继承

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






# 参考资料
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/19.%20%E8%A6%81%E4%B9%88%E8%AE%BE%E8%AE%A1%E7%BB%A7%E6%89%BF%E5%B9%B6%E6%8F%90%E4%BE%9B%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E%EF%BC%8C%E8%A6%81%E4%B9%88%E7%A6%81%E7%94%A8%E7%BB%A7%E6%89%BF
