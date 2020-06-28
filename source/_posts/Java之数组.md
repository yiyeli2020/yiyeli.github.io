---
title: Java之数组

date: 2020-6-26 19:30:12

categories: 2020年6月

tags: [Java,Array]

---
数组对于每一门编程语言来说都是重要的数据结构之一，当然不同语言对数组的实现及处理也不尽相同。

Java 语言中提供的数组是用来存储固定大小的同类型元素。


<!-- more -->
## 声明数组变量
首先必须声明数组变量，才能在程序中使用数组。下面是声明数组变量的语法：

    dataType[] arrayRefVar;   // 首选的方法
 
或
 
    dataType arrayRefVar[];  // 效果相同，但不是首选方法
注意: 建议使用 dataType[] arrayRefVar 的声明风格声明数组变量。 dataType arrayRefVar[] 风格是来自 C/C++ 语言 ，在Java中采用是为了让 C/C++ 程序员能够快速理解java语言。

## 实例
下面是这两种语法的代码示例：

    double[] myList;         // 首选的方法
 
或
 
    double myList[];         //  效果相同，但不是首选方法


## 多维数组
多维数组可以看成是数组的数组，比如二维数组就是一个特殊的一维数组，其每一个元素都是一个一维数组，例如：

    String str[][] = new String[3][4];
### 多维数组的动态初始化（以二维数组为例）
1. 直接为每一维分配空间，格式如下：

    type[][] typeName = new type[typeLength1][typeLength2];
type 可以为基本数据类型和复合数据类型，arraylength1 和 arraylength2 必须为正整数，arraylength1 为行数，arraylength2 为列数。

例如：

    int a[][] = new int[2][3];
解析：

二维数组 a 可以看成一个两行三列的数组。

2. 从最高维开始，分别为每一维分配空间，例如：
```  
    String s[][] = new String[2][];
    s[0] = new String[2];
    s[1] = new String[3];
    s[0][0] = new String("Good");
    s[0][1] = new String("Luck");
    s[1][0] = new String("to");
    s[1][1] = new String("you");
    s[1][2] = new String("!");
```
解析：

    s[0]=new String[2] 和 s[1]=new String[3] 
    
是为最高维分配引用空间，也就是为最高维限制其能保存数据的最长的长度，然后再为其每个数组元素单独分配空间 s0=new String("Good") 等操作。


# Arrays 类
java.util.Arrays 类能方便地操作数组，它提供的所有方法都是静态的。

具有以下功能：
    
    给数组赋值：通过 fill 方法。
    对数组排序：通过 sort 方法,按升序。
    比较数组：通过 equals 方法比较数组中元素值是否相等。
    查找数组元素：通过 binarySearch 方法能对排序好的数组进行二分查找法操作。

1.	public static int binarySearch(Object[] a, Object key)

用二分查找算法在给定数组中搜索给定值的对象(Byte,Int,double等)。数组在调用前必须排序好的。如果查找值包含在数组中，则返回搜索键的索引；否则返回 (-(插入点) - 1)。
2.	public static boolean equals(long[] a, long[] a2)

如果两个指定的 long 型数组彼此相等，则返回 true。如果两个数组包含相同数量的元素，并且两个数组中的所有相应元素对都是相等的，则认为这两个数组是相等的。换句话说，如果两个数组以相同顺序包含相同的元素，则两个数组是相等的。同样的方法适用于所有的其他基本数据类型（Byte，short，Int等）。
3.	public static void fill(int[] a, int val)

将指定的 int 值分配给指定 int 型数组指定范围中的每个元素。同样的方法适用于所有的其他基本数据类型（Byte，short，Int等）。
4.	public static void sort(Object[] a)

对指定对象数组根据其元素的自然顺序进行升序排列。同样的方法适用于所有的其他基本数据类型（Byte，short，Int等）。

[^1]:https://www.runoob.com/java/java-array.html