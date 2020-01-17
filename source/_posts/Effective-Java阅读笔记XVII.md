title: Effective-Java阅读笔记XVII
date: 2020-1-17 02:33:12
categories: 2020年1月
tags: [Java]

---

第五章阅读： 泛型
33.优先考虑类型安全的异构容器


<!-- more -->

　　泛型的常见用法包括集合，如 Set<E> 和 Map<K，V> 和单个元素容器，如 ThreadLocal<T> 和 AtomicReference<T>。 在所有这些用途中，它都是参数化的容器。 这限制了每个容器只能有固定数量的类型参数。 通常这正是你想要的。 一个 Set 有单一的类型参数，表示它的元素类型; 一个 Map 有两个，代表它的键和值的类型；等等。

　　然而有时候，你需要更多的灵活性。 例如，数据库一行记录可以具有任意多列，并且能够以类型安全的方式访问它们是很好的。 幸运的是，有一个简单的方法可以达到这个效果。 这个想法是参数化键（key）而不是容器。 然后将参数化的键提交给容器以插入或检索值。 泛型类型系统用于保证值的类型与其键一致。

　　作为这种方法的一个简单示例，请考虑一个 Favorites 类，它允许其客户端保存和检索任意多种类型的 favorite 实例。 该类型的 Class 对象将扮演参数化键的一部分。其原因是这 Class 类是泛型的。 类的类型从字面上来说不是简单的 Class，而是 Class<T>。 例如，String.class 的类型为 Class<String>，Integer.class 的类型为 Class<Integer>。 当在方法中传递字面类传递编译时和运行时类型信息时，它被称为类型令牌（type token）[Bracha04]。

　　Favorites 类的 API 很简单。 它看起来就像一个简单 Map 类，除了该键是参数化的以外。 客户端在设置和获取 favorites 实例时呈现一个 Class 对象。 如下是 API：




# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/33.%20%E4%BC%98%E5%85%88%E8%80%83%E8%99%91%E7%B1%BB%E5%9E%8B%E5%AE%89%E5%85%A8%E7%9A%84%E5%BC%82%E6%9E%84%E5%AE%B9%E5%99%A8
