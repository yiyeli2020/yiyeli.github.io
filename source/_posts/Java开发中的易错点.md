title: Java开发中的易错点
date: 2020-1-3 16:11:12
categories: 2020年1月
tags: [Java]

---

Java开发中的易错点

<!-- more -->

# 避免非空判断仅判断是否为empty
反例：

    public  static  void  main(String[] args){
        Map data=new HashMap();
        data=null;
        if(data.isEmpty()){
            return;
        }
    }

只判断data是否为isEmpty，但当数据为null时会报空指针异常

    Exception in thread "main" java.lang.NullPointerException
    	at Application.main(Application.java:454)

# log抛出要有详情和具体说明
反例：

    try {
      param = JSONObject.parseObject(methodParam,method.getParameterTypes()[0]);
    } catch (Exception e){
      throw new ParamException(e.getMessage());
    }
信息需要详细具体，参见
正例：

    try {
      ...
    } catch (IOException e){
      logger.error("....Failed",e);
      return null;
    }

# 配置文件尽量在配置文件定义，代码中不要配置

反例：

  String ip="127.0.0.1";
  Socket socket=new Socket(ip,6667);

正例：

  String ip= System.getProperty("myapplication.ip");
  Socket socket=new Socket(ip,6667);

# 精度转换

BigDecimal转double，需先转换为string再 BigDecimal（String s），再使用BigDecimal.valueOf()方法，避免精度丢失

BigDecimal(double val)构造，用double当参数来构造一个BigDecimal对象。
但是BigDecimal(0.1)实际上等于0.1000000000000000055511151231257827021181583404541015625，因为准确的来说0.1本身不能算是一个double（其实0.1不能代表任何一个定长二进制分数）。

BigDecimal(String val)构造是靠谱的，BigDecimal(“0.1”)就是等于0.1，推荐大家用这个构造。

如果非要用一个double变量来构造一个BigDecimal，请使用静态方法valueOf(double)，这个方法跟new Decimal(Double.toString(double))效果是一样的。

    public static void doubleToBigDecimal(){
        float f=0.1f;
        BigDecimal decimal1=new BigDecimal(f);
        System.out.println(decimal1);
        double d=0.1;
        BigDecimal decimal2=new BigDecimal(d);
        System.out.println(decimal2);
        BigDecimal decimal3=new BigDecimal(Double.toString(d));
        System.out.println(decimal3);
        BigDecimal decimal4=BigDecimal.valueOf(d);
        System.out.println(decimal4);
    }

输出结果：

    0.100000001490116119384765625
    0.1000000000000000055511151231257827021181583404541015625
    0.1
    0.1

# Map、Set、list初始化

    Map source = new HashMap(){{ // Noncompliant
        put("firstName", "John");
        put("lastName", "Smith");
    }};

上述方式潜在问题：

看起来优雅了不少，一步到位。问题来了，这里的双大括号到底什么意思，什么用法呢？
双大括号,用来初始化，使代码简洁易读。
第一层括弧实际是定义了一个匿名内部类 (Anonymous Inner Class)，第二层括弧实际上是一个实例初始化块 (instance initializer block)，这个块在内部匿名类构造时被执行。

1.此种方式是匿名内部类的声明方式，所以引用中持有着外部类的引用。所以当时串行化这个集合时外部类也会被不知不觉的串行化，当外部类没有实现serialize接口时，就会报错。

2.上例中，其实是声明了一个继承自HashMap的子类。然而有些串行化方法，例如要通过Gson串行化为json，或者要串行化为xml时，类库中提供的方式，是无法串行化Hashset或者HashMap的子类的，从而导致串行化失败。
解决办法：重新初始化为一个HashMap对象：
new HashMap(map);
这样就可以正常初始化了。

    Map source = new HashMap() // compliant
    source.put("firstName", "John");
    source.put("lastName", "Smith");

# null值时刻注意非空判断

反例：

    TreeMap<String,String> paramTreeMap=convertMapToTreeMap(paramMap);
    try{
      .....
    } catch (Exception e){
      throw new FailException(" .... param="+ paramMap.toString() e);
    }

paramMap.toString()这里没有考虑为null的情况

# 文件数据流、zip流、字节流等一定要谨记在finally关闭数据流

反例1:

    private void readFile() throw IOException{
      Path path=Paths.get(this.fileName);
      BufferedReader reader=Files.newBufferedReader(path,this.charset);
      reader.close();   //Noncompliant
    }
反例2:

    private void doSomething(){
      OutputStream stream=null;
      try {
        for(String property:propertyList){
          stream = new FileOutputStream("file.txt"); //Noncompliant
          // ...
        }
      } catch (Exception e){
        //...
      } finally{
        stream.close(); //Multiple streams were opened. Only the last is closed.
      }
    }

# 返回值尽量不要进行复杂的条件判断，条件和结果尽量清楚明了

# 把数组转成ArrayList

为了将数组转换为ArrayList，开发者经常会这样做：

    List list = Arrays.asList(arr);

使用Arrays.asList()方法可以得到一个ArrayList，但是得到这个ArrayList其实是定义在Arrays类中的一个私有的静态内部类。这个类虽然和java.util.ArrayList同名，但是并不是同一个类。java.util.Arrays.ArrayList类中实现了set(), get(), contains()等方法，但是并没有定义向其中增加元素的方法。也就是说通过Arrays.asList()得到的ArrayList的大小是固定的。

如果在开发过程中，想得到一个真正的ArrayList对象（java.util.ArrayList的实例），可以通过以下方式：

    ArrayList<String> arrayList = new ArrayList<String>(Arrays.asList(arr));

java.util.ArrayList中包含一个可以接受集合类型参数的构造函数。因为java.util.Arrays.ArrayList这个内部类继承了AbstractList类，所以，该类也是Collection的子类。

# 判断一个数组是否包含某个值

在判断一个数组中是否包含某个值的时候，开发者经常这样做：

    Set<String> set = new HashSet<String>(Arrays.asList(arr));
    return set.contains(targetValue);

在《在Java中如何高效的判断数组中是否包含某个元素》[2]中深入分析过,以上方式虽然可以实现功能，但是效率却比较低。因为将数组压入Collection类型中，首先要将数组元素遍历一遍，然后再使用集合类做其他操作。
在判断一个数组是否包含某个值的时候，推荐使用for循环遍历的形式或者使用Apache Commons类库中提供的ArrayUtils类的contains方法。


# 在循环中删除列表中的元素

在讨论这个问题之前，先考虑以下代码的输出结果：

    ArrayList<String> list = new ArrayList<String>(Arrays.asList("a","b","c","d"));
    for(int i=0;i<list.size();i++){
    list.remove(i);
    }
    System.out.println(list);

输出结果：

    [b,d]

以上代码的目的是想遍历删除list中所有元素，但是结果却没有成功。原因是忽略了一个关键的问题：当一个元素被删除时，列表的大小缩小并且下标也会随之变化，所以当你想要在一个循环中用下标删除多个元素的时候，它并不会正常的生效。

也有些人知道以上代码的问题就由于数组下标变换引起的。所以，他们想到使用增强for循环的形式：

    ArrayList<String> list = new ArrayList<String>(Arrays.asList("a","b","c","d"));
    for(String s:list){
        if(s.equals("a")){
            list.remove(s);
        }
    }

但是，很不幸的是，以上代码会抛出ConcurrentModificationException，有趣的是，如果在remove操作后增加一个break，代码就不会报错：

    ArrayList<String> list = new ArrayList<String>(Arrays.asList("a","b","c","d"));
    for(String s:list){
        if(s.equals("a")){
            list.remove(s);
            break;
        }
    }

在《Java中的fail-fast机制》[3]一文中，深入分析了几种在遍历数组的同时删除其中元素的方法以及各种方法存在的问题。其中就介绍了上面的代码出错的原因。
迭代器（Iterator）是工作在一个独立的线程中，并且拥有一个 mutex 锁。 迭代器被创建之后会建立一个指向原来对象的单链索引表，当原来的对象数量发生变化时，这个索引表的内容不会同步改变，所以当索引指针往后移动的时候就找不到要迭代的对象，所以按照 fail-fast 原则 迭代器会马上抛出java.util.ConcurrentModificationException 异常。

所以，正确的在遍历过程中删除元素的方法应该是使用Iterator：

    ArrayList<String> list = new ArrayList<String>(Arrays.asList("a", "b", "c", "d"));
    Iterator<String> iter = list.iterator();
    while (iter.hasNext()) {
        String s = iter.next();
        if (s.equals("a")) {
            iter.remove();
        }
    }

next()方法必须在调用remove()方法之前调用。如果在循环过程中先调用remove()，再调用next()，就会导致异常ConcurrentModificationException。原因如上。

# HashTable 和 HashMap 的选择
了解算法的人可能对HashTable比较熟悉，因为他是一个数据结构的名字。但在Java里边，用HashMap来表示这样的数据结构。Hashtable和 HashMap的一个关键性的不同是，HashTable是同步的，而HashMap不是。所以通常不需要HashTable，HashMap用的更多。
《HashMap完全解读》[4]、《Java中常见亲属比较》[5]等文章中介绍了他们的区别和如何选择。

# 使用原始集合类型

在Java里边，原始类型和无界通配符类型很容易混合在一起。以Set为例，Set是一个原始类型，而Set< ? >是一个无界通配符类型。 （可以把原始类型理解为没有使用泛型约束的类型）

考虑下面使用原始类型List作为参数的代码：

    public static void add(List list, Object o){
        list.add(o);
    }

    public static void main(String[] args){
        List<String> list = new ArrayList<String>();
        add(list, 10);
        String s = list.get(0);
    }

上面的代码将会抛出异常：

    java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String

使用原始集合类型是很危险的，因为原始集合类型跳过了泛型类型检查，是不安全的。Set、Set< ? >和Set< Object >之间有很大差别。关于泛型，可以参考下列文章：《Java基础知识——泛型》[12]

# 访问级别

程序员们经常使用public作为类中的字段的修饰符，因为这样可以很简单的通过引用得到值，但这并不是好的设计，按照经验，分配给成员变量的访问级别应该尽可能的低。参考《Java中的四种访问级别》[13]

# ArrayList与LinkedList的选择

当程序员们不知道ArrayList与LinkedList的区别时，他们经常使用ArrayList，因为它看起来比较熟悉。然而，它们之前有巨大的性能差别。在《ArrayList vs LinkedList vs Vector 区别》[8]、《Java中常见亲属比较》[9]等文章中介绍过，简而言之，如果有大量的增加删除操作并且没有很多的随机访问元素的操作，应该首先LinkedList。（LinkedList更适合从中间插入或者删除（链表的特性））

# 可变与不可变
在《为什么Java要把字符串设计成不可变的》[10]一文中介绍过，不可变对象有许多的优点，比如简单，安全等等。同时，也有人提出疑问：既然不可变有这么多好处，为什么不把所有类都搞成不可变的呢？

通常情况下，可变对象可以用来避免产生过多的中间对象。一个经典的实例就是连接大量的字符串，如果使用不可变的字符串，将会产生大量的需要进行垃圾回收的对象。这会浪费CPU大量的时间，使用可变对象才是正确的方案(比如StringBuilder)。

    String result="";
    for(String s: arr){
        result = result + s;
    }
StackOverflow[11]中也有关于这个的讨论。

# ””还是构造函数
关于这个问题，也是程序员经常出现困惑的地方，在《该如何创建字符串，使用” “还是构造函数？》[12]中也介绍过。
如果你只需要创建一个字符串，你可以使用双引号的方式，如果你需要在堆中创建一个新的对象，你可以选择构造函数的方式。
在String d = new String("abcd")时，因为字面值“abcd”已经是字符串类型，那么使用构造函数方式只会创建一个额外没有用处的对象。



# 参考资料
【1】https://www.hollischuang.com/archives/1360
【2】http://www.hollischuang.com/archives/1269
【3】http://www.hollischuang.com/archives/33
【4】http://www.hollischuang.com/archives/82
【5】http://www.hollischuang.com/archives/442
【6】http://www.hollischuang.com/archives/1182
【7】http://www.hollischuang.com/archives/1182
【8】http://www.hollischuang.com/archives/1349
【9】http://www.hollischuang.com/archives/442
【10】http://www.hollischuang.com/archives/1246
【11】https://stackoverflow.com/questions/23616211/why-we-need-mutable-classes
【12】http://www.hollischuang.com/archives/1249
