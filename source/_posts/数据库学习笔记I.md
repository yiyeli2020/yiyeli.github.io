---
title: 数据库学习笔记I
date: 2019-7-31 11:12:12
categories: 2019年7月
tags: [Database，SQL]

---

SQL基本查询和分页查询

<!-- more -->
# SQL查询
AS 一般是重命名列名或者表名。

***例如有表table，列 column_1,column_2***

你可以写成

    select  column_1  as  列1,
            column_2  as  列2   
            from  table  as  表

上面的语句就可以解释为，选择 column_1  作为  列1,column_2 作为   列2  从 table  当成 表

***SELECT * FROM Employee AS emp***

这句意思是查找所有Employee 表里面的数据，并把Employee表格命名为 emp。
当你命名一个表之后，你可以在下面用 emp 代替 Employee.
例如 SELECT * FROM emp.

***把查询对像起个别名的作用***

    select ID as 用户ID，Name as 用户名 from Table_user

查出结果就以中文显示

    select mytableA.* ,mytableB.*
    from
    tb_user as mytableA
    join
    Tb_UserGroup as mytableB
    on mytableA.ID=mytableB.ID。

这样就可以把查询结果起别名


# SQL分页查询
使用SELECT查询时，如果结果集数据量很大，比如几万行数据，放在一个页面显示的话数据量太大，不如分页显示，每次显示100条。

要实现分页功能，实际上就是从结果集中显示第1 ~ 100条记录作为第1页，显示第101~200条记录作为第2页，以此类推。

因此，分页实际上就是从结果集中“截取”出第M~N条记录。这个查询可以通过LIMIT <M> OFFSET <N>子句实现。

例如基本查询语句：

    SELECT id,name,gender,score
    FROM students
    ORDER BY score DESC;

现在将结果集分页，每页3条记录。要获取第一页的记录，可以使用 LIMIT 3 OFFSET 0:

    SELECT id,name,gender,score
    FROM students
    ORDER BY score DESC
    LIMIT 3 OFFSET 0;

上述查询LIMIT 3 OFFSET 0表示：
对结果集从0号记录开始，最多取3条（注意SQL记录集对索引从0开始）。
若要查询第2页，那么我们只需要跳过前3条记录，即对结果集从3号记录开始查询，把OFFSET设定为3:

    SELECT id,name,gender,score
    FROM students
    ORDER BY score DESC
    LIMIT 3 OFFSET 3;

类似的，查询第3页的时候，OFFSET应该设为6，查询第4页的时候，OFFSET应该设为9.如果查询的表第4页只有1条记录，则最终结果按照实际数量1显示，LIMIT 3表示的意思是“最多3条记录”。

可见分页查询的关键在于，首先要确定每页需要显示的结果数量pageSize，然后根据当前页的索引pageIndex（从1开始），确定LIMIT和OFFSET应该设定的值：

LIMIT总是设定为pageSize；
OFFSET计算公式为pageSize*（pageIndex-1）。

## 扩展思考
如果原本记录集只有10条记录，但我们将OFFSET设置为20，结果会怎样呢？
答案是OFFSET超过查询的最大数量并不会报错，而是会得到一个空的结果集

## 注意
OFFSET是可选的，OFFSET缺省值为0.即LIMIT 3 相当于LIMIT 3 OFFSET 0
在MySQL中，LIMIT 3 OFFSET 0也可简写为LIMIT 3，0
使用LIMIT M OFFSET N进行分页查询时，N越大查询效率越低。

# SQL连接查询
连接查询是将两个或两个以上的表按某些条件连接起来，从中选取需要的数据。可以分为内连接查询(通过where实现)和外连接查询（join）。

## 内连接查询
只有不同表中有相同意义的字段时才能进行连接，而且内连接查询只查询出指定字段取值相同的记录。
![https://img-blog.csdn.net/20171209135846780?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGxnMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast](assets/markdown-img-paste-20190801165654826.png)

    一般语法:
    select a.* , b.*
    from table_a as a, table_b as b
    where a.id = b.id;
## 外连接查询
需要通过指定字段来进行连接。当该字段取值相等时，可以查询出该记录；而且当该字段不等时，也可以查询出来。包括左连接，右连接查询。

    一般语法：
    select 属性名列表
    from 表1
    left | right join 表2
    on 表1.属性名 = 表2.属性名;

### 左外连接
left join 是left outer join的简写，它的全称是左外连接，是外连接中的一种。
左(外)连接，左表(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。
![https://img-blog.csdn.net/20171209142610819?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGxnMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast](assets/markdown-img-paste-2019080116575036.png)

### 右外连接
右外连接与左外连接相对称，
right join是right outer join的简写，它的全称是右外连接，是外连接中的一种。
与左(外)连接相反，右(外)连接，左表(a_table)只会显示符合搜索条件的记录，而右表(b_table)的记录将会全部表示出来。左表记录不足的地方均为NULL。
![https://img-blog.csdn.net/20171209144056668?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGxnMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast](assets/markdown-img-paste-20190801175330121.png)
### 外连接查询加条件语句

使用外连接查询时，可以加上各种条件进行筛选。

    select table1.column1, table2.column1
    from table1
    join table2
    on table1.column2 = table2.column3;

    select table1.column1, table2.column1
    from table1,table2
    where table1.column2 = table2.column3;

## 子查询
子查询时将一个查询语句嵌套在另一个查询语句中，内层查询语句的查询结果，可以为外层查询语句提供查询条件。在特定情况下：一个查询语句的条件需要另一个查询语句来获取。
子查询，又叫内部查询，相对于内部查询，包含内部查询的就称为外部查询。

子查询可以包含普通select可以包括的任何子句，比如：distinct、 group by、order by、limit、join和union等；但是对应的外部查询必须是以下语句之一：select、insert、update、delete、set或 者do。

注：一个查询语句只能有一个order by ，在子查询中只能位于外部查询后面.

### 分类

子查询分为如下几类：
1）. 标量子查询：返回单一值的标量，最简单的形式。
2）. 列子查询：返回的结果集是 N 行一列。
3）. 行子查询：返回的结果集是一行 N 列。
4）. 表子查询：返回的结果集是 N 行 N 列。
可以使用的操作符：= > < >= <= <> ANY IN SOME ALL EXISTS

释义：一个子查询会返回一个标量（就一个值）、一个行、一个列或一个表，这些子查询称之为标量、行、列和表子查询。

### 带有any关键字的子查询

any关键字表示满足其中任一条件，使用any关键字时，只要满足内层查询语句返回的结果中的任何一个，就可以通过该条件来执行外层查询语句。

从computer表中查询哪些人分数高于任何一个奖学金的最低分。

    select * from computer_stu
    where score >= ANY
                      (select score From  scholarship);
### 带有all关键字的子查询

表示需要满足所有的条件。只有满足内层查询语句返回的所有结果，才可以执行外层查询语句。

### EXISTS和NOT EXISTS用法

EXISTS关键字表示存在，内层查询语句不返回查询的记录，而是返回一个真假值，如果内层查询语句查询到满足条件的记录，就返回一个true，外层查询语句将进行查询。

    select * from employee
    where exists (select d_name from department where d_id = 1003);
    //如果department存在d_id为1003，则查询employee表。
还可以分为相关子查询，独立子查询。以上子查询与外层查询没有关联，称为独立子查询，如果子查询有用到外层查询的字段，则称相关子查询，相关子查询容易产生性能问题。

EXISTS （sql 返回结果集为真）
NOT EXISTS (sql 不返回结果集为真）

表A

    ID NAME
    1    A1
    2    A2
    3    A3
表B

    ID AID NAME
    1    1 B1
    2    2 B2
    3    2 B3

表A和表B是１对多的关系 A.ID => B.AID

    SELECT ID,NAME
    FROM A
    WHERE EXIST (
      SELECT * FROM B
      WHERE A.ID=B.AID)
执行结果为

    1 A1
    2 A2
## TOP关键字
TOP关键字在SQL语言中用来限制返回结果集中的记录条数
（1）返回确定数目的记录个数

语法格式：

    SELECT TOP n <列名表> FROM <表名> [查询条件]

其中，n为要返回结果集中的记录条数

（2）返回结果集中指定百分比的记录数

语法格式：

    SELECT TOP n PERCENT <列名表> FROM <表名> [查询条件]

其中，n为所返回的记录数所占结果集中记录数目的百分比数

使用例子：
查询某个条件多条数据中创建时间最新的一条

      select top 1 * from tablename
      where id=123456
      order by 时间 desc      

但是需要注意的是在MySQL中没有此语法，MySQL中使用limit来实现相关功能。
同样的例子，MySQL中可以写为：

      select * from tablename
      where id=123456
       order by 时间 desc limit 1
# NOT IN 关键字

示例：
    select customers.name as 'Customers'
    from customers
    where customers.id not in
    (
        select customerid from orders
    );

参考练习：https://leetcode.com/problems/customers-who-never-order/solution/
# MySQL中IN、NOT IN、EXISTS、NOT EXISTS的差别

## IN和EXISTS
in是把外表和内表作hash连接，而exists是对外表作loop循环，每次loop循环再对内表进行查询，一直以来认为exists比in效率高的说法是不准确的。如果查询的两个表大小相当，那么用in和exists差别不大；如果两个表中一个较小一个较大，则子查询表大的用exists，子查询表小的用in。

例如：表A(小表)，表B(大表)

    select * from A where cc in(select cc from B)　　               -->效率低，用到了A表上cc列的索引；
    select * from A where exists(select cc from B where cc=A.cc)　　-->效率高，用到了B表上cc列的索引;
相反的

    select * from B where cc in(select cc from A)　　               -->效率高，用到了B表上cc列的索引;
    select * from B where exists(select cc from A where cc=B.cc)　　-->效率低，用到了A表上cc列的索引;

## not in和not exists

如果查询语句使用了not in，那么对内外表都进行全表扫描，没有用到索引；而not exists的子查询依然能用到表上的索引。所以无论哪个表大，用not exists都比not in要快。

## in与=的区别

    select name from student where name in('zhang','wang','zhao');

与

    select name from student where name='zhang' or name='wang' or name='zhao'
结果是相同的。

## 关于EXISTS：
EXISTS用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回值True或False。

EXISTS 指定一个子查询，检测行的存在。

语法： EXISTS subquery

参数： subquery 是一个受限的 SELECT 语句 (不允许有 COMPUTE 子句和 INTO 关键字)。

结果类型： Boolean 如果子查询包含行，则返回 TRUE ，否则返回 FLASE 。

结论：

    select * from A where exists (select 1 from B where A.id=B.id)
EXISTS(包括 NOT EXISTS )子句的返回值是一个boolean值。 EXISTS内部有一个子查询语句(SELECT ... FROM...),我将其称为EXIST的内查询语句。其内查询语句返回一个结果集, EXISTS子句根据其内查询语句的结果集空或者非空，返回一个布尔值。

一种通俗的可以理解为：将外查询表的每一行，代入内查询作为检验，如果内查询返回的结果取非空值，则EXISTS子句返回TRUE，这一行行可作为外查询的结果行，否则不能作为结果。

分析器会先看语句的第一个词，当它发现第一个词是SELECT关键字的时候，它会跳到FROM关键字，然后通过FROM关键字找到表名并把表装入内存。接着是找WHERE关键字，如果找不到则返回到SELECT找字段解析，如果找到WHERE，则分析其中的条件，完成后再回到SELECT分析字段。最后形成一张我们要的虚表。

WHERE关键字后面的是条件表达式。条件表达式计算完成后，会有一个返回值，即非0或0，非0即为真(true)，0即为假(false)。同理WHERE后面的条件也有一个返回值，真或假，来确定接下来执不执行SELECT。

分析器先找到关键字SELECT，然后跳到FROM关键字将STUDENT表导入内存，并通过指针找到第一条记录，接着找到WHERE关键字计算它的条件表达式，如果为真那么把这条记录装到一个虚表当中，指针再指向下一条记录。如果为假那么指针直接指向下一条记录，而不进行其它操作。一直检索完整个表，并把检索出来的虚拟表返回给用户。EXISTS是条件表达式的一部分，它也有一个返回值(true或false)。



# 参考资料：

【1】https://www.liaoxuefeng.com/wiki/1177760294764384/1217864791925600
【2】https://blog.csdn.net/plg17/article/details/78758593
【3】https://cloud.tencent.com/developer/article/1333120
【4】https://www.jianshu.com/p/24e4e8437686
