---
title: SpringBoot学习笔记XI
date: 2019-10-15 10:53:12
categories: 2019年10月
tags: [SpringBoot，Java]

---

SpringBoot系统学习。

<!-- more -->

1。迁移时以原数据库的描述作为标题
2。mysql数据源改为shangtongdai
3。迁移时默认状态为关闭，试验性迁移几条后发送给自己后再进行批量迁移
4。和前端沟通将查询tag改为查询标题
5。cms分享材料准备

id=1612 ，1613，1614，1618 数据源有问题

1.共用笔记模块：笔记添加共用笔记模块,数据库要加笔记类型,共有笔记可单独列表展示,所有人可见
解决方案：
后端改动：

  tbl_notes表增加新字段note_type，有public和private两种。
  在NotesController笔记的增删改查接口都要增加相应的属性。
  原有的queryNoteByTag和queryNotes接口都只能查询到当前操作人和public属性的笔记。

前端改动：

  需要在保存为笔记-》设置笔记信息中添加保存为公共笔记还是私有笔记等选项。
  “我的笔记”菜单名字改动，因为查询到的是公共笔记和我的笔记的并集。
  请输入查询tag改为请输入查询标题


 2.对慢sql定时的队列延迟处理：系统现在有慢sql限制,默认最多同时单台服务器最多执行15个,对于配置的定时sql进行延迟队列处理

 解决方案：
    使用DataSourcePoolWrapper数据链接池包装类，使用ScheduledExecutorService实现延迟任务队列。
    遇到慢sql时先判断当前线程池中的慢sql数量，如果有可用慢sql资源时则加入线程池，然后进行慢sql数量的更新和操作。
    如果当前的慢sql池剩余资源不足，然后将慢sql加入延迟任务队列，延迟指定时间，再次执行该sql时如果仍无可用慢sql资源时，则再次加入延迟队列，指定延迟时间，如果有可用慢sql资源时则加入线程池，然后进行慢sql数量的更新和操作。


3.支持oracle库数据源,数据源添加执行类型字段,判断能否语法优化部分sql，针对oracle和mysql，DB2做优化：支持dianxiao的oracle数据库配置

解决方案：

使用DataSourceProperties和application-env.yml中的driverClassName作为区分字段，来对数据源类型进行判断。

4.添加慢sql黑白名单记录：对于慢sql的执行进行黑名单记录,记录执行人.执行耗时,执行sql(同一个sql记录一次),执行次数

解决方案：

新建黑名单表tbl_sql_warning，属性有id，sql_note,operator_no,cost_time,count,创建时间和更新时间，可以酌情增加数据源，查询类型,md5_tag，memo等属性，preProcess中添加相关逻辑，确认是慢sql时在黑名单表中搜索是否已存在，如果已存在则执行次数加一，更新耗时和更新时间，如不存在则添加相应记录。


5. 添加表结构框,例子sql:select * from information_schema.tables where TABLE_SCHEMA='shangtongdai' 

解决方案：
后端：

ToolQueryController中增加接口selectSchema根据选择的数据库和表拼接sql语句sql:select * from information_schema.tables where TABLE_SCHEMA='shangtongdai' ，然后调用baseExecute执行

前端：
在Report Tool中增加查询表结构的按钮，选择了查询的库之后点击查询表结构按钮调用selectSchema接口
# 10.21
report测试环境变更，在lain3上部署新的环境，注意在更新中修改URL和端口
排查时可以在容器Prods的日志中的>_ 命令行中输入查询命令来搜索

cms技术分享修改

# 10.22

定时任务迁移
重构shangtongdai-task：
#以下几个个是report-tool 定时调用执行 report-tool 相关开发看一下
*/5 8-20 * * * curl -d "" "localhost:8003/sms/common/CommonSQL?sqlId=1280&smsTemplateId=100010&debug=false" > /dev/null
*/5 8-20 * * * curl -d "" "localhost:8003/sms/common/CommonSQL?sqlId=1292&smsTemplateId=100011&debug=false" > /dev/null
45 8 1 * * curl -d "" "localhost:8003/huifang/common/CommonSQL?sqlId=1335&debug=false" > /dev/null #预催收T-
把里面绑定的端口都改为9000
*/5 8-20 * * * curl -d "" "localhost:9000/sms/common/CommonSQL?sqlId=1280&smsTemplateId=100010&debug=false" > /dev/null
## 执行步骤：

将shangtongdai的sbt仓库文件内容复制到~/.sbt/repositories 中

    cd shangtongdai/
    cat sbtrepositories > ~/.sbt/repositories

找到readme.md，按照其中的步骤执行
如果在本地使用apollo，需要在根目录 /opt/settings下面新建server.properties
 内容为env=DEV
 则可以读取测试环境apollo配置

再使用sbt-idea配置IDE

    sbt "gen-idea no-classifiers" # no-classifiers表示不下载源代码，若删除会比较慢

然后执行

    sbt "project task-scheduler" run

其中要执行的项目文件是 task-scheduler

遇到问题：

    sbt.ResolveException: unresolved dependency: net.koofr#play2-sprites;1.1.3-SNAPSHOT: not found

在项目中搜索play2-sprites然后注释掉，因为该插件版本库现在已不支持。
要更新为：

    addSbtPlugin("net.koofr" % "play2-sprites" % "1.1.4-SNAPSHOT")
在该文件中找到此行

    resolvers += "internal maven snapshots" at "http://artifactory.yxapp.xyz/artifactory/mvn-snapshot/"
然后更新为

    resolvers += "internal maven snapshots" at "http://mirrors.bdp.idc/libs-snapshot-local/"



# 10.23
1.cms技术分享文档整理：
full_flow 和状态机展开详细说一下，总结一下

信审3.0 接口找测试要账号在本地测一下，在cms页面的初审终审复核按钮中找到哪个和这些接口对应

2.定时任务重构
3.STD-5213
1.1添加共用笔记模块：
	1. 说明:
	笔记添加共用笔记模块,数据库要加笔记类型,共有笔记可单独列表展示,所有人可见
	2. 后端改动：

  1. tbl_notes表增加新字段note_type，有public和private两种。
  2. 在NotesController笔记的增删改查接口都要增加相应的属性。
  3. 原有的queryNoteByTag和queryNotes接口都只能查询到当前操作人和public属性的笔记。

1.3 支持oracle库数据源
  问题:
  数据源添加执行类型字段,判断能否语法优化部分sql，针对oracle和mysql，DB2做优化：支持dianxiao的oracle数据库配置---


  解决方案：

  	1. 使用DataSourceProperties和application-env.yml添加driverType作为区分字段，来对数据源类型进行判断, mysql or oracle。
  	2. 对于sql优化部分,判断如果为oracle暂不做优化处理
  	3. oracle 能够正常查询数据




1.4.  添加表结构展示功能
  目标:
  查询表时,经常忘了表名,和哪些表存在,添加此功能便于使用


  解决方案
  后端：
  示例 ,添加restful 接口 --/tool/tables/{datasourceName}
  ToolQueryController中增加接口上述根据选择的数据库和表拼接sql语句sql: select * from information_schema.tables where TABLE_SCHEMA='shangtongdai'，然后执行sql

  采集字段有:TABLE_SCHEMA (库名),TABLE_NAME(表名),TABLE_ROWS(数据量),TABLE_COMMENT(表说明)
1.6 定时配置,添加‘无数据是否发送’单选框

  前段,添加页面展示






  后端对此参数进行处理,对于配置为否的定时,当查询数据为0是,不进行邮件发送


  1.7 反馈邮件类型--》改为 发送类型
  选项变为,1、邮件正文,2、邮件附件、3、短信


1.7 定时查询添加 id, 收件人 ,创建者三个查询条件
  1.后段需要接口改动

  2.前端需要添加查询条件

11.05
一.成员管理页面:

查询页面:- - 李一野
1. 分页查询所有操作员信息 reporting.auth_users
2. 冻结/解冻 操作员 reporting.auth_users 中的status修改

二.角色树管理

功能树管理页面:
1. 根据当前用户获取所有功能/报表展示树信息--李一野

从user中找到当前用户，再找出具有哪些角色，auth_roles中根据role_type获取信息，FUNCTION和MENU属于功能信息，REPORT属于报表信息,在layout_id中根据格式如22-23-34的可以看到当前所属id为34，上一层为23，再上一层为22，根据改层级返回树形json串。

返回的json串也要是树形嵌套的

添加另一个同名用户yangkai38@creditease.com，其类型为platform，下面有其它角色，显示树结构时也有这部分的信息（即上一层，目前只能显示余下层的树信息）



四. 角色树维护:

查询页面:
查询页面:
1. 分页查询接口--李一野
auth_roles分页查询，根据页面显示相应的信息

11.7
目前前端拿不到用户邮箱， 接口里加一个是否是本人添加的字段吧
前端根据这个字段决定是否可以修改或者删除


表结构展示功能，把这个表结构的名字换成中文吧，库名,表名,已有数据行数,表说明,字段信息，"TABLE_ROWS"这个去掉吧，不准


11.13
功能树管理页面:
1. 根据当前用户获取所有功能/报表展示树信息--李一野
添加另一个同名用户yangkai38@creditease.com，其类型为platform，下面有其它角色，显示树结构时也有这部分的信息（即上一层，目前只能显示余下层的树信息）

2.修改笔记遇到名字重复的bug
3.共有笔记返回DTO里加入属性字段modify：是否能修改，共有笔记不能修改，只能查看

11.14
auth_users表unique-key只用account一列
auth_roles表 unique-key加入role—type一列
软删除auth_roles表中的application_name，并在涉及到的接口进行修改
查询所有用户中的应用访问权限现在只显示 所有level=1角色的role—name
修改角色信息中所属应用返回所有level=1 的role-name
修改上级角色返回所有level大于1的role-name

技术研究：
XXL-JOB缓存工具：自动降级，引入包，

# 参考资料：
【1】 http://std-docs.laincloud.in/%E5%BC%80%E5%8F%91%E9%83%A8%E7%BD%B2/develop-guidelines/
