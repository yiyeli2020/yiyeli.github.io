title: 数据库学习笔记VII
date: 2020-1-3 11:12:12
categories: 2020年1月
tags: [Database，MySQL]

---
MySQL建表,LIMIT,OFFSET的用法  

<!-- more -->
#  MySQL建表

下面是通用的SQL语法用来创建MySQL表：

    CREATE TABLE table_name (column_name column_type);

现在，我们将在 test 数据库中创建以下表。

    create table tutorials_tbl(
       tutorial_id INT NOT NULL AUTO_INCREMENT,
       tutorial_title VARCHAR(100) NOT NULL,
       tutorial_author VARCHAR(40) NOT NULL,
       submission_date DATE,
       PRIMARY KEY ( tutorial_id )
    );

在这里，一些数据项需要解释：
字段使用NOT NULL属性，是因为我们不希望这个字段的值为NULL。 因此，如果用户将尝试创建具有NULL值的记录，那么MySQL会产生错误。

字段的AUTO_INCREMENT属性告诉MySQL自动增加id字段下一个可用编号。

# LIMIT,OFFSET

    SELECT column_name FROM table_name LIMIT 4 OFFSET 3;
LIMIT 4 OFFSET 3指示MySQL等DBMS返回从第3行（从0行计数）起的4行数据。第一个数字是检索的行数，第二个数字是指从哪儿开始。

MySQL和MariaDB支持简化版的LIMIT 4 OFFSET 3语句，即LIMIT 3,4。使用这个语法，之前的值对应OFFSET之后的值对应LIMIT 。

 limit10000,20的意思扫描满足条件的10020行，扔掉前面的10000行，返回最后的20行.

参考练习题：https://leetcode.com/problems/nth-highest-salary

# mysql变量的种类
用户变量：以"@"开始，形式为"@变量名"。用户变量跟mysql客户端是绑定的，设置的变量，只对当前用户使用的客户端生效
全局变量：定义时，以如下两种形式出现，set GLOBAL 变量名  或者  set @@global.变量名，对所有客户端生效。只有具有super权限才可以设置全局变量
会话变量：只对连接的客户端有效。
局部变量：作用范围在begin到end语句块之间。在该语句块里设置的变量。declare语句专门用于定义局部变量。set语句是设置不同类型的变量，包括会话变量和全局变量
通俗理解术语之间的区别：

用户定义的变量就叫用户变量。这样理解的话，会话变量和全局变量都可以是用户定义的变量。只是他们是对当前客户端生效还是对所有客户端生效的区别了。所以，用户变量包括了会话变量和全局变量

局部变量与用户变量的区分在于:
1.用户变量是以"@"开头的。局部变量没有这个符号。
2.定义变量不同。用户变量使用set语句，局部变量使用declare语句定义
3.作用范围。局部变量只在begin-end语句块之间有效。在begin-end语句块运行完之后，局部变量就消失了。

所以，最后它们之间的层次关系是：变量包括局部变量和用户变量。用户变量包括会话变量和全局变量。



# mysql declare定义变量
mysql declare用于定义变量，在存储过程和函数中通过declare定义变量在BEGIN...END中，且在语句之前。并且可以通过重复定义多个变量

declare变量的作用范围同编程里面类似，在这里一般是在对应的begin和end之间。在end之后这个变量就没有作用了，不能使用了。这个同编程一样。

注意：declare定义的变量名不能带‘@’符号，mysql在这点做的确实不够直观，往往变量名会被错成参数或者字段名。

mysql存储过程中使用declare定义变量，实例如下：

 DROP PROCEDURE IF EXISTS insert_ten_rows $$
 CREATE PROCEDURE insert_ten_rows ()
     BEGIN
         DECLARE crs INT DEFAULT 0;

         WHILE crs < 10 DO
             INSERT INTO `continent`(`name`) VALUES ('cont'+crs);
             SET crs = crs + 1;
         END WHILE;
     END $$

 DELIMITER ;

# mysql SET定义变量
mysql set也可以用来定于变量，定义变量的形式是以"@"开始，如："@变量名"。

mysql SET定义变量实例：

   mysql> SET @t1=0, @t2=1, @t3=2;

   mysql> select @t1;
   +------+
   | @t1  |
   +------+
   | 0    |
   +------+

   mysql> select @t2;
   +------+
   | @t2  |
   +------+
   | 1    |
   +------+

   mysql> select @t3;
   +------+
   | @t3  |
   +------+
   | 2    |
   +------+

   mysql>

复杂一点的实例：

   mysql> SET @t1=0, @t2=1, @t3=2;

   mysql> SELECT @t1:=(@t2:=1)+@t3:=4,@t1,@t2,@t3;
   +----------------------+------+------+------+
   | @t1:=(@t2:=1)+@t3:=4 | @t1  | @t2  | @t3  |
   +----------------------+------+------+------+
   |                    5 | 5    | 1    | 4    |
   +----------------------+------+------+------+



# mysql declare和set定义变量的区别
mysql declare和set定义变量，除了一个不加@和一个加@这个区别之外，还有以下区别：

 declare用来定义局部变量

 @用来定义会话变量

 declare变量的作用范围同编程里面类似，在这里一般是在对应的begin和end之间。在end之后这个变量就没有作用了，不能使用了。这个同编程一样。

 另外有种变量叫做会话变量(session variable)，也叫做用户定义的变量(user defined variable)。这种变量要在变量名称前面加上“@”符号，叫做会话变量，代表整个会话过程他都是有作用的，这个有点类似于全局变量一样。这种变量用途比较广，因为只要在一个会话内(就是某个应用的一个连接过程中)，这个变量可以在被调用的存储过程或者代码之间共享数据。




#参考资料
【1】 https://tech.meituan.com/2014/06/30/mysql-index.html
【2】 https://blog.csdn.net/qq_37923253/article/details/79688313
【3】 http://www.manongjc.com/article/1441.html
