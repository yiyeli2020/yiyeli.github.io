---
title: Scala学习笔记III
date: 2019-12-23 15:08:12
categories: 2019年12月
tags: [Scala]

---

Scala学习：伴生对象中的apply方法

<!-- more -->

# 单例对象
Scala没有静态方法或字段，可以用object语法定义结构，对象定义了类的单个实例。
对象的构造器在该对象第一次使用时被调用。
不能提供构造器参数。
作为存放工具函数或常量的地方。
高效地共享单个不可变实例。

# 伴生对象
在Scala中，可通过类和类同名的“伴生”对象来达到静态方法的目的。
类和它的伴生对象可以相互访问私有特性，它们必须存在于同一个源文件中

示例：

  class Account {

    val id = Account.newUniqueNumber()

    private var balance = 0.0

    def deposit(amount: Double): Double = {
      balance += amount
      balance
    }

    def nowBalance = balance;

  }

  object Account {
    private var lastNumber = 0

    private def newUniqueNumber() = {
      lastNumber += 1
      lastNumber
    }
  }

  object Main {
    def main(args: Array[String]): Unit = {
      val account = new Account
      println(account.deposit(1))
      println("=" * 10)

      val account1 = new Account
      println(account1.id)
      println(account1.deposit(10))
      println("=" * 10)

      println("a " + account.nowBalance + "; b " + account1.nowBalance)
    }
  }

执行结果

  1
  1.0
  ==========
  2
  10.0
  ==========
  a 1.0; b 10.0

# apply方法
一般在伴生对象中定义apply方法
常用于初始化操作或创建单例对象
在生成这个类的对象时，就省去了new关键字
在遇到Object(参数1，参数2，……，参数n)时就会自动调用apply()方法
    class Student private (val studentID: Int, val name: String){

      override def toString: String = {
        "studentID " + studentID + " name " + name
      }
    }

    object Student {
      private var studentID = 0

      private def newSno = {
        studentID += 1
        studentID
      }

      def apply(name: String): Student = {

        println("call apply method...")
        new Student(newSno, name)
      }
    }

    object StudentMain extends App {
      // no new
      val student1 = Student("YiyeLi")
      println(student1.toString)

      println("*" * 10)
      val student2 = Student("YiyeLi")
      println(student2.toString)
    }
执行结果：

    call apply method...
    studentID 1 name YiyeLi
    **********
    call apply method...
    studentID 2 name YiyeLi

再举一个例子，一个trait可以看作是一个Java接口。我们使用一个伴生类Shape和一个伴生对象Shape，作为一个工厂。

    trait Shape {
      def area :Double
    }
    object Shape {
      private class Circle(radius: Double) extends Shape{
        override val area = 3.14*radius*radius
      }
      private class Rectangle (height: Double, length: Double)extends Shape{
        override val area = height * length
      }
      private class cube (x:Double,y:Double,z:Double) extends Shape{
        override val area= x * y * z
      }
      def apply(height :Double , length :Double ) : Shape = new Rectangle(height,length)
      def apply(radius :Double) : Shape = new Circle(radius)
      def apply(x:Double,y:Double,z:Double): Shape = new cube(x,y,z)
    }

    object Main extends App {
      val circle = Shape(2)
      println(circle.area)
      val rectangle = Shape(2,3)
      println(rectangle.area)
      val cube=Shape(3,4,5)
      println(cube.area)
    }

执行结果：

  12.56
  6.0
  60.0

# main函数
Scala程序必须从一个对象的main方法开始
有两种方法定义
  // 执行println语句
  object Main {
    def main(args: Array[String]): Unit = {

      println("=" * 10)

    }
  }

  // 扩展App特质
  object Main extends App {
    println("=" * 10)
  }



# 参考资料：
【1】https://zhuanlan.zhihu.com/p/32834063
【2】https://www.w3cschool.cn/scala/scala-companion-objects.html
