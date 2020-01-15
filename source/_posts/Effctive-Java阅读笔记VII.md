title: Effctive-Java阅读笔记VII
date: 2020-1-15 10:48:12
categories: 2020年1月
tags: [Java]

---

第三章阅读：Methods Common to All Objects
13.谨慎地重写 clone 方法

<!-- more -->

# 谨慎地重写 clone 方法

TODO：第一段没有读懂，留待看过20/65条后再回头来读。

Cloneable 接口的目的是作为一个 mixin 接口 （详见第 20 条），公布这样的类允许克隆。不幸的是，它没有达到这个目的。它的主要缺点是缺少 clone 方法，而 Object 的 clone 方法是受保护的。你不能，不借助反射 （详见第 65 条），仅仅因为它实现了 Cloneable 接口，就调用对象上的 clone 方法。即使是反射调用也可能失败，因为不能保证对象具有可访问的 clone 方法。尽管存在许多缺陷，该机制在合理的范围内使用，所以理解它是值得的。这个条目告诉你如何实现一个行为良好的 clone 方法，在适当的时候讨论这个方法，并提出替代方案。

　　既然 Cloneable 接口不包含任何方法，那它用来做什么？ 它决定了 Object 的受保护的 clone 方法实现的行为：如果一个类实现了 Cloneable 接口，那么 Object 的 clone 方法将返回该对象的逐个属性（field-by-field）拷贝；否则会抛出 CloneNotSupportedException 异常。这是一个非常反常的接口使用，而不应该被效仿。 通常情况下，实现一个接口用来表示可以为客户做什么。但对于 Cloneable 接口，它会修改父类上受保护方法的行为。

　　虽然规范并没有说明，但在实践中，实现 Cloneable 接口的类希望提供一个正常运行的公共 clone 方法。为了实现这一目标，该类及其所有父类必须遵循一个复杂的、不可执行的、稀疏的文档协议。由此产生的机制是脆弱的、危险的和不受语言影响的（extralinguistic）：它创建对象而不需要调用构造方法。

　　clone 方法的通用规范很薄弱的。 以下内容是从 Object 规范中复制出来的：

　　创建并返回此对象的副本。 「复制（copy）」的确切含义可能取决于对象的类。 一般意图是，对于任何对象 x，表达式 x.clone() != x 返回 true，并且 x.clone().getClass() == x.getClass() 也返回 true，但它们不是绝对的要求，但通常情况下，x.clone().equals(x) 返回 true，当然这个要求也不是绝对的。

　　根据约定，这个方法返回的对象应该通过调用 super.clone 方法获得的。 如果一个类和它的所有父类（Object 除外）都遵守这个约定，情况就是如此，x.clone().getClass() == x.getClass()。

　　根据约定，返回的对象应该独立于被克隆的对象。 为了实现这种独立性，在返回对象之前，可能需要修改由 super.clone 返回的对象的一个或多个属性。

　　这种机制与构造方法链（chaining）很相似，只是它没有被强制执行；如果一个类的 clone 方法返回一个通过调用构造方法获得而不是通过调用 super.clone 的实例，那么编译器不会抱怨，但是如果一个类的子类调用了 super.clone，那么返回的对象包含错误的类，从而阻止子类 clone 方法正常执行。如果一个类重写的 clone 方法是有 final 修饰的，那么这个约定可以被安全地忽略，因为子类不需要担心。但是，如果一个 final 类有一个不调用 super.clone 的 clone 方法，那么这个类没有理由实现 Cloneable 接口，因为它不依赖于 Object 的 clone 实现的行为。

　　假设你希望在一个类中实现 Cloneable 接口，它的父类提供了一个行为良好的 clone 方法。首先调用 super.clone。 得到的对象将是原始的完全功能的复制品。 在你的类中声明的任何属性将具有与原始属性相同的值。 如果每个属性包含原始值或对不可变对象的引用，则返回的对象可能正是你所需要的，在这种情况下，不需要进一步的处理。 例如，对于条目 11 中的 PhoneNumber 类，情况就是这样，但是请注意，不可变类永远不应该提供 clone 方法，因为这只会浪费复制。 有了这个警告，以下是 PhoneNumber 类的 clone 方法：



# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/13.%20%E8%B0%A8%E6%85%8E%E5%9C%B0%E9%87%8D%E5%86%99%20clone%20%E6%96%B9%E6%B3%95
