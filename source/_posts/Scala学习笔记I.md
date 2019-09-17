---
title: Scala学习笔记I
date: 2019-9-10 11:32:12
categories: 2019年9月
tags: [Scala]

---

Scala学习

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

# 参考资料
【1】 https://www.iteblog.com/archives/1228.html
【2】
