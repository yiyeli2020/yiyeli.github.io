---
title: SpringBoot学习笔记II
date: 2019-7-19 11:23:12
categories: 2019年7月
tags: [SpringBoot]

---

总结整理在学习SpringBoot的过程中遇到的问题和解决方案。

<!-- more -->
# 三省吾身：
Spring 是什么？

Spring和SpringBoot的区别？

@Resource和@Autowired的区别？

# JPA任务：
在Merchant表中完成一个JPA项目的增删改查,并在test/java/com.example.std.java.demo/DemoApplicationTests中进行测试

# Mybatis任务：
在Merchant表中完成一个Mybatis项目的增删改查,并在test/java/com.example.std.java.demo/DemoApplicationTests中进行测试

# 常见问题


## 建表注意事项：

创建数据访问接口userRepository建在dao层中。

社区版的Intellij IDEA可以直接在文件夹上右键New->Jpa Entities即可自动创建实体。

对数据库对增删改查可以直接写在test文件夹下的DemoApplicationTest中，每次执行时只需要Run相应的test函数即可，不需全部Run一遍。

## 常见问题application.properties文件解析错误：

application.yml和application.properties的格式是不同的，注意更改格式。
## 导入目录时格式混乱
打开本地项目时直接open pom.xml会自动解析项目

如果从git上拉分支则注意：
Intellij IDEA从git上拉取分支时要选择import project from external model，
在import Project一栏时Root directory栏目中不要默认目录，否则会找不到pom.xml，要选择clone下来的目录中的demo文件
## Maven配置问题

File->Preference->Maven中的Maven home directory; User settings file; Local repository三个根目录中都要选择公司私有的maven目录
## Debug问题

IDEA中错误信息以堆栈形式输出，所以最下面的错误是第一个错误，优先看


## hibernate.dialect配置问题
启动时报错：
Access to DialectResolutionInfo cannot be null when 'hibernate.dialect' not set

hibernate.dialect是为了更好的适配各种数据库，针对每种数据库都指定方言dialect，将各类数据库Oracle，Mysql等不同类型等语法
转换成hibernate能理解等统一的格式。
### 解决方法1:
这个问题花了很长时间，google到的方法都没有作用，最后确定原因是demo/config/JpaConfig文件中的factory.setJpaProperties(jpaProperties.getProperties());一行中Jpaproperties读到的属性为空，

demo/config/JpaConfig文件如下

    package com.example.std.java.demo.config;

    import javax.annotation.Resource;
    import javax.persistence.EntityManagerFactory;
    import javax.sql.DataSource;

    import com.example.std.java.demo.config.jpa.HibernateConfig;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
    import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Primary;
    import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
    import org.springframework.orm.jpa.DefaultJpaDialect;
    import org.springframework.orm.jpa.JpaTransactionManager;
    import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
    import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
    import org.springframework.transaction.PlatformTransactionManager;
    import org.springframework.transaction.annotation.EnableTransactionManagement;

    import java.util.Map;
    import java.util.Properties;

    @Configuration
    @EnableTransactionManagement
    @EnableJpaRepositories(
        entityManagerFactoryRef = "entityManagerFactory",
        transactionManagerRef = "platformTransactionManager",
        basePackages = {"com.example.std.java.demo.dao.jpa"}) //设置Repository所在位置
    public class JpaConfig {

        @Resource
        private HibernateConfig jpaProperties;

        @Resource(name = "testDataSource")
        private DataSource dataSource;



        @Primary
        @Bean(name = "entityManagerFactory")
        public EntityManagerFactory entityManagerFactory() {
            HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
            LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
            factory.setJpaVendorAdapter(vendorAdapter);
            factory.setPackagesToScan("com.example.std.java.demo.entity.jpa");
            factory.setDataSource(dataSource);//数据源
            factory.setJpaProperties(jpaProperties.getProperties());
            factory.afterPropertiesSet();//在完成了其它所有相关的配置加载以及属性设置后,才初始化
            return factory.getObject();
        }

        /**
         * 配置事物管理器
         * @return
         */
        @Bean(name = "platformTransactionManager")
        @Primary
        public PlatformTransactionManager platformTransactionManager() {
            JpaTransactionManager jpaTransactionManager = new JpaTransactionManager();
            jpaTransactionManager.setEntityManagerFactory(this.entityManagerFactory());
            return jpaTransactionManager;
        }


    }

为了能传进属性，在demo/config/jpa/HibernateConfig文件中增加属性properties


    package com.example.std.java.demo.config.jpa;


    import lombok.Data;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.context.annotation.Configuration;

    import java.util.Properties;

    @Configuration
    @ConfigurationProperties(prefix = "user.jpa")
    @Data
    public class HibernateConfig {

        Properties properties;


    }

其中注解中的的@ConfigurationProperties(prefix = "user.jpa")指的是resources/local/application.yml中的数据源配置


    spring:
      datasource:
        test:
          jdbcUrl: jdbc:mysql://10.143.248.78:3306/java_demo?autoReconnect=true&characterEncoding=UTF8&&parseTime=True
          type: org.apache.tomcat.jdbc.pool.DataSource
          username: root
          password: mojiti
          testOnBorrow: true
          validationQuery: "select version()"
          driver-class-name: com.mysql.jdbc.Driver
        shangtongdai:
          jdbcUrl: jdbc:mysql://10.143.248.78:3306/shangtongdai?autoReconnect=true&characterEncoding=UTF8&&parseTime=True
          type: org.apache.tomcat.jdbc.pool.DataSource
          username: root
          password: mojiti
          testOnBorrow: true
          validationQuery: "select version()"
          driver-class-name: com.mysql.jdbc.Driver
    user:
      jpa:
        properties:
          hibernate:
            hbm2ddlauto: update
            dialect: org.hibernate.dialect.MySQL5InnoDBDialect
### 解决方法2:
也可以直接将properties固定的写在JpaConfig中



## 新建SpringBoot项目启动时往往会报错
###  常见错误类型：

      ***************************
      APPLICATION FAILED TO START
      ***************************

      Description:

      Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

      Reason: Failed to determine a suitable driver class



### 解决办法：

配置Mybatis时出现的错误，原因是因为添加了数据库组件，所以autoconfig会去读取数据源但配置，而新建项目还没有配置数据源，所以会导致异常出现。

方法1:需要先在target/pom.xml中去掉对数据库的依赖，即去掉下面这段。

    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.0</version>
    </dependency>

方法2:也可以在启动类的@EnableAutoConfiguration或@SpringBootApplication中添加exclude = {DataSourceAutoConfiguration.class}，排除此类的autoconfig。启动以后就可以正常运行。修改如下：

    @SpringBootApplication(exclude= DataSourceAutoConfiguration.class)
    public class DemoApplication {
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }

### 常见错误类型：


    Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.

### 解决办法：

这是配置JPA时出现的错误，错误原因同上，同理可知解决方法。

## 建表问题：
在多数据源情况下，会出现所链接的数据库中没有需要查询的表，而该表在另一个数据库中，此时在demo/config/JpaConfig文件中找到

    @Qualifier("testDataSource")
    private DataSource dataSource;

改为
    @Resource(name = "testDataSource")
    private DataSource dataSource;
或者@Autowired注解来选择数据源，其中dataSource等来自于demo/config/DbAutoConfiguration

    package com.example.std.java.demo.config;

    import javax.sql.DataSource;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.boot.jdbc.DataSourceBuilder;
    import org.springframework.context.annotation.AdviceMode;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.transaction.annotation.EnableTransactionManagement;

    /**
     * @Description: db数据库自动化配置
     * @Author: 
     * @Date: 2019-07-11 14:50
     */
    @Configuration
    @EnableTransactionManagement(mode = AdviceMode.ASPECTJ)
    public class DbAutoConfiguration {

        @Bean("testDataSource")
        @ConfigurationProperties(prefix = "spring.datasource.test")
        public DataSource dataSource() {
            return DataSourceBuilder.create().build();
        }

        @Bean("dataSource")
        @ConfigurationProperties(prefix = "spring.datasource.shangtongdai")
        public DataSource dataSource2() {
            return DataSourceBuilder.create().build();
        }

    }

# 参考文献
[1]https://stackoverflow.com/questions/40738818/illegalargumentexception-at-least-one-jpa-metamodel-must-be-present?newreg=1d1be5c9c5a04ec2878d9fc8237bbda5
[2]]https://blog.csdn.net/zsg88/article/details/80780281
