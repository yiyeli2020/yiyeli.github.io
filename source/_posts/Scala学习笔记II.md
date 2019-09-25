---
title: Scala学习笔记II
date: 2019-9-25 15:17:12
categories: 2019年9月
tags: [Scala]

---

Scala学习：=>语法糖

<!-- more -->


# Scala的“=>”符号简介

Scala中的=>符号可以看做是创建函数实例的语法糖。例如：A => T，A,B => T表示一个函数的输入参数类型是“A”，“A,B”，返回值类型是T。请看下面这个实例：

    scala> val f: Int => String = myInt => "The value of myInt is: " + myInt.toString()
    f: Int => String = <function1>

    scala> println(f(3))
    The value of myInt is: 3

另外，() => T表示函数输入参数为空，而A => Unit则表示函数没有返回值。

    object HelloScala {

      def main(args: Array[String])  {

        /**
          * 首先定义函数d,参数类型是Int=>Int的函数,返回值根据上下文推算是Int。
          * 返回值: 发现没有,返回值是x(2),它调用了传入函数。结果自然就是6了。
          */
        def d(x: (Int) => Int) = x(2);
        println(d((x: Int) => x * 3));

        // 继续增加难度,设置2个值。仔细看变化,你会明白的
        def c(x: (Int, Int) => Int) = x(2, 3);
        println(c((x: Int, y: Int) => x * y * 3));

        // 加深难度,b第一次调用返回函数(y: Int) => x + y,在一次调用返回结果。
        // 相关文章参考快学scala 第十二章 高阶函数 145页
        val b = (x: Int) => (y: Int) => x + y;
        println(b.apply(5).apply(6));

      }
    }

# 参考资料：
【1】 https://www.orchome.com/401
