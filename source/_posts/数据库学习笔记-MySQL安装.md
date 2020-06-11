title: 数据库学习笔记-MySQL安装

date: 2020-6-10 18:12:12

categories: 2020年6月

tags: [Database，MySQL]

---

Mac上安装MySQL遇到的问题及解决方法

<!-- more -->

# 安装
安装步骤参见[^1]

安装完毕后，终端输入`myqsl -u root -p`启动MySQL

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

# 建立数据库

首先建立数据库，再在其中建立数据表即可：
    
    create database （database name）;
    
    use （database name）;

别忘记每句后加分号
    
    mysql> create database mysql_practice;
    Query OK, 1 row affected (0.00 sec)
    
    mysql> use mysql_practice
    Database changed


[^1]:https://juejin.im/post/5cc2a52ce51d456e7079f27f

[^2]:https://www.cnblogs.com/zhongyehai/p/10695334.html

[^3]:https://blog.csdn.net/qq_32786873/article/details/79225039