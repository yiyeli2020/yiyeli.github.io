title: 数据库学习笔记-MySQL安装

date: 2020-6-10 18:12:12

categories: 2020年6月

tags: [Database,MySQL]

---

Mac上安装MySQL遇到的问题及解决方法

<!-- more -->

# 安装
安装步骤参见[^1]

# 登录到MySQL数据库服务器

安装完毕后，终端输入`mysql -u root -p`启动MySQL

## 遇到问题

登录遇到错误
    
    ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)

## 解决方法
参见[^2][^3]
但是输入命令重启MySQL服务器：

    systemctl restart mysqld

发现报错：
    
    bash: systemctl: command not found

因为systemctl命令是用于Linux的，在MacOS上替代命令是launchctl，Apache的HTTP控制接口是apachectl。

先停止MySQL数据库：

    launchctl stop mysqlId

然后再重置密码

因为我是用[^1]中的下载dmg包安装MySQL的，所以可以直接去系统偏好设置中找到MySQL服务，然后重置密码。

这样就可以登录了。

# 列出所有的数据库

    show databases;

# 建立数据库

首先建立数据库，再在其中建立数据表即可：
    
    create database （database name）;

# 使用USE语句切换到特定的数据库 

    use （database name）;

别忘记每句后加分号,我们建立数据库mysql_practice：
    
    mysql> create database mysql_practice;
    Query OK, 1 row affected (0.00 sec)
    
    mysql> use mysql_practice
    Database changed

# 列出数据库中的所有表

    show tables;

SHOW TABLES命令可显示表是基表还是视图[^4]。 要在结果中包含表类型，请使用SHOW TABLES语句，如下所示 
    
    SHOW FULL TABLES;

对于具有很多表的数据库，一次显示所有表可能不免直观。

幸运的是，SHOW TABLES命令提供了一个选项，允许使用LIKE运算符或WHERE子句中的表达式对返回的表进行过滤，如下所示：

    SHOW TABLES LIKE pattern;
    
    SHOW TABLES WHERE expression;

例如，要显示数据库中以字母p开头的所有表，请使用以下语句:

    SHOW TABLES LIKE 'p%';

或者显示以’es‘字符串结尾的表，可使用以下语句：

    SHOW TABLES LIKE '%es';

更多使用方法详见：[^4]

# 退出MySQL

    exit


[^1]:https://juejin.im/post/5cc2a52ce51d456e7079f27f

[^2]:https://www.cnblogs.com/zhongyehai/p/10695334.html

[^3]:https://blog.csdn.net/qq_32786873/article/details/79225039

[^4]:https://www.yiibai.com/mysql/show-tables.html