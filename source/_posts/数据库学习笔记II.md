title: 数据库学习笔记II
date: 2019-8-9 15:12:12
categories: 2019年8月
tags: [Java,Mysql]

---

重新温习Mysqlkey 、primary key 、unique key 与index的区别


<!-- more -->

# key与primary key区别

    CREATE TABLE wh_logrecord (
      logrecord_id int(11) NOT NULL auto_increment,
      user_name varchar(100) default NULL,
      operation_time datetime default NULL,
      logrecord_operation varchar(100) default NULL,
      PRIMARY KEY (logrecord_id),
      KEY wh_logrecord_user_name (user_name)
      )

## 解析：

  KEY wh_logrecord_user_name (user_name)

  本表的user_name字段与wh_logrecord_user_name表user_name字段建立外键

  括号外是建立外键的对应表，括号内是对应字段

  类似还有 KEY user(userid)

  当然，key未必都是外键

## 总结：

  Key是索引约束，对表中字段进行约束索引的，都是通过primary foreign unique等创建的。
  常见有foreign key，外键关联用的。

  KEY forum (status,type,displayorder)  # 是多列索引（键）

  KEY tid (tid)                         # 是单列索引（键）。

  如建表时： KEY forum (status,type,displayorder)

  select * from table group by status,type,displayorder 是否就自动用上了此索引，

  而当 select * from table group by status 此索引有用吗？

  key的用途：主要是用来加快查询速度的。

# KEY与INDEX区别


KEY通常是INDEX同义词。

如果关键字属性PRIMARY KEY在列定义中已给定，则PRIMARY KEY也可以只指定为KEY。这么做的目的是与其它数据库系统兼容。

PRIMARY KEY是一个唯一KEY，此时，所有的关键字列必须定义为NOT NULL。

如果这些列没有被明确地定义为NOT NULL，MySQL应隐含地定义这些列。

一个表只有一个PRIMARY KEY。

## MySQL 中Index 与Key 的区别

Key即键值，是关系模型理论中的一部份，比如有主键（Primary Key)，外键（Foreign Key）等，用于数据完整性检否与唯一性约束等。

而Index则处于实现层面，比如可以对表个的任意列建立索引，那么当建立索引的列处于SQL语句中的Where条件中时，就可以得到快速的数据定位，从而快速检索。

至于Unique Index，则只是属于Index中的一种而已，建立了Unique Index表示此列数据不可重复，猜想MySQL对Unique Index类型的索引可以做进一步特殊优化吧。

于是乎，在设计表的时候，Key只是要处于模型层面的，而当需要进行查询优化，则对相关列建立索引即可。

另外，在MySQL中，对于一个Primary Key的列，MySQL已经自动对其建立了Unique Index，无需重复再在上面建立索引了。

搜索到的一段解释：

Note that “primary” is called PRIMARY KEY not INDEX.

KEY is something on the logical level, describes your table and database design (i.e. enforces referential integrity …)

INDEX is something on the physical level, helps improve access time for table operations.

Behind every PK there is (usually) unique index created (automatically).

# mysql中UNIQUE KEY和PRIMARY KEY有什么区别

1，Primary key的1个或多个列必须为NOT NULL，如果列为NULL，在增加PRIMARY KEY时，列自动更改为NOT NULL。而UNIQUE KEY 对列没有此要求

2，一个表只能有一个PRIMARY KEY，但可以有多个UNIQUE KEY

3，主键和唯一键约束是通过参考索引实施的，如果插入的值均为NULL，则根据索引的原理，全NULL值不被记录在索引上，所以插入全NULL值时，可以有重复的，而其他的则不能插入重复值。

    alter table t add constraint uk_t_1 unique (a,b);
    insert into t (a ,b ) values (null,1);    # 不能重复
    insert into t (a ,b ) values (null,null); # 可以重复

# 使用UNIQUE KEY

    CREATE TABLE `secure_vulnerability_warning` (
      `id` int(10) NOT NULL auto_increment,
      `date` date NOT NULL,
      `type` varchar(100) NOT NULL,
      `sub_type` varchar(100) NOT NULL,
      `domain_name` varchar(128) NOT NULL,
      `url` text NOT NULL,
      `parameters` text NOT NULL,
      `hash` varchar(100) NOT NULL,
      `deal` int(1) NOT NULL,
      `deal_date` date default NULL,
      `remark` text,
      `last_push_time` datetime default NULL,
      `push_times` int(11) default '1',
      `first_set_ok_time` datetime default NULL,
      `last_set_ok_time` datetime default NULL,
      PRIMARY KEY  (`id`),
      UNIQUE KEY `date` (`date`,`hash`)
      ) ENGINE=InnoDB  DEFAULT CHARSET=utf8

UNIQUE KEY的用途：主要是用来防止数据插入的时候重复的。

1，创建表时

    CREATE TABLE Persons(
      Id_P int NOT NULL,
      LastName varchar(255) NOT NULL,
      FirstName varchar(255),
      Address varchar(255),
      City varchar(255),
      UNIQUE (Id_P))

如果需要命名 UNIQUE 约束，以及为多个列定义 UNIQUE 约束，请使用下面的 SQL 语法：

    CREATE TABLE Persons(
      Id_P int NOT NULL,
      LastName varchar(255) NOT NULL,
      FirstName varchar(255),
      Address varchar(255),
      City varchar(255),
      CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName))

2，当表已被创建时，如需在 "Id_P" 列创建 UNIQUE 约束，请使用下列 SQL：

ALTER TABLE PersonsADD UNIQUE (Id_P)
如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：

ALTER TABLE PersonsADD CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)

3，撤销 UNIQUE 约束如需撤销 UNIQUE 约束，请使用下面的 SQL：

MySQL:ALTER TABLE PersonsDROP INDEX uc_PersonID





# 参考资料：
【1】https://www.jianshu.com/p/336170f4d649
