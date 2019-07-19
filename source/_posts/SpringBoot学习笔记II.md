---
title: SpringBoot学习笔记II
date: 2019-7-19 11:23:12
categories: 2019年7月
tags: [SpringBoot]

---

总结整理在学习SpringBoot的过程中遇到的问题和解决方案。

<!-- more -->


# 常见问题

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

# 参考文献
[1]https://stackoverflow.com/questions/40738818/illegalargumentexception-at-least-one-jpa-metamodel-must-be-present?newreg=1d1be5c9c5a04ec2878d9fc8237bbda5
[2]]https://blog.csdn.net/zsg88/article/details/80780281
