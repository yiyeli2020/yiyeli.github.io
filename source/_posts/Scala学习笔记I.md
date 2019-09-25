---
title: Scala学习笔记I
date: 2019-9-10 11:32:12
categories: 2019年9月
tags: [Scala]

---

Scala学习：集合，柯里化，implicit关键字

<!-- more -->

# Set（集合）


# fold,foldLeft和foldRight区别与联系

从本质上说，fold函数将一种格式的输入数据转化成另外一种格式返回。

    def fold(){
      val a=List(1,2,3,4)
      val res=a.fold(0){
        (z,i) => z+i
      }
      println(res)  //res=10
    }

List中的fold方法需要输入两个参数：初始值以及一个函数。输入的函数也需要输入两个参数：累加值和当前item的索引。那么上面的代码片段发生了什么事？

代码开始运行的时候，初始值0作为第一个参数传进到fold函数中，list中的第一个item作为第二个参数传进fold函数中。

1、fold函数开始对传进的两个参数进行计算，在本例中，仅仅是做加法计算，然后返回计算的值；

2、Fold函数然后将上一步返回的值作为输入函数的第一个参数，并且把list中的下一个item作为第二个参数传进继续计算，同样返回计算的值；

3、第2步将重复计算，直到list中的所有元素都被遍历之后，返回最后的计算值，整个过程结束；

4、这虽然是一个简单的例子，让我们来看看一些比较有用的东西。早在后面将会介绍foldLeft函数，并解释它和fold之间的区别，目前，你只需要想象foldLeft函数和fold函数运行过程一样。


# Scala 函数柯里化(Currying)
柯里化(Currying)指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数为参数的函数。

## 实例
首先我们定义一个函数:

    def add(x:Int,y:Int)=x+y

那么我们应用的时候，应该是这样用：add(1,2)

现在我们把这个函数变一下形：

    def add(x:Int)(y:Int) = x + y
那么我们应用的时候，应该是这样用：add(1)(2),最后结果都一样是3，这种方式（过程）就叫柯里化。

## 实现过程

add(1)(2) 实际上是依次调用两个普通函数（非柯里化函数），第一次调用使用一个参数 x，返回一个函数类型的值，第二次使用参数y调用这个函数类型的值。

实质上最先演变成这样一个方法：

    def add(x:Int)=(y:Int)=>x+y
那么这个函数是什么意思呢？ 接收一个x为参数，返回一个匿名函数，该匿名函数的定义是：接收一个Int型参数y，函数体为x+y。现在我们来对这个方法进行调用。

    val result = add(1)
返回一个result，那result的值应该是一个匿名函数：(y:Int)=>1+y

所以为了得到结果，我们继续调用result。

    val sum = result(2)

最后打印出来的结果就是3。

# Implicit详解

在 Scala 中的 implicit 定义指编译器在需要修复类型匹配时可以用来自动插入的定义。比如说，如果 x+y 类型不匹配，那么编译器可能试着使用 convert(x) + y， 其中 convert 由某个 implicit 定义的，这有点类似一个整数和一个浮点数相加，编译器可以自动把整数转换为浮点数。Scala 的 implicit 定义是对这种情况的一个推广，你可以定义一个类型在需要时，如何自动转换成另外一种类型。

Scala 的 implicit 定义符合下面一些规则：

## 标记规则
只有哪些使用 implicit 关键字的定义才是可以使用的隐式定义。关键字 implicit 用来标记一个隐式定义。编译器才可以选择它作为隐式变化的候选项。你可以使用 implicit 来标记任意变量，函数或是对象。

例如下面为一个隐式函数定义：

    implicit def intToString(x:Int) : x.toString
编译器只有在 convert 被标记成 implicit 才会将 x + y 改成convert(x) + y 。当然这是在 x + y 类型不匹配时。

## 范围规则

编译器在选择备选 implicit 定义时，只会选取当前作用域的定义，比如说编译器不会去调用 someVariable.convert。如果你需要使用 someVariable.convert，你必须把 someVarible 引入到当前作用域。也就是说编译器在选择备选 implicit 时，只有当 convert 是当前作用域下单个标志符时才会作为备选 implicit。比如说，对于一个函数库来说，在一个 Preamble 对象中定义一些常用的隐式类型转换非常常见，因此需要使用 Preamble 的代码可以使用 “import Preamble._”  把这些 implicit 定义引入到当前作用域才可以。

这个规则有一个例外，编译器也会在类的伙伴对象定义中查找所需的 implicit 定义。例如下面的定义：

    object Dollar {
        implicit def dollarToEuro(x:Dollar):Euro = ...
        ...
    }
    class Dollar {
       ...
    }
如果在 class Dollar 的方法有需要 Euro 类型，但输入数据使用的是 Dollar，编译器会在其伙伴对象 object Dollar 查找所需的隐式类型转换，本例定义一个从 Dollar 到 Euro 的 implicit 定义可以使用。

## 一次规则
编译器在需要使用 implicit 定义时，只会试图转换一次，也就是编译器永远不会把 x + y 改写成 convert1(convert2(x)) + y。

## 优先规则
编译器不会在 x+y 已经是合法的情况下去调用 implicit 规则。

## 命名规则
你可以为 implicit 定义任意的名称。通常情况下你可以任意命名，implicit 的名称只在两种情况下有用：一是你想在一个方法中明确指明，另外一个是想把那一个引入到当前作用域。比如我们定义一个对象，包含两个 implicit定义：

    object MyConversions {
        implicit def stringWrapper(s:String):IndexedSeq[Char] = ...
        implicit def intToString(x:Int):String = ...
    }
在你的应用中，你想使用 stringWrapper 变换，而不想把整数自动转换成字符串，你可以只引入 stringWrapper。

    import  MyConversions.stringWrapper
## 编译器使用 implicit 的几种情况
有三种情况使用 implicit: 一是转换成预期的数据类型，二是转换 selection 的 receiver，三是隐含参数。转换成预期的数据类型比如你有一个方法参数类型是 IndexedSeq[Char]，在你传入 String 时，编译器发现类型不匹配，就检查当前作用域是否有从 String 到 IndexedSeq 隐式转换。

转换 selection 的 receiver 允许你适应某些方法调用，比如 “abc”.exist ，”abc”类型为 String，本身没有定义 exist 方法，这时编辑器就检查当前作用域内 String 的隐式转换后的类型是否有 exist 方法，发现 stringWrapper 转换后成 IndexedSeq 类型后，可以有 exist 方法，这个和 C# 静态扩展方法功能类似。

隐含参数有点类似是缺省参数，如果在调用方法时没有提供某个参数，编译器会查找当前作用域是否有符合条件的 implicit 对象作为参数传入（有点类似 dependency injection)。


## implicit function 隐式函数
### 形式
第一种implicit的用法，是将其加在function定义的前面，形式为:

    implicit def int2String(someInt: Int): String = {
      //...
    }

### 作用
这种用法可以用来进行implicit conversion，隐式转换，也就是说，编译器可以选择在合适的时候调用这些函数来进行一个转换，来保证类型的正确性，比如我可以通过定义一个implicit的转换函数将java的类型转换为scala的类型，这样在需要scala类型但是却使用java类型作为参数的时候，编译器会自动加入这个转换函数.

      object HelloScala {
        implicit def conv(a: Int) = {
          println("in conv")
          a.toString
        }
        def say(b: String) = println(b)

        def main(args: Array[String])  {
          say(5)
        }
      }
      //输出结果:
      // in conv
      // 5
      //这说明过程是say(conv(5))
      //原因是编译器在检查的时候发现需要一个String类型的参数，但是代入的是一个Int，于是
      //他会在范围内寻找implicit的function，找到了符合这个要求的String => Int的function，于是调用

## implicit parameter & implicit value 隐式参数和隐式值
### 形式
### 隐式参数
隐式参数是在函数中，将参数标志出implicit，形式为:

      def func(implicit x: Int)
      def func2(x: Int)(implicit y: Int)
      def func3(implicit x: Int, y: Int)
这三种形式是有区别的，在参数中implicit只能出现一次，而在此之后，所有的参数都会变为implicit。

      func: x是implicit的
      func2: 只有y是implicit的
      func3: x和y都是implicit的

注意避免以下几种错误写法:
      //以下三种情况无法编译通过
      def err(x: Int, implicit y: Int)
      def err(implicit x: Int)(implicit y: Int)
      def err(implicit x: Int)(y: Int)
这三种情况都是无法编译通过的

## 隐式值


      implicit object Test
      implicit val x = 5
      implicit var y

### 作用

这种用法的作用主要是两种用法搭配起来来达到一个效果，隐式参数表明这个参数是可以缺少的，也就是说在调用的时候这个参数可以不用出现，那么这个值由什么填充呢？ 那就是用隐式的值了，以下的例子说明了这一点:
    object HelloScala {
      abstract class Sayable{
        def say
      }
      implicit object hello extends Sayable{
        override def say()={
          println("hello")
        }
      }
      def func(implicit x:Sayable): Unit ={
        x.say
      }
      implicit val impVal=5
      def func1(implicit x:Int)={
        println(x)
      }
      def main(args: Array[String])  {
        func
        func1
      }
    }

输出结果为:

      im in hello
      5

因为object的类型并不是object的名字，所以使用了一个抽象class来指明type。

在调用func的时候，没有代入参数，其参数是由编译器检查之后决定的，而这里决定的就是唯一的可能，hello那个object，所以这里的say调用的就是hello object里的say

在调用func1的时候，同样没有代入参数，需要一个Int作为参数，编译器寻找值的时候寻找到impVal是implicit的值，所以这里选择impVal作为他的值，输出了5

## implicit class 隐式类
这是一个在scala 2.10中新增的用法

### 形式

      implicit class MyClass(x: Int)

### 作用

这里的作用主要是其主构造函数可以作为隐式转换的参数，相当于其主构造函数可以用来当做一个implicit的function，下面举例说明一下:

    object HelloScala {
      implicit class MyName(x: Int) {
        println("im in cons")
        val y = x
      }

      def say(x: MyName) = {
        println(x.y)
      }

      def main(args: Array[String])  {
        say(5)
      }
    }

输出结果:

      im in cons
      5

这里的MyName是一个隐式类，其主构造函数可以用作隐式转换，所以say需要一个MyName类型的参数，但是调用的时候给的是一个Int，这里就会调用MyName的主构造函数转换为一个MyName的对象，然后再println其y的值



# 参考资料
【1】 https://www.iteblog.com/archives/1228.html
【2】 https://www.runoob.com/scala/currying-functions.html
【3】 https://blog.csdn.net/qq_29343201/article/details/58588470
