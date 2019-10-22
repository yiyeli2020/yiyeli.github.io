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

## 执行步骤：

将shangtongdai的sbt仓库文件内容复制到~/.sbt/repositories 中

    cd shangtongdai/
    cat sbtrepositories -> ~/.sbt/repositories

再执行

    sbt "project task-scheduler" run
其中要执行的项目文件是 task-scheduler
遇到问题：

    sbt.ResolveException: unresolved dependency: net.koofr#play2-sprites;1.1.3-SNAPSHOT: not found

在项目中搜索play2-sprites然后注释掉，因为该插件版本库现在已不支持。
