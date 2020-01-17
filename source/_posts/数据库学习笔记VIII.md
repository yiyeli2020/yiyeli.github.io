title: 数据库学习笔记VIII

date: 2020-1-17 17:51:12

categories: 2020年1月

tags: [Mysql]

---

MS SQL Server和MySQL区别  

<!-- more -->

#  MS SQL Server和MySQL区别
MySQL支持enum,和set类型，SQL Server不支持

MySQL不支持nchar,nvarchar,ntext类型

MySQL的递增语句是AUTO_INCREMENT，而MS SQL是identity(1,1)

MS SQL不支持replace into 语句，但是在最新的sql20008里面，也支持merge语法

MySQL支持insert into table1 set t1 = “”, t2 = “”,但是MSSQL不支持这样写

MySQL支持insert into tabl1 values (1,1), (1,1), (1,1),(1,1),(1,1), (1,1), (1,1)

MS SQL不支持limit语句，是非常遗憾的，只能用top 取代limt0,N，row_number()over()函数取代limit N,M

MySQL在创建表时要为每个表指定一个存储引擎类型，而MS SQL只支持一种存储引擎

PHP连接MySQL和MS SQL的方式都差不多，只需要将函数的MySQL替换成MS SQL即可。

MySQL的管理工具有几个比较好的，MySQL_front,和官方那个套件，不过都没有SSMS的使用方便，这是MySQL很大的一个缺点。

MySQL需要为表指定存储类型

MS SQL识别符是,[type]表示他区别于关键字，但是MySQL却是 `，也就是按键1左边的那个符号

MSSQL支持getdate()方法获取当前时间日期，但是MySQL里面可以分日期类型和时间类型，获取当前日期是cur_date()，当前完整时间是now()函数

MySQL不支持默认值为当前时间的datetime类型（MSSQL很容易做到），在MySQL里面是用timestamp类型

MS SQL里面检查是否有这个表再删除，需要这样：
    　　if exists (select * from dbo.systs
    where id = t_id(N"uc_newpm")andTPROPERTY(id,N"IsUserTable")=1)
　　但是在MySQL里面只需要 DROP TABLE IF EXISTS cdb_forums;

MySQL text字段类型不允许有默认值

MySQL的一个表的总共字段长度不超过65XXX。

MS SQL默认到处表创建语句的默认值表示是((0)),而在MySQL里面是不允许带两括号的

同样的负载压力，MySQL要消耗更少的CPU和内存，MS SQL的确是很耗资源。

MySQL支持无符号型的整数，那么比不支持无符号型的MS SQL就能多出一倍的最大数存储

MySQL不支持在MSSQL里面使用非常方便的varchar(max)类型，这个类型在MSSQL里面既可做一般数据存储，也可以做blob数据存储

MySQL创建非聚集索引只需要在创建表的时候指定为key就行，比如：KEYdisplayorder(fid,displayorder) 在MS SQL里面必须要：

  　　create unique nonclustered index
  　　index_uc_protectedmrs_username_appid on dbo.uc_protectedmrs
  　　(username asc,appid asc)
一个很表面的区别就是MySQL的安装特别简单，而且文件大小才110M（非安装版）

MySQL的存储过程只是出现在最新的版本中，稳定性和性能可能不如MS SQL。

MySQL支持date,time,year类型，MS SQL到2008才支持date和time。



#参考资料
【1】 https://blog.csdn.net/xc_zhou/article/details/89002519
