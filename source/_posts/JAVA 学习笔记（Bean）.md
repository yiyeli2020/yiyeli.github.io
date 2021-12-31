---
title: JAVA 学习笔记（Bean）

date: 2021-12-30 15:12:12  

categories: 2021年12月

tags: [Java, Spring]

---

Bean的概念

<!-- more -->

[TOC]

# 什么是Bean[^1][^2][^3]

你
一种规范，表达实体和信息的规范，便于封装重用。

Bean的中文含义是“豆子”，顾名思义JavaBean是一段Java小程序。JavaBean实际上是指一种特殊的Java类，它通常用来实现一些比较常用的简单功能，并可以很容易的被重用或者是插入其他应用程序中去。所有遵循一定编程原则的Java类都可以被称作JavaBean。


# 特点
1、若干private实例字段；

2、提供一个默认的无参构造函数；

3、可能有一系列的 getter 或 setter 方法；

4、需要被序列化并且实现serializable接口；

# JavaBean的作用
JavaBean主要用来传递数据，即把一组数据组合成一个JavaBean便于传输。此外，JavaBean可以方便地被IDE工具分析，生成读写属性的代码，主要用在图形界面的可视化设计中。

保持向后兼容性



[^1]:https://www.zhihu.com/question/19773379

[^2]:https://www.runoob.com/jsp/jsp-javabean.

[^3]:https://www.liaoxuefeng.com/wiki/1252599548343744/1260474416351680