---
title: JAVA 学习笔记（Spring 原理）

date: 2021-12-30 10:12:12  

categories: 2021年12月

tags: [Java, Spring]

---

Spring是一个全面的、企业应用开发一站式的解决方案，贯穿表现层、业务层、持久层。但是Spring仍然可以和其他的框架无缝整合。

<!-- more -->

[TOC]

# Spring 特点

## 轻量级
从大小与开销两方面而言Spring都是轻量级的，完整的Spring框架可以在一个大小只有1M多的JAR文件里发布，并且所需的处理开销也是微不足道的。所以可以在小型设备中使用。

此外，Spring是非侵入式的：典型的，Spring应用中的对象不依赖于Spring的特定类。

## 控制反转
控制反转（IOC）：通过控制反转实现解耦，大大降低代码量，促进了低耦合

当应用了IOC，一个对象依赖的其他对象会通过被动的方式传递进来，而不是这个对象自己创建或者查找依赖对象。

## 面向切面编程（AOP）
Spring支持面向切面的编程，并且把应用业务逻辑和系统服务分开。

在事务处理、日志管理、权限控制、异常处理这些板块有很明显的优势

## 容器

Spring提供了容器功能，容器可以管理对象的生命周期、对象与对象间的关系、我们可以通过编写XML来设置对象关系和初始值，这样容器在启动之后，所有的对象都直接可以使用，不用编写任何编码来产生对象。Spring有两种不同的容器：Bean工厂以及应用上下文

Spring包含并管理应用对象的配置和生命周期，在这个意义上它是一种容齐，你可以配置你的每个bean如何被创建。。。基于一个可配置原型，你的bean可以创建一个单独的实例或者每次需要时都生成一个新的实例以及它们是如何关联的。

## 框架集合
Spring可以将简单的组件配置，组合成为复杂的应用。

在Spring中，应用对象被声明式的组合，典型的是在一个XML文件里。

Spring也提供了很多基础功能（事务管理，持久化框架集成等），将应用逻辑的开发留给开发者。

可以集成mybatis， hibernate等框架

# Spring 核心组件

# Spring 常用模块

## 核心容器
核心容器提供Spring框架的基本功能，核心容器的主要组件是BeanFactory，它是工厂模式的实现，BeanFactory使用控制反转（IOC）模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。

## Spring上下文
Spring上下文是一个配置文件，向Spring框架提供上下文信息，Spring上下文包括企业服务，例如JNDI，EJB，电子邮件，国际化，校验和调度功能。

## Spring AOP
通过配置管理特性，Spring AOP模块直接将面向切面的编程功能集成到了Spring框架中，可以将一些通用任务，如安全，事务，日志等集中进行管理，提高了复用性和管理的便捷性。

## Spring DAO
为JDBC DAO抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商跑出的错误消息，异常层次结构简化了错误处理，并且极大地降低了需要便携的异常代码数量（例如打开和关闭连接）。Spring DAO的面向JDBC的异常遵从通用的DAO异常层次结构。

## Spring ORM
Spring框架插入了若干个ORM框架，从而提供了ORM的对象关系工具。其中包括JDO，Hiberinate和iBatis SQL Map。所有这些都遵从Spring的通用事务和DAO异常层次结构。

## Spring Web模块
WEb上下文模块建立在应用恒旭上下文模块之上，为基于Web的应用程序提供了上下文。所以，Spring框架支持与Jakarta Struts的集成。Web模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。

## Spring MVC框架
MVC框架是一个全功能的构建Web应用程序的MVC实现。通过策略接口，MVC框架变成为高度可配置的。MVC容纳了大量视图技术。MVC容纳了大量视图技术，其中包括JSP，Velocity，Tiles，iText和POI。

# Spring 主要包

## Core Container 核心容器
容器是Spring的核心部分，Core Container 模块是Spring框架的基础，所有模块都构建于核心模块之上。

Beans ： Beans模块是所有应用都要用到的，它包含访问配置文件、创建和管理bean以及进行Inversion of Control / Depen-dency Injection（IoC/DI）操作相关的所有类。

Core  ： Core模块主要包含Spring框架基本的核心工具类，Spring的其他组件要都要使用到这个包里的类，Core模块是其他组件的基本核心。当然你也可以在自己的应用系统中使用这些工具类。

Context  :  Spring的上下文即IoC容器，通过上下文可以获得容器中的Bean。ApplicationContext接口是Context模块的关键。  Context模块构建于Core和Beans模块基础之上，提供了一种类似于JNDI注册器的框架式的对象访问方法。

SpEl  :  Expression Language模块提供了一个强大的表达式语言用于在运行时查询和操纵对象。

# Spring 常用注解
bean 注入与装配的的方式有很多种，可以通过 xml，get set 方式，构造函数或者注解等。简单易用的方式就是使用 Spring 的注解了，Spring 提供了大量的注解方式。

## 声明bean的注解
@Component 泛指组件，没有明确的角色，当组件不好归类的时候，我们可以使用这个注解进行标注。

@Service 在业务逻辑层使用（service层）

@Repository 在数据访问层使用（dao层）

@Controller 在展现层使用，控制器的声明（C），用于标注控制层组件。

## 注入bean的注解
@Autowired：由Spring提供

@Inject：由JSR-330提供

@Resource：由JSR-250提供

都可以注解在set方法和属性上，推荐注解在属性上（一目了然，少写代码）。

## java配置类相关注解
@Configuration 声明当前类为配置类，相当于xml形式的Spring配置（类上）

@Bean 注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式（方法上）

@Configuration 声明当前类为配置类，其中内部组合了@Component注解，表明这个类是一个bean（类上）

@ComponentScan 用于对Component进行扫描，相当于xml中的（类上）

@WishlyConfiguration 为@Configuration与@ComponentScan的组合注解，可以替代这两个注解

## 切面（AOP）相关注解
Spring支持AspectJ的注解式切面编程。

@Aspect 声明一个切面（类上） 
使用@After、@Before、@Around定义建言（advice），可直接将拦截规则（切点）作为参数。

@After 在方法执行之后执行（方法上） 

@Before 在方法执行之前执行（方法上） 

@Around 在方法执行之前与之后执行（方法上）

@PointCut 声明切点 
在java配置类中使用@EnableAspectJAutoProxy注解开启Spring对AspectJ代理的支持（类上）

## @Bean的属性支持
@Scope 设置Spring容器如何新建Bean实例（方法上，得有@Bean） 
其设置类型包括：

Singleton （单例,一个Spring容器中只有一个bean实例，默认模式）, 

Protetype （每次调用新建一个bean）, 

Request （web项目中，给每个http request新建一个bean）, 

Session （web项目中，给每个http session新建一个bean）, 

GlobalSession（给每一个 global http session新建一个Bean实例）

@StepScope 在Spring Batch中还有涉及

@PostConstruct 由JSR-250提供，在构造函数执行完之后执行，等价于xml配置文件中bean的initMethod

@PreDestory 由JSR-250提供，在Bean销毁之前执行，等价于xml配置文件中bean的destroyMethod

## @Value注解
@Value 为属性注入值（属性上） 
支持如下方式的注入：

》注入普通字符
    
    @Value("Michael Jackson")
    String name;
》注入操作系统属性

    @Value("#{systemProperties['os.name']}")
    String osName;
》注入表达式结果

    @Value("#{ T(java.lang.Math).random() * 100 }") 
    String randomNumber;
》注入其它bean属性

    @Value("#{domeClass.name}")
    String name;
》注入文件资源

    @Value("classpath:com/hgs/hello/test.txt")
    String Resource file;
》注入网站资源

    @Value("http://www.cznovel.com")
    Resource url;
》注入配置文件

    @Value("${book.name}")
    String bookName;
注入配置使用方法： 

① 编写配置文件（test.properties）

    book.name=《三体》

② @PropertySource 加载配置文件(类上)

    @PropertySource("classpath:com/hgs/hello/test/test.propertie")

③ 还需配置一个PropertySourcesPlaceholderConfigurer的bean。

## 环境切换
@Profile 通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。（类或方法上）

@Conditional Spring4中可以使用此注解定义条件话的bean，通过实现Condition接口，并重写matches方法，从而决定该bean是否被实例化。（方法上）

## 异步相关
@EnableAsync 配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口（类上）

@Async 在实际执行的bean方法使用该注解来申明其是一个异步任务（方法上或类上所有的方法都将异步，需要@EnableAsync开启异步任务）

## 定时任务相关
@EnableScheduling 在配置类上使用，开启计划任务的支持（类上）

@Scheduled 来申明这是一个任务，包括cron,fixDelay,fixRate等类型（方法上，需先开启计划任务的支持）

## @Enable*注解说明
这些注解主要用来开启对xxx的支持。 
@EnableAspectJAutoProxy 开启对AspectJ自动代理的支持

@EnableAsync 开启异步方法的支持

@EnableScheduling 开启计划任务的支持

@EnableWebMvc 开启Web MVC的配置支持

@EnableConfigurationProperties 开启对@ConfigurationProperties注解配置Bean的支持

@EnableJpaRepositories 开启对SpringData JPA Repository的支持

@EnableTransactionManagement 开启注解式事务的支持

@EnableTransactionManagement 开启注解式事务的支持

@EnableCaching 开启注解式的缓存支持

## 测试相关注解

@RunWith 运行器，Spring中通常用于对JUnit的支持

@ContextConfiguration 用来加载配置ApplicationContext，其中classes属性用来加载配置类

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(classes={TestConfig.class})
    public class KjtTest {
     
        private static Logger logger = LoggerFactory.getLogger("KjtTest");
     
        @Autowired
        Service service;
     
     
        @Test
        public void test() {
            
        }
    }
 

## SpringMVC部分

@EnableWebMvc 在配置类中开启WebMVC的配置支持，如一些ViewResolver或者MessageConverter等，若无此句，重写WebMvcConfigurerAdapter方法（用于对SpringMVC的配置）。

@Controller 声明该类为SpringMVC中的Controller

@RequestMapping 用于映射Web请求，包括访问路径和参数（类或方法上）

@ResponseBody 支持将返回值放在response内，而不是一个页面，通常用户返回json数据（返回值旁或方法上）

@RequestBody 允许request的参数在request体中，而不是在直接连接在地址后面。（放在参数前）

@PathVariable 用于接收路径参数，比如@RequestMapping(“/hello/{name}”)申明的路径，将注解放在参数中前，即可获取该值，通常作为Restful的接口实现方法。

@RestController 该注解为一个组合注解，相当于@Controller和@ResponseBody的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody。

@ControllerAdvice 通过该注解，我们可以将对于控制器的全局配置放置在同一个位置，注解了@Controller的类的方法可使用@ExceptionHandler、@InitBinder、@ModelAttribute注解到方法上， 

这对所有注解了 @RequestMapping的控制器内的方法有效。

@ExceptionHandler 用于全局处理控制器里的异常

@InitBinder 用来设置WebDataBinder，WebDataBinder用来自动绑定前台请求参数到Model中。

@ModelAttribute 本来的作用是绑定键值对到Model里，在@ControllerAdvice中是让全局的@RequestMapping都能获得在此处设置的键值对。


# Spring 第三方框架结合

## 权限

### shiro 

java的一个安全框架

认证、授权、加密、会话管理、与Web集成、缓存

## 缓存
### Ehcache
是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider

### redis
一个开源的使用ANSIC语言便携、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库

## 持久层框架

### Hibernate

一个开放源代码的对象关系映射框架，它对JDBC进行了非常轻量级的对象封装，它将POJO与数据库表建立映射关系，是一个全自动的orm框架

### Mybatis
是支持普通SQL查询，存储过程和高级映射的优秀持久层框架

## 定时任务

### quartz
一个开源的作业调度框架，由java编写，在.NET平台为Quartz.Net ，通过Quart可以快速完成任务调度的工作

### Spring-Task
可以将它比作一个轻量级的Quartz，而且使用起来很简单，除spring相关的包外不需要额外的包，而且支持注解和配置文件两种形式

## 校验框架
### Hibernate validator

常用来验证bean的字段，基于注解，方便快捷高效

### Oval

一个可扩展的Java对象数据验证框架，验证的规则可以通过配置文件、Annotation、POJOs进行设定，可以使用纯Java语言、JavaScript，Groovy、BeanShell等进行规则的编写。

# Spring IOC 原理 (Inversion of Control，控制反转)
## 概念
Spring 通过一个配置文件描述 Bean 及 Bean 之间的依赖关系，利用 Java 语言的反射功能实例化Bean 并建立 Bean 之间的依赖关系。 Spring 的 IoC 容器在完成这些底层工作的基础上，还提供了 Bean 实例缓存、生命周期管理、 Bean 实例代理、事件发布、资源装载等高级服务。

## Spring 容器高层视图
Spring 启动时读取应用程序提供的Bean配置信息，并在Spring容器中生成一份相应的Bean配置注册表，然后根据这张注册表实例化 Bean，装配好 Bean 之间的依赖关系，为上层应用提供准备就绪的运行环境。其中 Bean 缓存池为 HashMap 实现

## IOC 容器实现
### BeanFactory-框架基础设施
BeanFactory 是 Spring 框架的基础设施，面向Spring本身；ApplicationContext面向使用Spring框架的开发者，几乎所有的应用场合我们都直接使用 ApplicationContext 而非底层的 BeanFactory


#### BeanDefinitionRegistry 注册表

1. Spring 配置文件中每一个节点元素在 Spring 容器里都通过一个 BeanDefinition 对象表示，它描述了 Bean 的配置信息。而 BeanDefinitionRegistry 接口提供了向容器手工注册BeanDefinition 对象的方法。

#### BeanFactory 顶层接口

2. 位于类结构树的顶端 ，它最主要的方法就是 getBean(String beanName)，该方法从容器中返回特定名称的Bean，BeanFactory的功能通过其他的接口得到不断扩展：

#### ListableBeanFactory

3. 该接口定义了访问容器中 Bean基本信息的若干方法，如查看Bean的个数、获取某一类型Bean的配置名、查看容器中是否包括某一 Bean 等方法；

#### HierarchicalBeanFactory 父子级联

4. 父子级联 IoC 容器的接口，子容器可以通过接口方法访问父容器； 通过HierarchicalBeanFactory 接口， Spring 的 IoC 容器可以建立父子层级关联的容器体系，子容器可以访问父容器中的 Bean，但父容器不能访问子容器的 Bean。Spring 使用父子容器实现了很多功能，比如在 Spring MVC 中，展现层 Bean 位于一个子容器中，而业务层和持久层的Bean位于父容器中。这样，展现层Bean就可以引用业务层和持久层的Bean，而业务层和持久层的 Bean 则看不到展现层的 Bean。

#### ConfigurableBeanFactory

5. 是一个重要的接口，增强了 IoC 容器的可定制性，它定义了设置类装载器、属性编辑器、容器初始化后置处理器等方法；

####  AutowireCapableBeanFactory 自动装配
6. 定义了将容器中的 Bean 按某种规则（如按名字匹配、按类型匹配等）进行自动装配的方法；

#### SingletonBeanRegistry 运行期间注册单例 Bean
7. 定义了允许在运行期间向容器注册单实例 Bean 的方法；对于单实例（ singleton）的 Bean来说，BeanFactory 会缓存 Bean 实例，所以第二次使用 getBean() 获取 Bean 时将直接从IoC 容器的缓存中获取 Bean 实例。Spring 在 DefaultSingletonBeanRegistry 类中提供了一个用于缓存单实例 Bean 的缓存器，它是一个用 HashMap 实现的缓存器，单实例的 Bean 以beanName 为键保存在这个 HashMap 中。

####  依赖日志框框
8. 在初始化 BeanFactory 时，必须为其提供一种日志框架，比如使用 Log4J， 即在类路径下提供 Log4J 配置文件，这样启动 Spring 容器才不会报错。

### ApplicationContext 面向开发应用
ApplicationContext 由 BeanFactory 派 生 而 来 ， 提 供 了 更 多 面 向 实 际 应 用 的 功 能 。ApplicationContext 继承了 HierarchicalBeanFactory 和 ListableBeanFactory 接口，在此基础上，还通过多个其他的接口扩展了 BeanFactory 的功能：

1. ClassPathXmlApplicationContext：默认从类路径加载配置文件
2. FileSystemXmlApplicationContext：默认从文件系统中装载配置文件
3. ApplicationEventPublisher：让容器拥有发布应用上下文事件的功能，包括容器启动事件、关闭事件等。
4. MessageSource：为应用提供 i18n 国际化消息访问的功能；
5. ResourcePatternResolver ： 所 有 ApplicationContext 实现类都实现了类似于PathMatchingResourcePatternResolver 的功能，可以通过带前缀的 Ant 风格的资源文件路径装载 Spring 的配置文件。
6. LifeCycle：该接口是 Spring 2.0 加入的，该接口提供了 start()和 stop()两个方法，主要用于控制异步处理过程。在具体使用时，该接口同时被 ApplicationContext 实现及具体Bean 实现， ApplicationContext 会将 start/stop 的信息传递给容器中所有实现了该接口的 Bean，以达到管理和控制 JMX、任务调度等目的。
7. ConfigurableApplicationContext 扩展于 ApplicationContext，它新增加了两个主要的方法： refresh()和 close()，让 ApplicationContext 具有启动、刷新和关闭应用上下文的能力。在应用上下文关闭的情况下调用refresh()即可启动应用上下文，在已经启动的状态下，调用refresh()则清除缓存并重新装载配置信息，而调用 close()则可关闭应用上下文。

### WebApplication 体系架构
WebApplicationContext 是专门为 Web 应用准备的，它允许从相对于 Web 根目录的路径中装载配置文件完成初始化工作。从 WebApplicationContext 中可以获得ServletContext 的引用，整个 Web 应用上下文对象将作为属性放置到 ServletContext中，以便 Web 应用环境可以访问 Spring 应用上下文。

## Spring Bean 作用域
Spring 3 中为 Bean 定义了 5 中作用域，分别为 singleton（单例）、prototype（原型）、request、session 和 global session，5 种作用域说明如下：

### singleton：单例模式（多线程下不安全）
1. singleton：单例模式，Spring IoC 容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，始终指向同一对象。该模式在多线程下是不安全的。Singleton 作用域是Spring 中的缺省作用域，也可以显示的将 Bean 定义为 singleton 模式，配置为：

<bean id="userDao" class="com.ioc.UserDaoImpl" scope="singleton"/>

### prototype:原型模式每次使用时创建
2. prototype:原型模式，每次通过 Spring 容器获取 prototype 定义的 bean 时，容器都将创建一个新的 Bean 实例，每个 Bean 实例都有自己的属性和状态，而 singleton 全局只有一个对象。根据经验，对有状态的bean使用prototype作用域，而对无状态的bean使用singleton
作用域。

### Request：一次 request 一个实例
3. request：在一次 Http 请求中，容器会返回该 Bean 的同一实例。而对不同的 Http 请求则会产生新的 Bean，而且该 bean 仅在当前 Http Request 内有效,当前 Http 请求结束，该 bean实例也将会被销毁。

<bean id="loginAction" class="com.cnblogs.Login" scope="request"/>

### session（会话）
4. session：在一次 Http Session 中，容器会返回该 Bean 的同一实例。而对不同的 Session 请求则会创建新的实例，该 bean 实例仅在当前 Session 内有效。同 Http 请求相同，每一次session请求创建新的实例，而不同的实例之间不共享属性，且实例仅在自己的session请求内有效，请求结束，则实例将被销毁。

<bean id="userPreference" class="com.ioc.UserPreference" scope="session"/>

### global Session
5. global Session：在一个全局的 Http Session 中，容器会返回该 Bean 的同一个实例，仅在使用 portlet context 时有效

## Spring Bean 生命周期
### 实例化
1. 实例化一个 Bean，也就是我们常说的 new。

### IOC 依赖注入
2. 按照 Spring 上下文对实例化的 Bean 进行配置，也就是 IOC 注入。

### setBeanName 实现
3. 如果这个 Bean 已经实现了 BeanNameAware 接口，会调用它实现的 setBeanName(String)方法，此处传递的就是 Spring 配置文件中 Bean 的 id 值

### BeanFactoryAware 实现
4. 如果这个 Bean 已经实现了 BeanFactoryAware接口，会调用它实现的setBeanFactory，setBeanFactory(BeanFactory)传递的是Spring工厂自身（可以用这个方式来获取其它 Bean，只需在 Spring 配置文件中配置一个普通的 Bean 就可以）。

### ApplicationContextAware 实现
5. 如果这个 Bean 已经实现了 ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文（同样这个方式也可以实现步骤 4 的内容，但比 4 更好，因为 ApplicationContext 是 BeanFactory 的子接口，有更多的实现方法）

### postProcessBeforeInitialization 接口实现-初始化预处理
6. 如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用 postProcessBeforeInitialization(Object obj, String s)方法，BeanPostProcessor 经常被用
作是 Bean 内容的更改，并且由于这个是在 Bean 初始化结束时调用那个的方法，也可以被应用于内存或缓存技术。

### init-method
7. 如果 Bean 在 Spring 配置文件中配置了 init-method 属性会自动调用其配置的初始化方法。

### postProcessAfterInitialization
8. 如果这个 Bean 关联了 BeanPostProcessor 接口，将会调用postProcessAfterInitialization(Object obj, String s)方法。

注：以上工作完成以后就可以应用这个 Bean 了，那这个 Bean是一个Singleton的，所以一般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在 Spring 配置文件中也可以配置非 Singleton。

### Destroy 过期自动清理阶段
9. 当 Bean 不再需要时，会经过清理阶段，如果 Bean 实现了 DisposableBean 这个接口，会调用那个其实现的 destroy()方法；

### destroy-method 自配置清理
10. 最后，如果这个 Bean 的 Spring 配置中配置了 destroy-method 属性，会自动调用其配置的销毁方法。


11. bean 标签有两个重要的属性（init-method 和 destroy-method）。用它们你可以自己定制初始化和注销方法。它们也有相应的注解（@PostConstruct 和@PreDestroy）。

<bean id="" class="" init-method="初始化方法" destroy-method="销毁方法">


## Spring 依赖注入四种方式
### 构造器注入

<details>
    <summary>构造器注入</summary>
    
```

/*带参数，方便利用构造器进行注入*/
 public CatDaoImpl(String message){
    this. message = message;
 }
 
<bean id="CatDaoImpl" class="com.CatDaoImpl">
    <constructor-arg value=" message "></constructor-arg>
</bean>

```
</details>


### setter 方法注入


<details>
    <summary>setter 方法注入</summary>
    
```
public class Id {
     private int id;
     public int getId() { return id; }
     public void setId(int id) { this.id = id; }
}
<bean id="id" class="com.id "> <property name="id" value="123"></property> </bean>

```
</details>

### 静态工厂注入
静态工厂顾名思义，就是通过调用静态工厂的方法来获取自己需要的对象，为了让spring管理所有对象，我们不能直接通过"工程类.静态方法()"来获取对象，而是依然通过 spring 注入的形式获取：



<details>
    <summary></summary>
    
```

    public class DaoFactory { //静态工厂
        public static final FactoryDao getStaticFactoryDaoImpl(){
            return new StaticFacotryDaoImpl();
        }
    }
    public class SpringAction {
        private FactoryDao staticFactoryDao; //注入对象
        //注入对象的 set 方法
        public void setStaticFactoryDao(FactoryDao staticFactoryDao) {
            this.staticFactoryDao = staticFactoryDao;
        }
    }
    //factory-method="getStaticFactoryDaoImpl"指定调用哪个工厂方法
     <bean name="springAction" class=" SpringAction" >
     <!--使用静态工厂的方法注入对象,对应下面的配置文件-->
     <property name="staticFactoryDao" ref="staticFactoryDao"></property>
     </bean>
     <!--此处获取对象的方式是从工厂类中获取静态方法-->
    <bean name="staticFactoryDao" class="DaoFactory" factory-method="getStaticFactoryDaoImpl"></bean>
```
</details>


### 实例工厂
实例工厂的意思是获取对象实例的方法不是静态的，所以你需要首先 new 工厂类，再调用普通的实例方法：



<details>
    <summary>实例工厂</summary>
    
```
    
    public class DaoFactory { //实例工厂
            public FactoryDao getFactoryDaoImpl(){
                return new FactoryDaoImpl();
            }
        }
        public class SpringAction {
            private FactoryDao factoryDao; //注入对象
            public void setFactoryDao(FactoryDao factoryDao) {
                this.factoryDao = factoryDao;
            }
        }
         <bean name="springAction" class="SpringAction">
         <!--使用实例工厂的方法注入对象,对应下面的配置文件-->
         <property name="factoryDao" ref="factoryDao"></property>
         </bean>
         <!--此处获取对象的方式是从工厂类中获取实例方法-->
        <bean name="daoFactory" class="com.DaoFactory"></bean>
        <bean name="factoryDao" factory-bean="daoFactory"
                factory-method="getFactoryDaoImpl"></bean>

```
</details>



## 5 种不同方式的自动装配
Spring 装配包括手动装配和自动装配，手动装配是有基于 xml 装配、构造方法、setter 方法等自动装配有五种自动装配的方式，可以用来指导 Spring 容器用自动装配方式来进行依赖注入。

1. no：默认的方式是不进行自动装配，通过显式设置 ref 属性来进行装配。
2. byName：通过参数名 自动装配，Spring 容器在配置文件中发现 bean 的 autowire 属性被设置成 byname，之后容器试图匹配、装配和该 bean 的属性具有相同名字的 bean。
3. byType：通过参数类型自动装配，Spring 容器在配置文件中发现 bean 的 autowire 属性被设置成 byType，之后容器试图匹配、装配和该 bean 的属性具有相同类型的 bean。如果有多个 bean 符合条件，则抛出错误。
4. constructor：这个方式类似于 byType， 但是要提供给构造器参数，如果没有确定的带参数的构造器参数类型，将会抛出异常。
5. autodetect：首先尝试使用 constructor 来自动装配，如果无法工作，则使用 byType 方式。


# Spring AOP 原理（Aspect Oriented Programming 面向切面编程）
## 概念
"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

使用"横切"技术，AOP 把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。

AOP 主要应用场景有：

1. Authentication 权限
2. Caching 缓存
3. Context passing 内容传递
4. Error handling 错误处理
5. Lazy loading 懒加载
6. Debugging 调试
7. logging, tracing, profiling and monitoring 记录跟踪 优化 校准
8. Performance optimization 性能优化
9. Persistence 持久化
10. Resource pooling 资源池
11. Synchronization 同步
12. Transactions 事务

## AOP 核心概念

1、切面（aspect）：类是对物体特征的抽象，切面就是对横切关注点的抽象

一个关注点的模块化，这个关注点可能会横切多个对象。事务管理是J2EE应用中一个关于横切关注点的很好的例子。在Spring AOP中，切面可以使用基于模式或者基于@Aspect注解的方式实现，可以简单地认为, 使用 @Aspect 注解的类就是切面。

2、横切关注点：对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点。


3、连接点（join point）：被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器。

在程序执行过程中某个特定的点，比如某方法调用的时候或者处理异常的时候。在Spring AOP中，一个连接点总是表示一个方法的执行。

4、切入点（point cut）：对连接点进行拦截的定义

匹配连接点的断言。通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行（例如，当执行某个特定名称的方法时）。切入点表达式如何和连接点匹配是AOP的核心：Spring缺省使用AspectJ切入点语法。

5、通知（advice）：所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类。

在切面的某个特定的连接点上执行的动作，许多AOP框架（包括Spring）都是以拦截器做通知模型，并维护一个以连接点为中心的拦截器链。
    


6、目标对象（Target Object)：代理的目标对象

被一个或多个切面所通知的对象。也被成为被通知（advised）对象,既然Spring AOP是通过运行时代理所实现的，这个对象永远是一个被代理(Proxied)对象

7、织入（weaving）：将切面应用到目标对象并导致代理对象创建的过程

把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象，这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其它纯Java AOP框架一样，在运行时完成织入。

8、引入（introduction）：在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段。

用来给一个类型声明额外的方法或属性（也被成为连接类型声明（inter-type declaration)).Spring允许引入新的接口及对应的实现到任何被代理的对象。例如可以使用引入来使一个bean实现IsModified接口，一边简化缓存机制。

9. AOP代理（AOP Proxy）：AOP框架创建的对象，用来实现切面契约（例如通知方法执行等等）在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。


参考：https://segmentfault.com/a/1190000007469968

### 通知类型

#### 前置通知（Before advice）

在某连接点之前执行的通知，但这个通知不能组织连接点之前的执行流程（除非它抛出一个异常）

#### 后置通知（After returning advice）

在某连接点正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。

#### 异常通知（After throwing advice）

在方法抛出异常退出时执行的通知

#### 最终通知（After/Finally advice）

当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）

#### 环绕通知（Around Advice）

包围一个连接点的通知，如方法调用，这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。


## 彻底理解 aspect, join point, point cut, advice
参考自：https://segmentfault.com/a/1190000007469968

下面以一个简单的例子来比喻一下 AOP 中 aspect, jointpoint, pointcut 与 advice 之间的关系.

让我们来假设一下, 从前有一个叫爪哇的小县城, 在一个月黑风高的晚上, 这个县城中发生了命案. 作案的凶手十分狡猾, 现场没有留下什么有价值的线索. 不过万幸的是, 刚从隔壁回来的老王恰好在这时候无意中发现了凶手行凶的过程, 但是由于天色已晚, 加上凶手蒙着面, 老王并没有看清凶手的面目, 只知道凶手是个男性, 身高约七尺五寸. 爪哇县的县令根据老王的描述, 对守门的士兵下命令说: 凡是发现有身高七尺五寸的男性, 都要抓过来审问. 士兵当然不敢违背县令的命令, 只好把进出城的所有符合条件的人都抓了起来.

来让我们看一下上面的一个小故事和 AOP 到底有什么对应关系.

首先我们知道, 在 Spring AOP 中 join point 指代的是所有方法的执行点, 而 point cut 是一个描述信息, 它修饰的是 join point, 通过 point cut, 我们就可以确定哪些 join point 可以被织入 Advice. 对应到我们在上面举的例子, 我们可以做一个简单的类比, join point 就相当于 爪哇的小县城里的百姓, point cut 就相当于 老王所做的指控, 即凶手是个男性, 身高约七尺五寸, 而 advice 则是施加在符合老王所描述的嫌疑人的动作: 抓过来审问.
为什么可以这样类比呢?

join point --> 爪哇的小县城里的百姓: 因为根据定义, join point 是所有可能被织入 advice 的候选的点, 在 Spring AOP中, 则可以认为所有方法执行点都是 join point. 而在我们上面的例子中, 命案发生在小县城中, 按理说在此县城中的所有人都有可能是嫌疑人.

point cut --> 男性, 身高约七尺五寸: 我们知道, 所有的方法(joint point) 都可以织入 advice, 但是我们并不希望在所有方法上都织入 advice, 而 pointcut 的作用就是提供一组规则来匹配joinpoint, 给满足规则的 joinpoint 添加 advice. 同理, 对于县令来说, 他再昏庸, 也知道不能把县城中的所有百姓都抓起来审问, 而是根据凶手是个男性, 身高约七尺五寸, 把符合条件的人抓起来. 在这里 凶手是个男性, 身高约七尺五寸 就是一个修饰谓语, 它限定了凶手的范围, 满足此修饰规则的百姓都是嫌疑人, 都需要抓起来审问.

advice --> 抓过来审问, advice 是一个动作, 即一段 Java 代码, 这段 Java 代码是作用于 point cut 所限定的那些 join point 上的. 同理, 对比到我们的例子中, 抓过来审问 这个动作就是对作用于那些满足 男性, 身高约七尺五寸 的爪哇的小县城里的百姓.

aspect: aspect 是 point cut 与 advice 的组合, 因此在这里我们就可以类比: "根据老王的线索, 凡是发现有身高七尺五寸的男性, 都要抓过来审问" 这一整个动作可以被认为是一个 aspect.

或则我们也可以从语法的角度来简单类比一下. 我们在学英语时, 经常会接触什么 定语, 被动句 之类的概念, 那么可以做一个不严谨的类比, 即 joinpoint 可以认为是一个 宾语, 而 pointcut 则可以类比为修饰 joinpoint 的定语, 那么整个 aspect 就可以描述为: 满足 pointcut 规则的 joinpoint 会被添加相应的 advice 操作.



## AOP 两种代理方式
Spring 提供了两种方式来生成代理对象:JDKProxy和Cglib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用 JDK 动态代理技术，否则使用 Cglib 来生成代理。

### JDK 动态接口代理
1. JDK 动态代理主要涉及到 java.lang.reflect包中的两个类：Proxy和InvocationHandler。InvocationHandler是一个接口，通过实现该接口定义横切逻辑，并通过反射机制调用目标类的代码，动态将横切逻辑和业务逻辑编制在一起。Proxy利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象。

### CGLib 动态代理
2. CGLib 全称为 Code Generation Library，是一个强大的高性能，高质量的代码生成类库，可以在运行期扩展 Java 类与实现 Java 接口，CGLib 封装了 asm，可以再运行期动态生成新的 class。和JDK动态代理相比较：JDK创建代理有一个限制，就是只能为接口创建代理实例，而对于没有通过接口定义业务方法的类，则可以通过 CGLib 创建动态代理。


## 实现原理



<details>
    <summary>CGLib</summary>
    
```
    @Aspect
    public class TransactionDemo {
        @Pointcut(value="execution(* com.yangxin.core.service.*.*.*(..))")
        public void point(){
        }
        @Before(value="point()")
        public void before(){
            System.out.println("transaction begin");
        }
        @AfterReturning(value = "point()")
        public void after(){
            System.out.println("transaction commit");
        }
        @Around("point()")
        public void around(ProceedingJoinPoint joinPoint) throws Throwable{
            System.out.println("transaction begin");
            joinPoint.proceed();
            System.out.println("transaction commit");
        }
    }

```
</details>

# Spring MVC 原理
Spring 的模型-视图-控制器（MVC）框架是围绕一个DispatcherServlet来设计的，这个Servlet会把请求分发给各个处理器，并支持可配置的处理器映射、视图渲染、本地化、时区与主题渲染等，甚至还能支持文件上传。

## Http 请求到 DispatcherServlet
(1) 客户端请求提交到 DispatcherServlet。

## HandlerMapping 寻找处理器
(2) 由 DispatcherServlet 控制器查询一个或多个 HandlerMapping，找到处理请求的Controller。

## 调用处理器 Controller
(3) DispatcherServlet 将请求提交到 Controller。

## Controller 调用业务逻辑处理后，返回 ModelAndView
(4)(5)调用业务处理和返回结果：Controller 调用业务逻辑处理后，返回 ModelAndView。

## DispatcherServlet 查询 ModelAndView
(6)(7)处理视图映射并返回模型： DispatcherServlet 查询一个或多个 ViewResoler 视图解析器，找到 ModelAndView 指定的视图。

## ModelAndView 反馈浏览器 HTTP
(8) Http 响应：视图负责将结果显示到客户端。

## MVC常用注解

# Spring Boot 原理
Spring Boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot 致力于在蓬勃发展的快速应用开发领域(rapid application
development)成为领导者。其特点如下：

1. 创建独立的 Spring 应用程序
2. 嵌入的 Tomcat，无需部署 WAR 文件
3. 简化 Maven 配置
4. 自动配置 Spring
5. 提供生产就绪型功能，如指标，健康检查和外部配置
6. 绝对没有代码生成和对 XML 没有要求配置

# JPA 原理（Java Persistence API）Java持久化API

是一套Sun公司Java官方制定的ORM 方案,是规范，是标准 ，sun公司自己并没有实现


关注点：ORM ，标准 概念 （关键字）

ORM是什么？

ORM（Object Relational Mapping）对象关系映射。

问：ORM有什么用？

在操作数据库之前，先把数据表与实体类关联起来。然后通过实体类的对象操作（增删改查）数据库表，这个就是ORM的行为！

所以：ORM是一个实现使用对象操作数据库的设计思想

通过这句话，我们知道JPA的作用就是通过对象操作数据库的，不用编写sql语句。

JPA的实现者

既然我们说JPA是一套标准，意味着，它只是一套实现ORM理论的接口。没有实现的代码。

那么我们必须要有具体的实现者才可以完成ORM操作功能的实现

市场上的主流的JPA框架（实现者）有：Hibernate （JBoos）、EclipseTop（Eclipse社区）、OpenJPA （Apache基金会）。


## 事务
事务是计算机应用中不可或缺的组件模型，它保证了用户操作的原子性(Atomicity)、一致性(Consistency)、隔离性 (Isolation) 和持久性 (Durabilily)。

## 本地事务
紧密依赖于底层资源管理器（例如数据库连接)，事务处理局限在当前事务资源内。此种事务处理方式不存在对应用服务器的依赖，因而部署灵活却无法支持多数据源的分布式事务。在数据库连接中使用本地事务示例如下：



<details>
    <summary>本地事务</summary>
    
```
    public void transferAccount() {
        Connection conn = null;
        Statement stmt = null;
        try{
            conn = getDataSource().getConnection();
            // 将自动提交设置为 false，若设置为 true 则数据库将会把每一次数据更新认定为一个事务并自动提交
            conn.setAutoCommit(false);
            stmt = conn.createStatement();
            // 将 A 账户中的金额减少 500
            stmt.execute("update t_account set amount = amount - 500 where account_id = 'A'");
            // 将 B 账户中的金额增加 500
            stmt.execute("update t_account set amount = amount + 500 where account_id = 'B'");
            // 提交事务
            conn.commit();
            // 事务提交：转账的两步操作同时成功
        } catch(SQLException sqle){
            // 发生异常，回滚在本事务中的操做
            conn.rollback();
            // 事务回滚：转账的两步操作完全撤销
            stmt.close();
            conn.close();
        }
    }

```
</details>

## 分布式事务
Java 事务编程接口（JTA：Java Transaction API）和 Java 事务服务 (JTS；Java Transaction Service)为J2EE平台提供了分布式事务服务。分布式事务（Distributed Transaction）包括事务管理器（Transaction Manager）和一个或多个支持 XA 协议的资源管理器 ( Resource Manager )。我们可以将资源管理器看做任意类型的持久化数据存储；事务管理器承担着所有事务参与单元的协调与控制。

<details>
    <summary>分布式事务</summary>
    
```
    
    public void transferAccount() {
        UserTransaction userTx = null;
        Connection connA = null; Statement stmtA = null;
        Connection connB = null; Statement stmtB = null;
        try{
            // 获得 Transaction 管理对象
            userTx = (UserTransaction)getContext().lookup("java:comp/UserTransaction");
            connA = getDataSourceA().getConnection();// 从数据库 A 中取得数据库连接
            connB = getDataSourceB().getConnection();// 从数据库 B 中取得数据库连接
            userTx.begin(); // 启动事务
            stmtA = connA.createStatement();// 将 A 账户中的金额减少 500
            stmtA.execute("update t_account set amount = amount - 500 where account_id = 'A'");
            // 将 B 账户中的金额增加 500
            stmtB = connB.createStatement();
            stmtB.execute("update t_account set amount = amount + 500 where account_id = 'B'");
            userTx.commit();// 提交事务
            // 事务提交：转账的两步操作同时成功（数据库 A 和数据库 B 中的数据被同时更新）
        } catch(SQLException sqle){
            // 发生异常，回滚在本事务中的操纵
            userTx.rollback();// 事务回滚：数据库 A 和数据库 B 中的数据更新被同时撤销
        } catch(Exception ne){ }
    }

```
</details>


## 两阶段提交
两阶段提交主要保证了分布式事务的原子性：即所有结点要么全做要么全不做，所谓的两个阶段是指：第一阶段：准备阶段；第二阶段：提交阶段

###  准备阶段
事务协调者(事务管理器)给每个参与者(资源管理器)发送 Prepare 消息，每个参与者要么直接返回失败(如权限验证失败)，要么在本地执行事务，写本地的 redo 和 undo 日志，但不提交，到达一种“万事俱备，只欠东风”的状态。

### 提交阶段：
如果协调者收到了参与者的失败消息或者超时，直接给每个参与者发送回滚(Rollback)消息；否则，发送提交(Commit)消息；参与者根据协调者的指令执行提交或者回滚操作，释放所有事务处理过程中使用的锁资源。(注意:必须在最后阶段释放锁资源)

将提交分成两阶段进行的目的很明确，就是尽可能晚地提交事务，让事务在提交前尽可能地完成所有能完成的工作。

# Mybatis 缓存
Mybatis 中有一级缓存和二级缓存，默认情况下一级缓存是开启的，而且是不能关闭的。一级缓存是指 SqlSession 级别的缓存，当在同一个 SqlSession 中进行相同的 SQL 语句查询时，第二次以后的查询不会从数据库查询，而是直接从缓存中获取，一级缓存最多缓存 1024 条 SQL。二级缓存是指可以跨 SqlSession 的缓存。是 mapper 级别的缓存，对于 mapper 级别的缓存不同的sqlsession 是可以共享的


## Mybatis 的一级缓存原理（sqlsession 级别）
第一次发出一个查询 sql，sql 查询结果写入 sqlsession 的一级缓存中，缓存使用的数据结构是一个 map。

key：MapperID+offset+limit+Sql+所有的入参

value：用户信息

同一个 sqlsession 再次发出相同的 sql，就从缓存中取出数据。如果两次中间出现 commit 操作（修改、添加、删除），本 sqlsession 中的一级缓存区域全部清空，下次再去缓存中查询不到所以要从数据库查询，从数据库查询到再写入缓存。

## 二级缓存原理（mapper 基本）
二级缓存的范围是 mapper 级别（mapper 同一个命名空间），mapper 以命名空间为单位创建缓存数据结构，结构是 map。mybatis 的二级缓存是通过 CacheExecutor 实现的。CacheExecutor其实是 Executor 的代理对象。所有的查询操作，在CacheExecutor中都会先匹配缓存中是否存在，不存在则查询数据库。

key：MapperID+offset+limit+Sql+所有的入参

具体使用需要配置：

1. Mybatis 全局配置中启用二级缓存配置
2. 在对应的 Mapper.xml 中配置 cache 节点
3. 在对应的 select 查询节点中添加 useCache=true

# Tomcat 架构

http://www.importnew.com/21112.html
