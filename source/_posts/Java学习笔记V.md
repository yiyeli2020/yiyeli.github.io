title: Java学习笔记V
date: 2019-8-12 14:48:12
categories: 2019年8月
tags: [Java]

---

学习Guaa中的字符串处理函数Joiner类的用法。

<!-- more -->
# Guava
Guava 是一个 Google 的基于java1.6的类库集合的扩展项目，Guava工程包含了若干被Google的 Java项目广泛依赖 的核心库，例如：集合 [collections] 、缓存 [caching] 、原生类型支持 [primitives support] 、并发库 [concurrency libraries] 、通用注解 [common annotations] 、字符串处理 [string processing] 、I/O 等等。

# 连接器Joiner
用分隔符把字符串序列连接起来也可能会遇上不必要的麻烦。如果字符串序列中含有null，那连接操作会更难。Fluent风格的Joiner让连接字符串更简单。

  Joiner joiner = Joiner.on("; ").skipNulls();
  return joiner.join("Harry", null, "Ron", "Hermione");

上述代码返回”Harry; Ron; Hermione”。另外，useForNull(String)方法可以给定某个字符串来替换null，而不像skipNulls()方法是直接忽略null。
 Joiner也可以用来连接对象类型，在这种情况下，它会把对象的toString()值连接起来。

    Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"

警告：joiner实例总是不可变的。用来定义joiner目标语义的配置方法总会返回一个新的joiner实例。
这使得joiner实例都是线程安全的，你可以将其定义为static final常量。

# 传统连接方法
相比之下传统的以某个分隔符来进行拼接的代码如下：

    public static String concatString(List<String> lists,String delimiter){
        StringBuilder builder=new StringBuilder();
        for(String s:lists){
            if(s!=null){
                builder.append(s).append(delimiter);
            }
        }
        builder.setLength(builder.length()-delimiter.length());
        return builder.toString();
    }
    public  static  void  main(String[] args) {
      List<String> list=new ArrayList();
      list.add("Traditional");
      list.add("delimiter");
      list.add("codingstyle");
      System.out.println(concatString(list,"-"));
    }

# MapJoiner

MapJoiner的用法和Joiner类似,不过MapJoiner主要针对map的字符串拼接例：

    Map<String,String> maps=Maps.newHashMap();
    maps.put("MapJoiner","1");
    maps.put("String","delimiter");
    String ss=Joiner.on("$").withKeyValueSeparator("=").join(maps);
    System.out.println(ss);

# 参考资料：
【1】https://wizardforcel.gitbooks.io/guava-tutorial/content/1.html

【2】https://blog.csdn.net/u012415194/article/details/84880258
