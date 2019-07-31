---
title: SpringBoot学习笔记VII
date: 2019-7-31 11:12:12
categories: 2019年7月
tags: [SpringBoot]

---

Mybatis的分页查询，日志log的写法详解

<!-- more -->

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


# Mybatis分页查询



















# 日志Log写法详解































# 参考资料：

【1】https://www.liaoxuefeng.com/wiki/1177760294764384/1217864791925600
