---
title: Java之String类

date: 2020-6-18 15:30:12

categories: 2020年6月

tags: [Java]

---

介绍Java的String类及其常用方法例如substring()方法。

<!-- more -->


字符串广泛应用 在 Java 编程中，在 Java 中字符串属于对象，Java 提供了 String 类来创建和操作字符串。[^1]


# String方法

## substring() 方法[^2]
substring() 方法返回字符串的子字符串。

### 语法

    public String substring(int beginIndex)
    
    或
    
    public String substring(int beginIndex, int endIndex)
### 参数
    
    beginIndex -- 起始索引（包括）, 索引从 0 开始。
    
    endIndex -- 结束索引（不包括）。

返回值

子字符串。

```
public class Test {
    public static void main(String args[]) {
        String Str = new String("www.runoob.com");
 
        System.out.print("返回值 :" );
        System.out.println(Str.substring(4) );
 
        System.out.print("返回值 :" );
        System.out.println(Str.substring(4, 10) );
    }
}
```
以上程序执行结果为：
    
    返回值 :runoob.com
    返回值 :runoob


## Java toCharArray() 方法[^3]

toCharArray() 方法将字符串转换为字符数组。

### 语法
public char[] toCharArray()
### 参数
无

### 返回值
字符数组。

### 实例
```
public class Test {
    public static void main(String args[]) {
        String Str = new String("www.runoob.com");

        System.out.print("返回值 :" );
        System.out.println( Str.toCharArray() );
    }
}
```
以上程序执行结果为：

    返回值 :www.runoob.com


[^1]:https://www.runoob.com/java/java-string.html

[^2]:https://www.runoob.com/java/java-string-substring.html

[^3]:https://www.runoob.com/java/java-string-tochararray.html