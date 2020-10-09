
title: Java 练手项目

date: 2020-8-17 18:22:12

categories: 2020年8月

tags: [Java]

---

Java 练手项目过程中的问题总结

<!-- more -->

# Github加速clone
首先从github上找了几个Java的练手项目[^1]，clone时就遇到问题，太慢了，所以参照方法[^2]

在git clone命令中将gitclone.com嵌入到克隆地址中即可

方法一（替换URL）

git clone https://gitclone.com/github.com/tendermint/tendermint.git

方法二（设置git参数）

git config --global url."https://gitclone.com/".insteadOf https://

git clone https://github.com/tendermint/tendermint.git

方法三（使用cgit客户端）

cgit clone https://github.com/tendermint/tendermint.git


# 在本地建立数据库

在本地建立数据库时要注意数据库版本是5.0还是8.0，因为很多项目的配置都是按照5.0配置的，所以我这个项目也选择5.0

我一开始运行项目，但是遇到以下问题
<details>
    <summary>报错信息</summary>
    
```
2020-08-17 18:08:40.855 ERROR 67432 --- [main] com.alibaba.druid.pool.DruidDataSource   : init datasource error, url: jdbc:mysql://127.0.0.1:3306/utf8mb4?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&useSSL=true

com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```
</details>

我把url中的选项useSSL改成false就不报错了，useSSL=false


[^1]:https://github.com/caozongpeng/SpringBootBlog

[^2]:https://gitclone.com/