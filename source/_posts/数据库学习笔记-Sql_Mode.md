---
title: 数据库学习笔记-Sql_Mode

date: 2020-6-23 10:12:12

categories: 2020年6月

tags: [Database,MySQL]

---

sql_mode是个很容易被忽视的变量,默认值是空值,在这种设置下是可以允许一些非法操作的,比如允许一些非法数据的插入。在生产环境必须将这个值设置为严格模式,所以开发、测试环境的数据库也必须要设置,这样在开发测试阶段就可以发现问题.

<!-- more -->

sql model 常用来解决下面几类问题

(1) 通过设置sql mode, 可以完成不同严格程度的数据校验，有效地保障数据准备性。

(2) 通过设置sql model 为宽松模式，来保证大多数sql符合标准的sql语法，这样应用在不同数据库之间进行迁移时，则不需要对业务sql 进行较大的修改。

(3) 在不同数据库之间进行数据迁移之前，通过设置SQL Mode 可以使MySQL 上的数据更方便地迁移到目标数据库中。

## 实际遇到问题
执行到查询语句时报错
```
<select id="selectByQuery" parameterType="com.microloan.boss.jhjj.model.form.UserQuery" resultType="com.microloan.boss.jhjj.model.vo.UserVO">
    select a.id as userId, name as userName, phone, email, identity_id as identityId, sys_type as sysType,
    a.created_at as createdAt, ip, device_id as deviceId, nick_name as nickName, a.user_type as userType, b.id as wechatId
    from users as a
    <if test="1 != wechat">
      left
    </if>
    join wechat_users as b
    on a.id = b.user_id
    where 1 = 1
    <if test="0 == wechat">
      and b.id is null
    </if>
    <if test="userId != null and userId != ''">
      and a.id = #{userId}
    </if>
    <if test="userName != null and userName != ''">
      and name = #{userName}
    </if>
    <if test="phone != null and phone != ''">
      and phone = #{phone}
    </if>
    <if test="identityId != null and identityId != ''">
      and identity_id = #{identityId}
    </if>
    <if test="sysType != null and sysType != ''">
      and a.sys_type = #{sysType}
    </if>
    <if test="ip != null and ip != ''">
      and ip = #{ip}
    </if>
    <if test="deviceId != null and deviceId != ''">
      and device_id = #{deviceId}
    </if>
    <if test="fromDate != null and fromDate != ''">
      and a.created_at &gt;= #{fromDate}
    </if>
    <if test="toDate != null and toDate != ''">
      and a.created_at &lt;= #{toDate}
    </if>
    group by a.id
  </select>
```
错误信息为：
```
{"status":"fail","msg":"\n### Error querying database.  Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Expression #10 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'microloan.b.nick_name' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by\n### The error may involve com.microloan.boss.jhjj.mapper.UsersMapper.selectByQuery-Inline\n### The error occurred while setting parameters\n### Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Expression #10 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'microloan.b.nick_name' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by\n; bad SQL grammar []; nested exception is com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Expression #10 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'microloan.b.nick_name' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by","data":null}
```

## 解决方法[^1]：

通过报错信息可以看到是 sql_mode=only_full_group_by 问题，

于是我们在mysql命令行输入：select @@sql_mode 查询当前数据库的默认sql_mode：

# sql_mode 配置解析
### ONLY_FULL_GROUP_BY

对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中。简而言之，就是SELECT后面接的列必须被GROUP BY后面接的列所包含。如：
select a,b from table group by a,b,c; (正确)
select a,b,c from table group by a,b; (错误)
这个配置会使得GROUP BY语句环境变得十分狭窄，所以一般都不加这个配置

### NO_AUTO_VALUE_ON_ZERO

该值影响自增长列的插入。默认设置下，插入0或NULL代表生成下一个自增长值。（不信的可以试试，默认的sql_mode你在自增主键列设置为0，该字段会自动变为最新的自增值，效果和null一样），如果用户希望插入的值为0（不改变），该列又是自增长的，那么这个选项就有用了。

### STRICT_TRANS_TABLES

在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做限制。（InnoDB默认事务表，MyISAM默认非事务表；MySQL事务表支持将批处理当做一个完整的任务统一提交或回滚，即对包含在事务中的多条语句要么全执行，要么全部不执行。非事务表则不支持此种操作，批处理中的语句如果遇到错误，在错误前的语句执行成功，之后的则不执行；MySQL事务表有表锁与行锁非事务表则只有表锁）

### NO_ZERO_IN_DATE

在严格模式下，不允许日期和月份为零

### NO_ZERO_DATE

设置该值，mysql数据库不允许插入零日期，插入零日期会抛出错误而不是警告。

### ERROR_FOR_DIVISION_BY_ZERO

在INSERT或UPDATE过程中，如果数据被零除，则产生错误而非警告。如 果未给出该模式，那么数据被零除时MySQL返回NULL

### NO_AUTO_CREATE_USER

禁止GRANT创建密码为空的用户

### NO_ENGINE_SUBSTITUTION

如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常

### PIPES_AS_CONCAT

将”||”视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样的，也和字符串的拼接函数Concat相类似

### ANSI_QUOTES

启用ANSI_QUOTES后，不能用双引号来引用字符串，因为它被解释为识别符

## 去除sql_mode中的ONLY_FULL_GROUP_BY[^2]
首先查询当前的sql_mode，分为全局的和当前session的。
###  方式一：
```
select @@session.sql_mode;

set session sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';

```
此方法只在当前会话中生效，关闭当前会话就不生效了。
###  方式二：
```
select @@global.sql_mode;

set global sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```
此方法在当前服务中生效，重新MySQL服务后失效


### 方法三：

在mysql的安装目录下，或my.cnf文件(windows系统是my.ini文件)，新增 sql_mode = ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION，

添加my.cnf如下：
```
    [mysqld]

    sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER
```


然后重启mysql。

此方法永久生效.当然生产环境上是禁止重启MySQL服务的，所以采用方式二加方式三来解决线上的问题，那么即便是有一天真的重启了MySQL服务，也会永久生效了。

如果设置的是宽松模式，那么我们在插入数据的时候，即便是给了一个错误的数据，也可能会被接受，并且不报错，例如：在创建一个表时，该表中有一个字段为name，给name设置的字段类型时char(10)，如果我在插入数据的时候，其中name这个字段对应的有一条数据的长度超过了10，例如'1234567890abc'，超过了设定的字段长度10，那么不会报错，并且取前十个字符存上，也就是说这个数据被存为了'1234567890',而'abc'就没有了，但是我们知道，我们给的这条数据是错误的，因为超过了字段长度，但是并没有报错，并且mysql自行处理并接受了，这就是宽松模式的效果，其实在开发、测试、生产等环境中，我们应该采用的是严格模式，出现这种错误，应该报错才对，所以MySQL5.7版本就将sql_mode默认值改为了严格模式，并且我们即便是用的MySQL5.6，也应该自行将其改为严格模式，而MySQL等等的这些数据库，都是想把关于数据的所有操作都自己包揽下来，包括数据的校验，其实好多时候，我们应该在自己开发的项目程序级别将这些校验给做了，虽然写项目的时候麻烦了一些步骤，但是这样做之后，我们在进行数据库迁移或者在项目的迁移时，就会方便很多。

 
[^1]:https://blog.csdn.net/Abysscarry/article/details/79468411

[^2]:https://cloud.tencent.com/developer/article/1338284