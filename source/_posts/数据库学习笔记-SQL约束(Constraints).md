title: 数据库学习笔记-SQL约束(Constraints)

date: 2020-6-9 20:24:12

categories: 2020年6月

tags: [Database，MySQL]

---

SQL约束(Constraints)

<!-- more -->

# SQL 约束[^1]
约束用于限制加入表的数据的类型。

1. 按照约束对象分类:

(1)域完整性约束条件: 施加在某一列上, 比如sage<25 and sage<40

(2)关系完整性约束条件: 施加在表上, 涉及多列, 2<=hours/credit<=16

2. 按照约束来源分类:

(1)结构约束: 主键约束, 外键约束,是否允许空值等: primary key, foreign key, not null

(2) 内容约束:  取值范围, check(sage<25 and sage<40)

3. 按照约束状态分类:

(1) 静态约束: 要求DB在任何时候都要满足的约束

(2) 动态约束: DB改变状态时要满足的约束, 例如salary 只能加不能减, 不能由1000改为500.---> 触发器

SQL支持如下几种约束: 

静态约束中的列完整性 与表完整性,  动态约束中的触发器

# 静态约束
可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句）。

注意: create table 中的约束条件 可以在后面根据需要进行撤销 ,也可以追加约束

    alter  table  tablename + 
    
    add 追加约束,  也可以增加新的一列
    
    drop 删除一列的约束,或者删除一列,
    
    modify 修改

我们将主要探讨以下几种约束：
    
    NOT NULL
    UNIQUE
    PRIMARY KEY
    FOREIGN KEY
    CHECK
    DEFAULT
还有断言assertion
## SQL NOT NULL 约束
NOT NULL 约束强制列不接受 NULL 值。

NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。

下面的 SQL 语句强制 "Id_P" 列和 "LastName" 列不接受 NULL 值：
```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

## SQL UNIQUE 约束
UNIQUE 约束唯一标识数据库表中的每条记录。

UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。

PRIMARY KEY 拥有自动定义的 UNIQUE 约束。

请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

### SQL UNIQUE Constraint on CREATE TABLE
下面的 SQL 在 "Persons" 表创建时在 "Id_P" 列创建 UNIQUE 约束：

MySQL:
```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
UNIQUE (Id_P)
)
```
SQL Server / Oracle / MS Access:
```
CREATE TABLE Persons
(
Id_P int NOT NULL UNIQUE,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```
如果需要命名 UNIQUE 约束，以及为多个列定义 UNIQUE 约束，请使用下面的 SQL 语法：

MySQL / SQL Server / Oracle / MS Access:
```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
)
```

### SQL UNIQUE Constraint on ALTER TABLE
当表已被创建时，如需在 "Id_P" 列创建 UNIQUE 约束，请使用下列 SQL：

MySQL / SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
ADD UNIQUE (Id_P)
```
如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：

MySQL / SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
ADD CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
```
撤销 UNIQUE 约束
如需撤销 UNIQUE 约束，请使用下面的 SQL：

MySQL:
```
ALTER TABLE Persons
DROP INDEX uc_PersonID
```
SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
DROP CONSTRAINT uc_PersonID
```
## SQL PRIMARY KEY 约束
PRIMARY KEY 约束唯一标识数据库表中的每条记录。

主键必须包含唯一的值。

主键列不能包含 NULL 值。

每个表都应该有一个主键，并且每个表只能有一个主键。

### SQL PRIMARY KEY Constraint on CREATE TABLE
下面的 SQL 在 "Persons" 表创建时在 "Id_P" 列创建 PRIMARY KEY 约束：

MySQL:
```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (Id_P)
)
```
SQL Server / Oracle / MS Access:
```
CREATE TABLE Persons
(
Id_P int NOT NULL PRIMARY KEY,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```
如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

MySQL / SQL Server / Oracle / MS Access:
```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
)
```
### SQL PRIMARY KEY Constraint on ALTER TABLE
如果在表已存在的情况下为 "Id_P" 列创建 PRIMARY KEY 约束，请使用下面的 SQL：

MySQL / SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
ADD PRIMARY KEY (Id_P)
```
如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

MySQL / SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
```
注释：如果您使用 ALTER TABLE 语句添加主键，必须把主键列声明为不包含 NULL 值（在表首次创建时）。

### 撤销 PRIMARY KEY 约束
如需撤销 PRIMARY KEY 约束，请使用下面的 SQL：

MySQL:
```
ALTER TABLE Persons
DROP PRIMARY KEY
```
SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
DROP CONSTRAINT pk_PersonID
```

## SQL FOREIGN KEY 约束
一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。

让我们通过一个例子来解释外键。请看下面两个表：

"Persons" 表：
```
Id_P	LastName	FirstName	Address	        City
1	    Adams	    John	    Oxford Street	London
2	    Bush	    George	    Fifth Avenue	New York
3	    Carter	    Thomas	    Changan Street	Beijing
```
"Orders" 表：
```
Id_O	OrderNo	Id_P
1	    77895	3
2	    44678	3
3	    22456	1
4	    24562	1
```
请注意，"Orders" 中的 "Id_P" 列指向 "Persons" 表中的 "Id_P" 列。

"Persons" 表中的 "Id_P" 列是 "Persons" 表中的 PRIMARY KEY。

"Orders" 表中的 "Id_P" 列是 "Orders" 表中的 FOREIGN KEY。

FOREIGN KEY 约束用于预防破坏表之间连接的动作。

FOREIGN KEY 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。

### SQL FOREIGN KEY Constraint on CREATE TABLE
下面的 SQL 在 "Orders" 表创建时为 "Id_P" 列创建 FOREIGN KEY：

MySQL:
```
CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
PRIMARY KEY (Id_O),
FOREIGN KEY (Id_P) REFERENCES Persons(Id_P)
)
```
SQL Server / Oracle / MS Access:
```
CREATE TABLE Orders
(
Id_O int NOT NULL PRIMARY KEY,
OrderNo int NOT NULL,
Id_P int FOREIGN KEY REFERENCES Persons(Id_P)
)
```
如果需要命名 FOREIGN KEY 约束，以及为多个列定义 FOREIGN KEY 约束，请使用下面的 SQL 语法：

MySQL / SQL Server / Oracle / MS Access:
```
CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
PRIMARY KEY (Id_O),
CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)
)
```
### SQL FOREIGN KEY Constraint on ALTER TABLE
如果在 "Orders" 表已存在的情况下为 "Id_P" 列创建 FOREIGN KEY 约束，请使用下面的 SQL：

MySQL / SQL Server / Oracle / MS Access:
```
ALTER TABLE Orders
ADD FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)
```
如果需要命名 FOREIGN KEY 约束，以及为多个列定义 FOREIGN KEY 约束，请使用下面的 SQL 语法：

MySQL / SQL Server / Oracle / MS Access:
```
ALTER TABLE Orders
ADD CONSTRAINT fk_PerOrders
FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)
```
### 撤销 FOREIGN KEY 约束
如需撤销 FOREIGN KEY 约束，请使用下面的 SQL：

MySQL:
```
ALTER TABLE Orders
DROP FOREIGN KEY fk_PerOrders
SQL Server / Oracle / MS Access:
ALTER TABLE Orders
DROP CONSTRAINT fk_PerOrderss
```

## SQL CHECK 约束
CHECK 约束用于限制列中的值的范围。

如果对单个列定义 CHECK 约束，那么该列只允许特定的值。

如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制。

### SQL CHECK Constraint on CREATE TABLE
下面的 SQL 在 "Persons" 表创建时为 "Id_P" 列创建 CHECK 约束。CHECK 约束规定 "Id_P" 列必须只包含大于 0 的整数。

My SQL:
```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CHECK (Id_P>0)
)
```
SQL Server / Oracle / MS Access:
```
CREATE TABLE Persons
(
Id_P int NOT NULL CHECK (Id_P>0),
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```
如果需要命名 CHECK 约束，以及为多个列定义 CHECK 约束，请使用下面的 SQL 语法：

MySQL / SQL Server / Oracle / MS Access:
```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
)
```
### SQL CHECK Constraint on ALTER TABLE
如果在表已存在的情况下为 "Id_P" 列创建 CHECK 约束，请使用下面的 SQL：

MySQL / SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
ADD CHECK (Id_P>0)
```
如果需要命名 CHECK 约束，以及为多个列定义 CHECK 约束，请使用下面的 SQL 语法：

MySQL / SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
ADD CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
```
### 撤销 CHECK 约束
如需撤销 CHECK 约束，请使用下面的 SQL：

SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
DROP CONSTRAINT chk_Person
```
MySQL:
```
ALTER TABLE Persons
DROP CHECK chk_Person
```


## SQL DEFAULT 约束
DEFAULT 约束用于向列中插入默认值。

如果没有规定其他的值，那么会将默认值添加到所有的新记录。

### SQL DEFAULT Constraint on CREATE TABLE
下面的 SQL 在 "Persons" 表创建时为 "City" 列创建 DEFAULT 约束：

My SQL / SQL Server / Oracle / MS Access:
```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255) DEFAULT 'Sandnes'
)
```
通过使用类似 GETDATE() 这样的函数，DEFAULT 约束也可以用于插入系统值：
```

CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
OrderDate date DEFAULT GETDATE()
)
```
### SQL DEFAULT Constraint on ALTER TABLE
如果在表已存在的情况下为 "City" 列创建 DEFAULT 约束，请使用下面的 SQL：

MySQL:
```
ALTER TABLE Persons
ALTER City SET DEFAULT 'SANDNES'
```
SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
ALTER COLUMN City SET DEFAULT 'SANDNES'
```
### 撤销 DEFAULT 约束
如需撤销 DEFAULT 约束，请使用下面的 SQL：

MySQL:
```
ALTER TABLE Persons
ALTER City DROP DEFAULT
```
SQL Server / Oracle / MS Access:
```
ALTER TABLE Persons
ALTER COLUMN City DROP DEFAULT
```
## 断言 assertion 

一个断言就是一个谓词表达式, 它表达了希望数据库总能满足的条件, 表约束与 列约束就是一些特殊的断言.

还有复杂的断言 

    create assertion [assertion name] check [predicate]

那么之后数据库的每一次更新去判断是否违反该断言, 断言测试增加了数据库维护的负担, 非必需不要使用断言
使用例子：创建断言检查Person表中的有效期字段license_expiry是否都在1/JAN/2015之后

```
CREATE ASSERTION AssertLicenses 
     CHECK ( 
         NOT EXISTS ( 
             SELECT license_expiry 
             FROM Person 
             WHERE license_expiry < '1/JAN/2015' 
         ) 
    );
 ```



# 动态约束

以上 create table 中的表约束与列约束 是静态约束, 下面介绍动态约束--> 触发器 trigger

动态约束是一种过程完整性的约束, 相比之下, 之前的create table 的约束是非过程性约束

动态约束是一个程序, 该程序可以在特定的时刻被自动触发执行: 比如在一次更新之前, 

或者一次更新之后的执行. 

动态约束 intergrity constraint::=(O,P,A,R), O P A R 都需要定义, 再来回顾下 

    O : 数据集合, 约束的对象 ?: 列, 多列的元组集合
    
    P: 谓词条件: 什么样的约束?
    
    A: 触发条件: 什么时候检查?
    
    R: 响应动作: 不满足怎么办?

## 触发器[^2]

触发器是一种特殊类型的存储过程，它不同于之前的我们介绍的存储过程。触发器主要是通过事件进行触发被自动调用执行的。而存储过程可以通过存储过程的名称被调用。

触发器对表进行插入、更新、删除的时候会自动执行的特殊存储过程。触发器一般用在check约束更加复杂的约束上面。触发器和普通的存储过程的区别是：触发器是当对某一个表进行操作。诸如：update、insert、delete这些操作的时候，系统会自动调用执行该表上对应的触发器。SQL Server 2005中触发器可以分为两类：DML触发器和DDL触发器，其中DDL触发器它们会影响多种数据定义语言语句而激发，这些语句有create、alter、drop语句。

DML触发器分为：

1、 after触发器（之后触发）

    a、 insert触发器

    b、 update触发器

    c、 delete触发器



2、 instead of 触发器 （之前触发）

 

其中after触发器要求只有执行某一操作insert、update、delete之后触发器才被触发，且只能定义在表上。而instead of触发器表示并不执行其定义的操作（insert、update、delete）而仅是执行触发器本身。既可以在表上定义instead of触发器，也可以在视图上定义。



触发器有两个特殊的表：插入表（instered表）和删除表（deleted表）。这两张是逻辑表也是虚表。有系统在内存中创建者两张表，不会存储在数据库中。而且两张表的都是只读的，只能读取数据而不能修改数据。这两张表的结果总是与被改触发器应用的表的结构相同。当触发器完成工作后，这两张表就会被删除。Inserted表的数据是插入或是修改后的数据，而deleted表的数据是更新前的或是删除的数据。



Update数据的时候就是先删除表记录，然后增加一条记录。这样在inserted和deleted表就都有update后的数据记录了。注意的是：触发器本身就是一个事务，所以在触发器里面可以对修改数据进行一些特殊的检查。如果不满足可以利用事务回滚，撤销操作。

创建触发器

语法
```
create trigger tgr_name
on table_name
with encrypion –加密触发器
    for update...
as
    Transact-SQL
```
SQLite 中创建触发器（Trigger）的基本语法如下：
```
CREATE  TRIGGER trigger_name [BEFORE|AFTER] event_name 
ON table_name
BEGIN
 -- Trigger logic goes here....
END;
```
更多实例参考[^3]


[^1]:https://www.w3school.com.cn/sql/sql_constraints.asp

[^2]:https://www.cnblogs.com/hoojo/archive/2011/07/20/2111316.html

[^3]:https://wizardforcel.gitbooks.io/w3school-sql/92.html