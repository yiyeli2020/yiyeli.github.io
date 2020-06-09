title: 数据库学习笔记-SQL约束(Constraints)

date: 2020-6-9 20:24:12

categories: 2020年6月

tags: [Database，MySQL]

---

SQL约束(Constraints)

<!-- more -->

# SQL 约束[^1]
约束用于限制加入表的数据的类型。

可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句）。

我们将主要探讨以下几种约束：
    
    NOT NULL
    UNIQUE
    PRIMARY KEY
    FOREIGN KEY
    CHECK
    DEFAULT

## SQL NOT NULL 约束
NOT NULL 约束强制列不接受 NULL 值。

NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。

下面的 SQL 语句强制 "Id_P" 列和 "LastName" 列不接受 NULL 值：
    
    CREATE TABLE Persons
    (
    Id_P int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255)
    )

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



[^1]:https://www.w3school.com.cn/sql/sql_constraints.asp