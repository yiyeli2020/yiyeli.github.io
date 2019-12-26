title: 数据库学习笔记VI
date: 2019-11-5 11:54:12
categories: 2019年11月
tags: [Mysql]

---

MySQL优化及常用函数

<!-- more -->
# SQL CONCAT 函数
CONCAT 函数用于将两个字符串连接为一个字符串，试一下下面这个例子：

    SQL> SELECT CONCAT('FIRST ', 'SECOND');
    +----------------------------+
    | CONCAT('FIRST ', 'SECOND') |
    +----------------------------+
    | FIRST SECOND               |
    +----------------------------+
    1 row in set (0.00 sec)

# case when then else end用法

    -简单case函数
    case sex
      when '1' then '男'
      when '2' then '女’
      else '其他' end
    --case搜索函数
    case when sex = '1' then '男'
         when sex = '2' then '女'
         else '其他' end  
需要注重的问题，case函数只返回第一个符合条件的值，剩下的case部分将会被自动忽略。

# GROUP BY 语句
GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。

实例
我们拥有下面这个 "Orders" 表：

    O_Id	OrderDate	OrderPrice	Customer
    1	2008/12/29	1000	Bush
    2	2008/11/23	1600	Carter
    3	2008/10/05	700	Bush
    4	2008/09/28	300	Bush
    5	2008/08/06	2000	Adams
    6	2008/07/21	100	Carter
现在，我们希望查找每个客户的总金额（总订单）。

我们想要使用 GROUP BY 语句对客户进行组合。

我们使用下列 SQL 语句：

    SELECT Customer,SUM(OrderPrice) FROM Orders
    GROUP BY Customer
结果集类似这样：

    Customer	SUM(OrderPrice)
    Bush	2000
    Carter	1700
    Adams	2000
#参考资料
【1】 https://tech.meituan.com/2014/06/30/mysql-index.html
