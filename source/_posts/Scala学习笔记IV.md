---
title: Scala学习笔记IV
date: 2019-12-23 18:08:12
categories: 2019年12月
tags: [Scala]

---

数据库连接和事务处理

<!-- more -->

# Session 管理

现在有了一个数据库对象可以打开一个数据库（Slick 函数库封装了一个 Session 对象）

Database 的 withSession 方法，创建一个 Session 对象，它可以传递给一个函数，函数返回时自动关闭这个 Session 对象，如果你使用连接池，关闭 Session 对象，自动将连接退回连接池。

    val query = for (c <- coffees) yield c.name
    val result = db.withSession {
        session =>
        query.list()( session )
    }

可以看到，我们可以在 withSession 之外定义查询，只有在实际执行查询时才需要一个 Session 对象，要注意的是 Session 的缺省模式为自动提交（auto-commit )模式。每个数据库指令（比如 insert )都自动提交给数据库。 如果需要将几个指令作为一个整体，那么就需要使用事务处理（Transaction） 上面的例子，我们在执行查询时，明确指明了 session 对象，你可以使用隐含对象来避免这种情况，比如：

    val query = for (c <- coffees) yield c.name
    val result = db.withSession {
        implicit session =>
        query.list // <- takes session implicitly
    }
    // query.list // <- would not compile, no implicit value of type Session



# 参考资料：
【1】http://wiki.jikexueyuan.com/project/slick-guide/database-and-transaction-processing.html
