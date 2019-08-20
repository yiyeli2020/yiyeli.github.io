---
title: SpringBoot学习笔记VIII
date: 2019-8-5 11:22:12
categories: 2019年8月
tags: [SpringBoot，Java]

---

CMS系统学习。

<!-- more -->
# 任务
cn.kael.query.inner中逻辑写在biz中，config/InnerDbConfig是内部数据库配置，
controller接口，dao数据库自动生成，


kael_query表中的tbl_jobs是定时任务相关的表,其中tag是标签名字，暂时是唯一索引，入参，不能改动，sql_note是sql语句内容，operator_no是操作者编号，cron是定时任务，receiver是收件人，memo是备注内容，task_status是定时任务状态，datasour_name要执行任务sql的数据源

tbl_notes是笔记相关的表，tag是笔记名字，暂时不能改动，后面可以置为能改动的列，根据id来查，两张表都是tag和operator_no不能改动，其它可以改动

tbl_sql_warning 是SQL语句对应的表，
完成两张表到增删改查功能，在DatasourceInfoController中查询并返回可用的数据源列表。
周一前完成

# 常见问题
出现找不到主类的错误首先将porm文件右键选择Add to the maven project添加到右侧到maven库中，然后再将工程文件右键Mark Directory as->Sources Root

java:程序包XXXX不存在
可以先删去导入的该包然后用快捷键Alt+Enter选择add to library搜索导入该包

## 驼峰命名法
方法名开头字母要小写
不能全写到一个UserService里，要有相应的区分
ServiceImpl参数不能直接传实体，不利于可读性和可维护性，用哪些传哪些，里面只写数据库相关操作，涉及EntityMapper的，逻辑内容写到biz文件夹中
mybatis-config不用加到resources.local里面，在resources里面就行
config文件夹内还要有相应的数据库自动化配置和Mybatis配置文件
