---
title: Scala学习笔记II
date: 2019-9-25 15:17:12
categories: 2019年9月
tags: [Scala]

---

Scala学习：=>语法糖,模式匹配

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

# 模式匹配

    object scala {
        def main(args: Array[String]):Unit={
            println(matchTest(3))
        }
        def matchTest(x:Int):String =x match{
          case 1=>"one"
          case 2=>"two"
          case _=>"many"
        }
    }
match 对应 Java 里的 switch，但是写在选择器表达式之后。即： 选择器 match {备选项}。

match 表达式通过以代码编写的先后次序尝试每个模式来完成计算，只要发现有一个匹配的case，剩下的case不会继续匹配。

接下来我们来看一个不同数据类型的模式匹配：

    object scala {
        def main(args: Array[String]):Unit={
          println(matchTest("two"))
          println(matchTest("test"))
          println(matchTest(1))
          println(matchTest(6))
        }
        def matchTest(x:Any) : Any=x match {
          case 1=>"one"
          case "two"=>2
          case y:Int=>"scala.Int"
          case _=>"many"
        }
    }
输出结果为：

    2
    many
    one
    scala.Int
实例中第一个 case 对应整型数值 1，第二个 case 对应字符串值 two，第三个 case 对应类型模式，用于判断传入的值是否为整型，相比使用isInstanceOf来判断类型，使用模式匹配更好。第四个 case 表示默认的全匹配备选项，即没有找到其他匹配时的匹配项，类似 switch 中的 default。

## 使用样例类
使用了case关键字的类定义就是就是样例类(case classes)，样例类是种特殊的类，经过优化以用于模式匹配。

以下是样例类的简单实例:










# 参考资料：
【1】 https://www.orchome.com/401
