title: Lombok的基本注解使用
date: 2020-5-18 22:05:12
categories: 2020年5月
tags: [Java]

---

学习使用Lombok的基本注解：@Data，@AllArgsConstructor，@NoArgsConstructor，@Builder

<!-- more -->

Lombok 是一种 Java™ 实用工具，可用来帮助开发人员消除 Java 的冗长，尤其是对于简单的 Java 对象（POJO）。它通过注解实现这一目的。

# 添加依赖
在 pom.xml 文件中添加相关依赖：

    <lombok.version>1.16.20</lombok.version>

    <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <scope>provided</scope>
            </dependency>

# 安装插件
由于 Lombok 采取的注解形式的，在编译后，自动生成相应的方法，需要下载插件来支持它。
以 IDEA 为例：查找插件 lombok plugin 安装即可。

以 User 实体类为例（set,get,toString 方法），

拿lombok官网的一个例子来说:
    public class Mountain{
        private String name;
        private double longitude;
        private String country;
    }
要使用这个对象,必须还要写一些getter和setter方法,可能还要写一个构造器、equals方法、或者hash方法.这些方法很冗长而且没有技术含量,我们叫它样板式代码.
lombok的主要作用是通过一些注解，消除样板式代码，像这样：

    @Data
    public class Mountain{
        private String name;
        private double longitude;
        private String country;
    }
然后这个类自动生成了这些方法.

# @Data 生成getter,setter等函数
使用这个注解，就不用再去手写Getter,Setter,equals,canEqual,hasCode,toString等方法了，注解后在编译时会自动加进去。@Data 包含了 @ToString、@EqualsAndHashCode、@Getter / @Setter和@RequiredArgsConstructor的功能.
# @AllArgsConstructor 生成全参数构造函数
使用后添加一个构造函数，该构造函数含有所有已声明字段属性参数
# @NoArgsConstructor 生成全参数构造函数
使用后创建一个无参构造函数,即没有参数的构造函数。
# @Builder
关于Builder较为复杂一些，Builder的作用之一是为了解决在某个类有很多构造函数的情况，也省去写很多构造函数的麻烦，在设计模式中的思想是：用一个内部类去实例化一个对象，避免一个类出现过多构造函数，

举一个例子：

    import lombok.*;
    @Data //生成getter,setter等函数
    @AllArgsConstructor //生成全参数构造函数
    @NoArgsConstructor//生成无参构造函数
    @Builder
    public class User {
        private String name;
        private String age;
        private String sex;
    }
我们来为User类赋值并打印一下结果：

    public  static   void  main(String[] args){
        User userAllArgs=new User("27","yiye","male");//有全参数构造函数可如此赋值
        User userNoArgs=new User();//有无参构造函数时可以如此使用
        userNoArgs= User.builder().age("26").name("yezi").sex("male").build();//有
        System.out.println("userAllArgs:"+userAllArgs);
        System.out.println("userNoArgs:"+userNoArgs);
    }

打印结果是：

    userAllArgs:User(name=27, age=yiye, sex=male)
    userNoArgs:User(name=yezi, age=26, sex=male)


# 无参构造函数失效的情况
***注意的是，同时使用@Data 和 @AllArgsConstructor 后 ，默认的无参构造函数失效，如果需要它，要重新设置 @NoArgsConstructor***
这里我们要回顾一下Java中无参构造函数的相关知识：

1. 如果一个类没有定义任何构造函数，那么该类会自动生成1个默认的构造函数。默认构造函数没有参数。

2. 如果一个类定义了构造函数，但这些构造函数都有参数，那么不会生成默认构造函数，也就是说此时类没有无参的构造函数。

这种错误会出现在一些场合，例如我们定义一个DTO

    @Data
    @AllArgsConstructor
    public class TemplateInfoResponseDTO {
        private String code;
        private String msg;
        private Boolean success;
        private String data;
    }

然后通过向某个已提供的接口发送post请求，然后将接收到的结果解析为TemplateInfoResponseDTO，

    String response = httpRequestHelper.sendPostWithLongWaitTime(BaseUrl + GETTEMPLATEINFO_URL, JSON.toJSONString(templateInfo));
            TemplateInfoResponseDTO templateInfoResponseDTO = JSON.parseObject(response, TemplateInfoResponseDTO.class);

但是返回的了错误信息是set property error, TemplateInfoResultVO#data，
查看该接口平台的日志可以看到，
com.alibaba.fastjson.JSONException:default constructor not found. class ...... TemplateInfoResponseDTO
说明TemplateInfoResponseDTO 没有默认的无参构造函数，所以string 转 object 失败了，因为同时使用@Data 和 @AllArgsConstructor 后 ，默认的无参构造函数失效，要么再加上@NoArgsConstructor注解，要么删除@AllArgsConstructor。


Lombok更多的注解使用方法参见参考资料【1】

# 参考资料：
【1】https://www.jianshu.com/p/365ea41b3573
