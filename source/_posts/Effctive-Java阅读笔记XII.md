title: Effctive-Java阅读笔记XII
date: 2020-1-16 17:17:12
categories: 2020年1月
tags: [Java]

---

第四章阅读：类和接口

23.类层次结构优于标签类

<!-- more -->

# 类层次结构优于标签类
　　有时你可能会碰到一个类，它的实例有两个或更多的风格，并且包含一个标签字段（tag field），表示实例的风格。 例如，考虑这个类，它可以表示一个圆形或矩形：

    // Tagged class - vastly inferior to a class hierarchy!
    class Figure {
        enum Shape { RECTANGLE, CIRCLE };

        // Tag field - the shape of this figure
        final Shape shape;

        // These fields are used only if shape is RECTANGLE
        double length;
        double width;

        // This field is used only if shape is CIRCLE
        double radius;

        // Constructor for circle
        Figure(double radius) {
            shape = Shape.CIRCLE;
            this.radius = radius;
        }

        // Constructor for rectangle
        Figure(double length, double width) {
            shape = Shape.RECTANGLE;
            this.length = length;
            this.width = width;
        }

        double area() {
            switch(shape) {
              case RECTANGLE:
                return length * width;
              case CIRCLE:
                return Math.PI * (radius * radius);
              default:
                throw new AssertionError(shape);
            }
        }
    }

　　这样的标签类具有许多缺点。 它们充斥着杂乱无章的样板代码，包括枚举声明，标签字段和 switch 语句。 可读性更差，因为多个实现在一个类中混杂在一起。 内存使用增加，因为实例负担属于其他风格不相关的领域。 字段不能成为 final，除非构造方法初始化不相关的字段，导致更多的样板代码。 构造方法在编译器的帮助下，必须设置标签字段并初始化正确的数据字段：如果初始化错误的字段，程序将在运行时失败。 除非可以修改其源文件，否则不能将其添加到标记的类中。 如果你添加一个风格，你必须记得给每个 switch 语句添加一个 case，否则这个类将在运行时失败。 最后，一个实例的数据类型没有提供任何关于风格的线索。 总之，***标签类是冗长的，容易出错的，而且效率低下。***
　　幸运的是，像 Java 这样的面向对象的语言为定义一个能够表示多种风格对象的单一数据类型提供了更好的选择：子类型化（subtyping）。标签类仅仅是一个类层次的简单的模仿。

　　要将标签类转换为类层次，首先定义一个包含抽象方法的抽象类，该标签类的行为取决于标签值。 在 Figure 类中，只有一个这样的方法，就是 area 方法。 这个抽象类是类层次的根。 如果有任何方法的行为不依赖于标签的值，把它们放在这个类中。 同样，如果有所有的方法使用的数据字段，把它们放在这个类。Figure 类中不存在这种与类型无关的方法或字段。

　　接下来，为原始标签类的每种类型定义一个根类的具体子类。 在我们的例子中，有两个类型：圆形和矩形。 在每个子类中包含特定于改类型的数据字段。 在我们的例子中，半径字段是属于圆的，长度和宽度字段都是矩形的。 还要在每个子类中包含根类中每个抽象方法的适当实现。 这里是对应于 Figure 类的类层次：


# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/23.%20%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E4%BC%98%E4%BA%8E%E6%A0%87%E7%AD%BE%E7%B1%BB
