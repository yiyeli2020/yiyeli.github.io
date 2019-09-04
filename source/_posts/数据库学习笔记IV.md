title: 数据库学习笔记III
date: 2019-9-4 10:00:12
categories: 2019年9月
tags: [Mysql]

---

SQL插入，多表查询


<!-- more -->

# 权限配置
在reporting数据库中找到auth_users 表，代表的是用户信息，auth_roles代表的是不同的角色信息

auth_users_roles_rela代表的是不同用户所具有的角色，auth_resources代表资源，其中要添加之前写的inner和core里controller所具有的方法，type为FUNCTION，url就是执行的url，例如/notes/updateNote，level设置为1
auth_role_resources_rela是不同角色所具有的资源信息，
现在要使用SQL语句添加新的资源，将用户yang.kai


# INSERT语句

基本语法是：

    INSERT INTO <表名> (字段1, 字段2, ...) VALUES (值1, 值2, ...);

例如：
    INSERT INTO auth_resources
    (resource_name, external_id,resource_type,application_name,resource_url,status,parent_id,level,sort)
    VALUES
    ('根据ID删除笔记',-1,'FUNCTION','reporting','/notes/deleteNote','ACTIVE',0,1,0)

还可以同时添加多条数据：

    INSERT INTO <表名> (字段1, 字段2, ...)
    VALUES (值1, 值2, ...),
    (值3, 值4, ...),
    (值5, 值6, ...);

例如在auth_role_resources_rela中添加不同角色所拥有的资源信息：
    INSERT INTO auth_role_resources_rela
    (role_id,resource_id,status,created,updated)
    VALUES
    (3,43,'ACTIVE',NOW(),NOW()),
    (3,44,'ACTIVE',NOW(),NOW()),
    (3,45,'ACTIVE',NOW(),NOW()),
    (3,46,'ACTIVE',NOW(),NOW()),
    (3,47,'ACTIVE',NOW(),NOW()),
    (3,48,'ACTIVE',NOW(),NOW()),
    (3,49,'ACTIVE',NOW(),NOW()),
    (3,50,'ACTIVE',NOW(),NOW()),
    (3,51,'ACTIVE',NOW(),NOW()),
    (3,52,'ACTIVE',NOW(),NOW()),
    (3,53,'ACTIVE',NOW(),NOW());



# 连接查询
auth_users表，代表的是用户信息，auth_roles代表的是不同的角色信息，

auth_users_roles_rela代表的是不同用户所具有的角色，

现在要根据用户账号查询某个用户所具有的所有角色id

    SELECT role_id
    FROM auth_users
    INNER JOIN auth_user_roles_rela
    ON auth_users.id=auth_user_roles_rela.user_id
    WHERE auth_users.account='kai.yang'

再根据角色id到auth_role_resources_rela中找到该角色所拥有的资源resource_id
    SELECT resource_id
    FROM auth_role_resources_rela
    WHERE role_id=(
      SELECT role_id
      FROM auth_users
      INNER JOIN auth_user_roles_rela
      ON auth_users.id=auth_user_roles_rela.user_id
      WHERE auth_users.account='kai.yang'
      )

根据资源id连接查询auth_resources中的资源名称

    SELECT resource_name
    FROM auth_resources
    INNER JOIN auth_role_resources_rela
    ON auth_resources.id=auth_role_resources_rela(
      SELECT resource_id
      FROM auth_role_resources_rela
      WHERE role_id=(
        SELECT role_id
        FROM auth_users
        INNER JOIN auth_user_roles_rela
        ON auth_users.id=auth_user_roles_rela.user_id
        WHERE auth_users.account='kai.yang'
        )
      )  auth_role_resources_rela

## 注意INNER JOIN查询的写法

    先确定主表，仍然使用FROM <表1>的语法；

    再确定需要连接的表，使用INNER JOIN <表2>的语法；

    然后确定连接条件，使用ON <条件...>，这里的条件是auth_users.id=auth_user_roles_rela.user_id，表示auth_users表的id列与auth_user_roles_rela表的user_id列相同的行需要连接；

    可选：加上WHERE子句、ORDER BY等子句。

    使用别名不是必须的，但可以更好地简化查询语句。

## 内连接和外连接的区别：

假设查询语句是：

    SELECT ... FROM tableA ??? JOIN tableB ON tableA.column1 = tableB.column2;

我们把tableA看作左表，把tableB看成右表，那么

***INNER JOIN是选出两张表都存在的记录***

***LEFT OUTER JOIN是选出左表存在的记录***


***RIGHT OUTER JOIN是选出右表存在的记录***


***FULL OUTER JOIN则是选出左右表都存在的记录***
