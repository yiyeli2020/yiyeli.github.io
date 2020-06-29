---
title: Java之TreeSet类

date: 2020-6-29 16:21:12

categories: 2020年6月

tags: [Java,TreeSet]

---

java.util.TreeSet 类实现Set接口。
 
<!-- more -->

以下是关于TreeSet的要点[^1]：

- TreeSet类保证该映射将在升序键顺序，由TreeMap支持。
- 该映射是按照自然排序方法该键类，或在集创建时提供的比较器，这将取决于其构造函数中使用排序。
- 顺序必须是总为了使树到功能属性。
 
这里仅列出几个常用的方法，更多详见[^1]

# 类方法

## E ceiling(E e) 方法

用来返回大于或等于给定的元素的最小元素，如果不存在这样的元素返回null。

## E floor(E e) 方法
此方法返回小于或等于给定的元素的最大元素，如果不存在这样的元素返回null。

[^1]:http://gitbook.net/java/util/java_util_treeset.html