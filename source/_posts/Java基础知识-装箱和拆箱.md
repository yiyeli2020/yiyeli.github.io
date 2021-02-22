---

title: Java基础知识-装箱和拆箱

date: 2021-02-22 11:25:12

categories: 2021年2月

tags: [Java]


---
 
装箱和拆箱

<!-- more -->

# 什么是装箱？什么是拆箱？

在java中有八种基本数据类型对应每种基本类型又有八种包装类型：[^1]

基本类型：boolean， char， int， byte，short，long， float，double

包装器类型：Boolean，Character，Integer，Byte，Short，Long，Float，Double

从上面我们可以看到除了 char和int其它的包装类型名称和对应的基本类型一样只是首字母大写了。

既然有了基本类型为什么还要有包装类呢？我们在使用的过程中究竟用基本类型还是包装类呢？

在某些场合不能使用基本类型必须使用包装类，比如集合能接收的类型为Object,基本类型是无法添加进去的，还有范型也必须使用包装类。

另外假设我们要定义一个变量表示分数 如果用基本类型表示的话：int score;

默认值为零，如果我想表示分数为空也就是没有参加考试就没法表现了因为值类型是无法赋空值的，如果使用包装类型Integer score,就可以表示这种情况，因为Integer的默认值为空。

在Java SE5之前，如果要生成一个数值为10的Integer对象，必须这样进行：


    Integer i = new Integer(10);
　　而在从Java SE5开始就提供了自动装箱的特性，如果要生成一个数值为10的Integer对象，只需要这样就可以了：

    Integer i = 10;
　　这个过程中会自动根据数值创建对应的 Integer对象，这就是装箱。

　　那什么是拆箱呢？顾名思义，跟装箱对应，就是自动将包装器类型转换为基本数据类型：


    Integer i = 10;  //装箱
    int n = i;   //拆箱
　　
　　简单一点说，装箱就是  自动将基本数据类型转换为包装器类型；拆箱就是  自动将包装器类型转换为基本数据类型。

# 装箱和拆箱是如何实现的

这部分主要内容来自于[^2]，作者是Matrix海子，写的很清楚，这里为了阅读方便转载一下，内容如下：

<details>
    <summary>装箱和拆箱是如何实现的</summary>

```

public class Main {
    public static void main(String[] args) {
         
        Integer i = 10;
        int n = i;
    }
}

```
</details>

先编译`javac Main.java`，然后反编译class文件 `javap -c Main`后

从反编译得到的字节码内容可以看出，在装箱的时候自动调用的是Integer的valueOf(int)方法。而在拆箱的时候自动调用的是Integer的intValue方法。

　　其他的也类似，比如Double、Character，不相信的朋友可以自己手动尝试一下。

　　因此可以用一句话总结装箱和拆箱的实现过程：

　　装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的。（xxx代表对应的基本数据类型）。


# 面试中相关的问题

1. 代码的输出结果是什么
<details>
    <summary>代码的输出结果是什么</summary>

```
public class Main {
    public static void main(String[] args) {
         
        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;
         
        System.out.println(i1==i2);
        System.out.println(i3==i4);
    }
}

```
</details>


输出结果表明i1和i2指向的是同一个对象，而i3和i4指向的是不同的对象。此时只需一看源码便知究竟,Integer的valueOf方法的具体实现和IntegerCache类的实现，通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。

　　上面的代码中i1和i2的数值为100，因此会直接从cache中取已经存在的对象，所以i1和i2指向的是同一个对象，而i3和i4则是分别指向不同的对象。



2. 代码的输出结果是什么

<details>
    <summary>代码的输出结果是什么</summary>

```
public class Main {
    public static void main(String[] args) {
         
        Double i1 = 100.0;
        Double i2 = 100.0;
        Double i3 = 200.0;
        Double i4 = 200.0;
         
        System.out.println(i1==i2);
        System.out.println(i3==i4);
    }
}

```
</details>

具体为什么，可以去看Double类的valueOf的实现。

　　在这里只解释一下为什么Double类的valueOf方法会采用与Integer类的valueOf方法不同的实现。很简单：在某个范围内的整型数值的个数是有限的，而浮点数却不是。

　　注意，Integer、Short、Byte、Character、Long这几个类的valueOf方法的实现是类似的。

　　　　　Double、Float的valueOf方法的实现是类似的。
　


3. 代码的输出结果是什么

<details>
    <summary>代码的输出结果是什么</summary>

```
public class Main {
    public static void main(String[] args) {

        Boolean i1 = false;
        Boolean i2 = false;
        Boolean i3 = true;
        Boolean i4 = true;
         
        System.out.println(i1==i2);
        System.out.println(i3==i4);
    }
}

```
</details>


输出结果都是true,至于为什么是这个结果，同样地，看了Boolean类的源码也会一目了然

4. 谈谈Integer i = new Integer(xxx)和Integer i =xxx;这两种方式的区别。

　　当然，这个题目属于比较宽泛类型的。但是要点一定要答上，我总结一下主要有以下这两点区别：

　　1）第一种方式不会触发自动装箱的过程；而第二种方式会触发；

　　2）在执行效率和资源占用上的区别。第二种方式的执行效率和资源占用在一般性情况下要优于第一种情况（注意这并不是绝对的）。



5. 代码的输出结果是什么

<details>
    <summary>代码的输出结果是什么</summary>

```
public class Main {
    public static void main(String[] args) {
         
        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Integer d = 3;
        Integer e = 321;
        Integer f = 321;
        Long g = 3L;
        Long h = 2L;
         
        System.out.println(c==d);
        System.out.println(e==f);
        System.out.println(c==(a+b));
        System.out.println(c.equals(a+b));
        System.out.println(g==(a+b));
        System.out.println(g.equals(a+b));
        System.out.println(g.equals(a+h));
    }
}

```
</details>

当 "=="运算符的两个操作数都是包装器类型的引用，则是比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）。另外，对于包装器类型，equals方法并不会进行类型转换。明白了这2点之后，上面的输出结果便一目了然：
    
    true
    false
    true
    true
    true
    false
    true

第一个和第二个输出结果没有什么疑问。第三句由于  a+b包含了算术运算，因此会触发自动拆箱过程（会调用intValue方法），因此它们比较的是数值是否相等。而对于c.equals(a+b)会先触发自动拆箱过程，再触发自动装箱过程，也就是说a+b，会先各自调用intValue方法，得到了加法运算后的数值之后，便调用Integer.valueOf方法，再进行equals比较。同理对于后面的也是这样，不过要注意倒数第二个和最后一个输出的结果（如果数值是int类型的，装箱过程调用的是Integer.valueOf；如果是long类型的，装箱调用的Long.valueOf方法）。

如果对上面的具体执行过程有疑问，可以尝试获取反编译的字节码内容进行查看。





[^1]:https://zhuanlan.zhihu.com/p/78590948

[^2]:https://www.cnblogs.com/dolphin0520/p/3780005.html