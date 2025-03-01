---
title: JAVA 学习笔记（IoC控制反转）

date: 2021-12-30 18:12:12  

categories: 2021年12月

tags: [Java, Spring]

---

Spring提供的容器又称为IoC容器，什么是IoC？

IoC全称Inversion of Control，直译为控制反转。IoC又称为依赖注入（DI：Dependency Injection）

<!-- more -->

[TOC]

# 控制反转[^1]

那么何谓IoC？在理解IoC之前，我们先看看通常的Java组件是如何协作的。

我们假定一个在线书店，通过BookService获取书籍：

```    
    public class BookService {
        private HikariConfig config = new HikariConfig();
        private DataSource dataSource = new HikariDataSource(config);
    
        public Book getBook(long bookId) {
            try (Connection conn = dataSource.getConnection()) {
                ...
                return book;
            }
        }
    }

```
为了从数据库查询书籍，BookService持有一个DataSource。为了实例化一个HikariDataSource，又不得不实例化一个HikariConfig。

现在，我们继续编写UserService获取用户：

```
    public class UserService {
        private HikariConfig config = new HikariConfig();
        private DataSource dataSource = new HikariDataSource(config);
    
        public User getUser(long userId) {
            try (Connection conn = dataSource.getConnection()) {
                ...
                return user;
            }
        }
    }
```
因为UserService也需要访问数据库，因此，我们不得不也实例化一个HikariDataSource。

在处理用户购买的CartServlet中，我们需要实例化UserService和BookService：

```
    public class CartServlet extends HttpServlet {
        private BookService bookService = new BookService();
        private UserService userService = new UserService();
    
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            long currentUserId = getFromCookie(req);
            User currentUser = userService.getUser(currentUserId);
            Book book = bookService.getBook(req.getParameter("bookId"));
            cartService.addToCart(currentUser, book);
            ...
        }
    }
```
类似的，在购买历史HistoryServlet中，也需要实例化UserService和BookService：
```
    public class HistoryServlet extends HttpServlet {
        private BookService bookService = new BookService();
        private UserService userService = new UserService();
    }
```
上述每个组件都采用了一种简单的通过new创建实例并持有的方式。仔细观察，会发现以下缺点：

实例化一个组件其实很难，例如，BookService和UserService要创建HikariDataSource，实际上需要读取配置，才能先实例化HikariConfig，再实例化HikariDataSource。

没有必要让BookService和UserService分别创建DataSource实例，完全可以共享同一个DataSource，但谁负责创建DataSource，谁负责获取其他组件已经创建的DataSource，不好处理。类似的，CartServlet和HistoryServlet也应当共享BookService实例和UserService实例，但也不好处理。

很多组件需要销毁以便释放资源，例如DataSource，但如果该组件被多个组件共享，如何确保它的使用方都已经全部被销毁？

随着更多的组件被引入，例如，书籍评论，需要共享的组件写起来会更困难，这些组件的依赖关系会越来越复杂。

测试某个组件，例如BookService，是复杂的，因为必须要在真实的数据库环境下执行。

从上面的例子可以看出，如果一个系统有大量的组件，其生命周期和相互之间的依赖关系如果由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。

因此，核心问题是：

- 谁负责创建组件？
- 谁负责根据依赖关系组装组件？
- 销毁时，如何按依赖顺序正确销毁？
解决这一问题的核心方案就是IoC。

传统的应用程序中，控制权在程序本身，程序的控制流程完全由开发者控制，例如：

CartServlet创建了BookService，在创建BookService的过程中，又创建了DataSource组件。这种模式的缺点是，一个组件如果要使用另一个组件，必须先知道如何正确地创建它。

在IoC模式下，控制权发生了反转，即从应用程序转移到了IoC容器，所有组件不再由应用程序自己创建和配置，而是由IoC容器负责，这样，应用程序只需要直接使用已经创建好并且配置好的组件。为了能让组件在IoC容器中被“装配”出来，需要某种“注入”机制，例如，BookService自己并不会创建DataSource，而是等待外部通过setDataSource()方法来注入一个DataSource：

```
public class BookService {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

不直接new一个DataSource，而是注入一个DataSource，这个小小的改动虽然简单，却带来了一系列好处：

- BookService不再关心如何创建DataSource，因此，不必编写读取数据库配置之类的代码；
- DataSource实例被注入到BookService，同样也可以注入到UserService，因此，共享一个组件非常简单；
- 测试BookService更容易，因为注入的是DataSource，可以使用内存数据库，而不是真实的MySQL配置。

# 依赖注入（DI：Dependency Injection）
因此，IoC又称为依赖注入（DI：Dependency Injection），它解决了一个最主要的问题：将组件的创建+配置与组件的使用相分离，并且，由IoC容器负责管理组件的生命周期。

因为IoC容器要负责实例化所有的组件，因此，有必要告诉容器如何创建组件，以及各组件的依赖关系。一种最简单的配置是通过XML文件来实现，例如：
```
    <beans>
        <bean id="dataSource" class="HikariDataSource" />
        <bean id="bookService" class="BookService">
            <property name="dataSource" ref="dataSource" />
        </bean>
        <bean id="userService" class="UserService">
            <property name="dataSource" ref="dataSource" />
        </bean>
    </beans>
```
上述XML配置文件指示IoC容器创建3个JavaBean组件，并把id为dataSource的组件通过属性dataSource（即调用setDataSource()方法）注入到另外两个组件中。

在Spring的IoC容器中，我们把所有组件统称为JavaBean，即配置一个组件就是配置一个Bean。

# 依赖注入方式
我们从上面的代码可以看到，依赖注入可以通过set()方法实现。但依赖注入也可以通过构造方法实现。

很多Java类都具有带参数的构造方法，如果我们把BookService改造为通过构造方法注入，那么实现代码如下：
```
    public class BookService {
        private DataSource dataSource;
    
        public BookService(DataSource dataSource) {
            this.dataSource = dataSource;
        }
    }
```
Spring的IoC容器同时支持属性注入和构造方法注入，并允许混合使用。

# 无侵入容器
在设计上，Spring的IoC容器是一个高度可扩展的无侵入容器。所谓无侵入，是指应用程序的组件无需实现Spring的特定接口，或者说，组件根本不知道自己在Spring的容器中运行。这种无侵入的设计有以下好处：

- 应用程序组件既可以在Spring的IoC容器中运行，也可以自己编写代码自行组装配置；
- 测试的时候并不依赖Spring容器，可单独进行测试，大大提高了开发效率。

# 控制反转（IOC）和依赖注入（DI）的区别[^2]

IOC   inversion of control  控制反转

DI   Dependency Injection  依赖注入

要理解这两个概念，首先要搞清楚以下几个问题：

- 参与者都有谁？
- 依赖：谁依赖于谁？为什么需要依赖？
- 注入：谁注入于谁？到底注入什么？
- 控制反转：谁控制谁？控制什么？为何叫反转（有反转就应该有正转了）？
- 依赖注入和控制反转是同一概念吗？


下面就来简要的回答一下上述问题，把这些问题搞明白了，IoC/DI也就明白了。

## 参与者都有谁：

一般有三方参与者，一个是某个对象；一个是IoC/DI的容器；另一个是某个对象的外部资源。

又要名词解释一下，某个对象指的就是任意的、普通的Java对象;IoC/DI的容器简单点说就是指用来实现IoC/DI功能的一个框架程序；对象的外部资源指的就是对象需要的，但是是从对象外部获取的，都统称资源，比如：对象需要的其它对象、或者是对象需要的文件资源等等。

## 谁依赖于谁：

当然是某个对象依赖于IoC/DI的容器

## 为什么需要依赖：

对象需要IoC/DI的容器来提供对象需要的外部资源

## 谁注入于谁：

很明显是IoC/DI的容器 注入 某个对象

## 到底注入什么：

就是注入某个对象所需要的外部资源

## 谁控制谁：

当然是IoC/DI的容器来控制对象了

## 控制什么：

主要是控制对象实例的创建

## 为何叫反转：

反转是相对于正向而言的，那么什么算是正向的呢？考虑一下常规情况下的应用程序，如果要在A里面使用C，你会怎么做呢？当然是直接去创建C的对象，也就是说，是在A类中主动去获取所需要的外部资源C，这种情况被称为正向的。那么什么是反向呢？就是A类不再主动去获取C，而是被动等待，等待IoC/DI的容器获取一个C的实例，然后反向的注入到A类中。

## 依赖注入和控制反转是同一概念吗？

根据上面的讲述，应该能看出来，依赖注入和控制反转是对同一件事情的不同描述，从某个方面讲，就是它们描述的角度不同。

依赖注入是从应用程序的角度在描述，可以把依赖注入描述完整点：

应用程序依赖容器创建并注入它所需要的外部资源；

而控制反转是从容器的角度在描述，描述完整点：

容器控制应用程序，由容器反向的向应用程序注入应用程序所需要的外部资源。


## 总结：

其实IoC/DI对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IoC/DI思想中，应用程序就变成被动的了，被动的等待IoC/DI容器来创建并注入它所需要的资源了。

这么小小的一个改变其实是编程思想的一个大进步，这样就有效的分离了对象和它所需要的外部资源，使得它们松散耦合，有利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活

## 接下演示一下依赖注入机制的过程
<details>
    <summary>待注入的业务对象Content.java</summary>

```

package com.zj.ioc.di.ctor;
import com.zj.ioc.di.Content;
 
public class MyBusiness {
    private Content myContent;
 
    public MyBusiness(Content content) {
       myContent = content;
    }
   
    public void doBusiness(){
       myContent.BusniessContent();
    }
   
    public void doAnotherBusiness(){
       myContent.AnotherBusniessContent();
    }
}
 
```
</details>


MyBusniess类展示了一个业务组件，它的实现需要对象Content的注入。分别演示构造子注入（Constructor Injection），设值注入（Setter Injection）和接口注入（Interface Injection）三种方式。

<details>
    <summary>构造子注入（Constructor Injection）MyBusiness.java</summary>

```

package com.zj.ioc.di.ctor;
import com.zj.ioc.di.Content;
 
public class MyBusiness {
    private Content myContent;
 
    public MyBusiness(Content content) {
       myContent = content;
    }
   
    public void doBusiness(){
       myContent.BusniessContent();
    }
   
    public void doAnotherBusiness(){
       myContent.AnotherBusniessContent();
    }
}


```
</details>


<details>
    <summary>设值注入（Setter Injection） MyBusiness.java</summary>

```

package com.zj.ioc.di.iface;
import com.zj.ioc.di.Content;
 
public interface InContent {
    void createContent(Content content);
}


```
</details>


<details>
    <summary>设置注入接口InContent.java</summary>

```
package com.zj.ioc.di.iface;
import com.zj.ioc.di.Content;
 
public interface InContent {
    void createContent(Content content);
}


```
</details>





<details>
    <summary>接口注入（Interface Injection）MyBusiness.java</summary>

```

    
package com.zj.ioc.di.iface;
import com.zj.ioc.di.Content;
 
public class MyBusiness implements InContent{
    private Content myContent;
 
    public void createContent(Content content) {
       myContent = content;
    }
   
    public void doBusniess(){
       myContent.BusniessContent();
    }
   
    public void doAnotherBusniess(){
       myContent.AnotherBusniessContent();
    }
}

```
</details>


[^1]:https://www.liaoxuefeng.com/wiki/1252599548343744/1282381977747489

[^2]:https://www.cnblogs.com/jett010/p/10984020.html