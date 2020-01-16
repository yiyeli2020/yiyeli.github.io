title: Effctive-Java阅读笔记XII
date: 2020-1-16 17:17:12
categories: 2020年1月
tags: [Java]

---

第四章阅读：类和接口

23.类层次结构优于标签类
当遇到一个带有标签字段的现有类时，可以考虑将其重构为一个类层次结构。
24.支持使用静态成员类而不是非静态类


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

    // Class hierarchy replacement for a tagged class
    abstract class Figure {
        abstract double area();
    }

    class Circle extends Figure {
        final double radius;

        Circle(double radius) { this.radius = radius; }

        @Override double area() { return Math.PI * (radius * radius); }
    }
    class Rectangle extends Figure {
        final double length;
        final double width;

        Rectangle(double length, double width) {
            this.length = length;
            this.width  = width;
        }
        @Override double area() { return length * width; }
    }

　　这个类层次纠正了之前提到的标签类的每个缺点。 代码简单明了，不包含原文中的样板文件。 每种类型的实现都是由自己的类来分配的，而这些类都没有被无关的数据字段所占用。 所有的字段是 final 的。 编译器确保每个类的构造方法初始化其数据字段，并且每个类都有一个针对在根类中声明的每个抽象方法的实现。 这消除了由于缺少 switch-case 语句而导致的运行时失败的可能性。 多个程序员可以独立地继承类层次，并且可以相互操作，而无需访问根类的源代码。 每种类型都有一个独立的数据类型与之相关联，允许程序员指出变量的类型，并将变量和输入参数限制为特定的类型。

　　类层次的另一个优点是可以使它们反映类型之间的自然层次关系，从而提高了灵活性，并提高了编译时类型检查的效率。 假设原始示例中的标签类也允许使用正方形。 类层次可以用来反映一个正方形是一种特殊的矩形（假设它们是不可变的）：

    class Square extends Rectangle {
        Square(double side) {
            super(side, side);
        }
    }
　　请注意，上述层次结构中的字段是直接访问的，而不是通过访问器方法访问的。 这里是为了简洁起见，如果类层次是公开的（详见第 16 条），这将是一个糟糕的设计。

　　总之，标签类很少有适用的情况。 如果你想写一个带有显式标签字段的类，请考虑标签字段是否可以被删除，并是否能被类层次结构替换。 当遇到一个带有标签字段的现有类时，可以考虑将其重构为一个类层次结构。

# 支持使用静态成员类而不是非静态类

　　嵌套类（nested class）是在另一个类中定义的类。 嵌套类应该只存在于其宿主类（enclosing class）中。 如果一个嵌套类在其他一些情况下是有用的，那么它应该是一个顶级类。 有四种嵌套类：静态成员类，非静态成员类，匿名类和局部类。 除了第一种以外，剩下的三种都被称为内部类（inner class）。 这个条目告诉你什么时候使用哪种类型的嵌套类以及为什么使用。

　　静态成员类是最简单的嵌套类。 最好把它看作是一个普通的类，恰好在另一个类中声明，并且可以访问所有宿主类的成员，甚至是那些被声明为私有类的成员。 静态成员类是其宿主类的静态成员，并遵循与其他静态成员相同的可访问性规则。 如果它被声明为 private，则只能在宿主类中访问，等等。

　　静态成员类的一个常见用途是作为公共帮助类，仅在与其外部类一起使用时才有用。 例如，考虑一个描述计算器支持的操作的枚举类型（详见第 34 条）。 Operation 枚举应该是 Calculator 类的公共静态成员类。 Calculator 客户端可以使用 Calculator.Operation.PLUS 和 Calculator.Operation.MINUS 等名称来引用操作。

　　在语法上，静态成员类和非静态成员类之间的唯一区别是静态成员类在其声明中具有 static 修饰符。 尽管句法相似，但这两种嵌套类是非常不同的。 非静态成员类的每个实例都隐含地与其包含的类的宿主实例相关联。 在非静态成员类的实例方法中，可以调用宿主实例上的方法，或者使用限定的构造[JLS，15.8.4] 获得对宿主实例的引用。 如果嵌套类的实例可以与其宿主类的实例隔离存在，那么嵌套类必须是静态成员类：不可能在没有宿主实例的情况下创建非静态成员类的实例。

　　非静态成员类实例和其宿主实例之间的关联是在创建成员类实例时建立的，并且之后不能被修改。 通常情况下，通过在宿主类的实例方法中调用非静态成员类构造方法来自动建立关联。 尽管很少有可能使用表达式 enclosingInstance.new MemberClass(args) 手动建立关联。 正如你所预料的那样，该关联在非静态成员类实例中占用了空间，并为其构建添加了时间开销。

　　非静态成员类的一个常见用法是定义一个 Adapter [Gamma95]，它允许将外部类的实例视为某个不相关类的实例。 例如，Map 接口的实现通常使用非静态成员类来实现它们的集合视图，这些视图由 Map 的 keySet，entrySet 和 values 方法返回。 同样，集合接口（如 Set 和 List）的实现通常使用非静态成员类来实现它们的迭代器：


# 参考资料：
【1】http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/23.%20%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E4%BC%98%E4%BA%8E%E6%A0%87%E7%AD%BE%E7%B1%BB
