---
title: Java学习笔记XIV
date: 2020-01-24 17:34:12
categories: 2020年1月
tags: [Java]

---

Java的值传递和引用传递

<!-- more -->


java中数据类型分为基本数据类型和引用数据类型。

基本数据类型

    整型：byte，short，int，long
    浮点型：float，double
    字符型：char
    布尔型：boolean
引用数据类型

    数组
    类
    接口

方法的参数分为实际参数，和形式参数。

    形式参数：定义方法时写的参数。
    实际参数：调用方法时写的具体数值。

一般情况下，在数据做为参数传递的时候，***基本数据类型是值传递，引用数据类型是引用传递（地址传递）。***

# 值传递

    public static void main(String[] args) {
        int num1 = 10;
        int num2 = 20;

        swap(num1, num2);

        System.out.println("num1 = " + num1);
        System.out.println("num2 = " + num2);
    }

    public static void swap(int a, int b) {
        int temp = a;
        a = b;
        b = temp;

        System.out.println("a = " + a);
        System.out.println("b = " + b);
    }

运行结果：

    a = 20
    b = 10
    num1 = 10
    num2 = 20

## 流程：

主函数进栈，num1、num2初始化。
调用swap方法，swap( )进栈，将num1和num2的值，复制一份给a和b。
swap方法中对a、b的值进行交换。
swap方法运行完毕，a、b的值已经交换。
swap方法弹栈。
主函数弹栈。

## 解析：

在swap方法中，a、b的值进行交换，并不会影响到num1、num2。因为，a、b中的值，只是从num1、num2的复制过来的。
也就是说，a、b相当于num1、num2的副本，副本的内容无论怎么修改，都不会影响到原件本身。

# 引用传递

    public static void main(String[] args) {
        int[] arr = {1,2,3,4,5};

        change(arr);

        System.out.println(arr[0]);
    }

    //将数组的第一个元素变为0
    public static void change(int[] array) {
        int len = array.length;
        array[0] = 0;
    }

运行结果是：
    0

# 流程：

主函数进栈，int[] arr初始化。
调用change方法，change( )进栈，将arr的地址值，复制一份给array。
change方法中，根据地址值，找到堆中的数组，并将第一个元素的值改为0。
change方法运行完毕，数组中第一个元素的值已经改变。
change方法弹栈。
主函数弹栈。

# 解析：

调用change()的时候，形参array接收的是arr地址值的副本。并在change方法中，通过地址值，对数组进行操作。change方法弹栈以后，数组中的值已经改变。main方法中，打印出来的arr[0]也就从原来的1变成了0.

无论是主函数，还是change方法，操作的都是同一个地址值对应的数组。
就像你把自己家的钥匙给了另一个人，这个人拿着钥匙在你家一顿瞎折腾，然后走了。等你拿着钥匙回到家以后，家里已经变成了被折腾过后，惨不忍睹的样子。。
这里的钥匙就相当于地址值，家就相当于数组本身。

# String类型传递

    public static void main(String[] args) {
        String str = "AAA";

        change(str);

        System.out.println(str);
    }   
    public static void change(String s) {
        s = "abc";
    }
运行的结果是：

    AAA

String是一个类，类是引用数据类型，做为参数传递的时候，应该是引用传递。但是从结果看起来却是值传递。

## 原因：
String的API中有这么一句话：“their values cannot be changed after they are created”，
意思是：String的值在创建之后不能被更改。
API中还有一段：
String str = "abc";
等效于：

    char data[] = {'a', 'b', 'c'};
    String str = new String(data);
也就是说：对String对象str的任何修改 等同于 重新创建一个对象，并将新的地址值赋值给str。

这样的话，上面的代码就可以写成：

    public static void main(String[] args) {
        String str1 = "AAA";

        change(str1);

        System.out.println(str1);
    }   
    public static void change(String s) {
        char data[] = {'a', 'b', 'c'}
        String str = new String(data);
        s = str;
    }

## 流程：
主函数进栈，str1初始化。
调用change方法，change( )进栈，将str1的地址值，复制一份给s。
change方法中，重现创建了一个String对象”abc”，并将s指向了新的地址值。
change方法运行完毕，s所指向的地址值已经改变。
change方法弹栈。
主函数弹栈。
## 解析：
String对象做为参数传递时，走的依然是引用传递，只不过String这个类比较特殊。
String对象一旦创建，内容不可更改。每一次内容的更改都是重现创建出来的新对象。
当change方法执行完毕时，s所指向的地址值已经改变。而s本来的地址值就是copy过来的副本，所以并不能改变str1的值。

# String类型类似情况：

    class Person {
        String name;

        public Person(String name) {
            this.name = name;
        }
    }
    public class Test {
        public static void main(String[] args) {
            Person p = new Person("张三");

            change(p);

            System.out.println(p.name);
        }

        public static void change(Person p) {
            Person person = new Person("李四");
            p = person;
        }
    }

运行的结果是：

    张三
# 总结
值传递的时候，将实参的值，copy一份给形参。
引用传递的时候，将实参的地址值，copy一份给形参。
也就是说，不管是值传递还是引用传递，形参拿到的仅仅是实参的副本，而不是实参本身。

# 参考资料：
【1】https://blog.csdn.net/zhzhao999/article/details/53449504
