title: 数据库学习笔记IV
date: 2019-9-4 10:00:12
categories: 2019年9月
tags: [Mysql]

---

SQL建表，插入，多表查询


<!-- more -->

# 权限配置
在reporting数据库中找到auth_users 表，代表的是用户信息，auth_roles代表的是不同的角色信息

auth_users_roles_rela代表的是不同用户所具有的角色，auth_resources代表资源，其中要添加之前写的inner和core里controller所具有的方法，type为FUNCTION，url就是执行的url，例如/notes/updateNote，level设置为1
auth_role_resources_rela是不同角色所具有的资源信息，
现在要使用SQL语句添加新的资源，将用户yang.kai

# 建表语句CREATE

    create table tableName(
       Id int,
       Name varchar(64),
       Salary int,
       DepartmentId int
    )

# 页面权限配置
resource_type 为MENU，level=0时为一级菜单，level=1时为二级菜单，现将http://std-report-fe.laincloud.xyz/#/reportool中的不同级别菜单的http请求url添加到reporting数据库的资源中

    INSERT INTO reporting.auth_resources
    (resource_name, external_id,resource_type,application_name,resource_url,status,parent_id,level,sort)
    VALUES
    ('首页',-1,'MENU','reporting','/','ACTIVE',0,0,0),
    ('报表',-1,'MENU','reporting','/report/home','ACTIVE',0,0,0),
    ('night watcher',-1,'MENU','reporting','/night/home','ACTIVE',0,0,0),
    ('report tool',-1,'MENU','reporting','/reportool','ACTIVE',0,0,0),
    ('短信模板',-1,'MENU','reporting','/night/template/sms','ACTIVE',0,1,0),
    ('邮件模板',-1,'MENU','reporting','/night/template/mail','ACTIVE',0,1,0),
    ('新增模板',-1,'MENU','reporting','/night/template/mail','ACTIVE',0,1,0),
    ('定时任务列表',-1,'MENU','reporting','/reportool/jobs','ACTIVE',0,1,0),
    ('我的笔记',-1,'MENU','reporting','/reportool/notes','ACTIVE',0,1,0);

再次添加一些界面：

    INSERT INTO reporting.auth_resources
    (resource_name, external_id,resource_type,application_name,resource_url,status,parent_id,level,sort)
    VALUES
    ('表结构查询',-1,'FUNCTION','reporting','/tool/tables','ACTIVE',0,1,0)


    INSERT INTO reporting.auth_resources
    (resource_name, external_id,resource_type,application_name,resource_url,status,parent_id,level,sort)
    VALUES
    ('根据当前用户获取所有报表展示树信息',-1,'FUNCTION','reporting','/auth/role/getLayout','ACTIVE',0,1,0),
    ('根据当前用户获取所有功能展示树信息',-1,'FUNCTION','reporting','/auth/role/getFunctionLayout','ACTIVE',0,1,0),
    ('分页查询角色信息',-1,'FUNCTION','reporting','/auth/role/queryRolesInfo','ACTIVE',0,1,0),
    ('分页查询所有操作员信息',-1,'FUNCTION','reporting','/auth/user/queryInfo','ACTIVE',0,1,0),
    ('更新操作员状态（冻结/解冻）',-1,'FUNCTION','reporting','/auth/user/updateStatus','ACTIVE',0,1,0)

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
    (3,1,'ACTIVE',NOW(),NOW()),
    (3,2,'ACTIVE',NOW(),NOW())

再添加一些权限

    INSERT INTO auth_role_resources_rela
    (role_id,resource_id,status,created,updated)
    VALUES
    (3,246,'ACTIVE',NOW(),NOW()),
    (3,247,'ACTIVE',NOW(),NOW()),
    (3,248,'ACTIVE',NOW(),NOW()),
    (3,249,'ACTIVE',NOW(),NOW()),
    (3,250,'ACTIVE',NOW(),NOW())

例如在线上生产环境中添加角色权限
    INSERT INTO reporting.auth_user_roles_rela
    (user_id,role_id,create_user,status,created,updated)
    VALUES
    (14,1,'yiyeli@***.cn','ACTIVE',NOW(),NOW()),
    (14,5,'yiyeli@***.cn','ACTIVE',NOW(),NOW())

例如添加某个用户具有的角色信息：


    INSERT INTO reporting.auth_user_roles_rela
    (user_id,role_id,create_user,status,created,updated)
    VALUES
    (22,4,'yiyeli@***.cn','ACTIVE',NOW(),NOW()),
    (22,5,'yiyeli@***.cn','ACTIVE',NOW(),NOW())

    INSERT INTO reporting.auth_user_roles_rela
    (user_id,role_id,create_user,status,created,updated)
    VALUES
    (41,3,'kaiyang38@***.cn','ACTIVE',NOW(),NOW()),
    (41,4,'kaiyang38@***.cn','ACTIVE',NOW(),NOW())


    INSERT INTO reporting.auth_user_roles_rela
    (user_id,role_id,create_user,status,created,updated)
    VALUES
    (41,21,'zhenxinlin2','ACTIVE',NOW(),NOW()),
    (41,23,'zhenxinlin2','ACTIVE',NOW(),NOW())
在角色表中添加新的角色

    INSERT INTO reporting.auth_roles
    (application_name,department,role_name,layout_id,role_type,role_level,resource_level,description,status,created,updated)
    VALUES

    ('platform','商通贷','platform-FUNCTION','47','FUNCTION',1,'HIGHT', '角色树根节点','ACTIVE',NOW(),NOW()),
    ('reporting','商通贷','reporting-FUNCTION','47','FUNCTION',1,'HIGHT', '角色树根节点','ACTIVE',NOW(),NOW())


# 连接查询
auth_users表，代表的是用户信息，auth_roles代表的是不同的角色信息，

auth_users_roles_rela代表的是不同用户所具有的角色，

现在要根据用户账号查询某个用户所具有的所有角色id

    SELECT role_id
    FROM auth_users
    INNER JOIN auth_user_roles_rela
    ON auth_users.id=auth_user_roles_rela.user_id
    WHERE auth_users.account='kai.yang'

根据角色id到auth_role_resources_rela中找到该角色所拥有的资源resource_id
    SELECT resource_id
    FROM auth_role_resources_rela
    WHERE role_id=(
      SELECT role_id
      FROM auth_users
      INNER JOIN auth_user_roles_rela
      ON auth_users.id=auth_user_roles_rela.user_id
      WHERE auth_users.account='kai.yang'
      )

根据角色id连接查询auth_resources中的资源名称

    SELECT auth_resources.id,resource_name,resource_type,resource_url
    FROM auth_resources
    INNER JOIN auth_role_resources_rela
    ON auth_resources.id=auth_role_resources_rela.resource_id
    WHERE auth_role_resources_rela.role_id=(
      SELECT role_id
      FROM auth_users
      INNER JOIN auth_user_roles_rela
      ON auth_users.id=auth_user_roles_rela.user_id
      WHERE auth_users.account='kai.yang'
      )

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
