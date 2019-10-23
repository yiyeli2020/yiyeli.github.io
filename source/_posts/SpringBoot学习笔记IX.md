---
title: SpringBoot学习笔记IX
date: 2019-8-5 11:22:12
categories: 2019年8月
tags: [SpringBoot，Java]

---

CMS系统学习。

<!-- more -->
# 任务
cn.kael.query.inner中逻辑写在biz中，config/InnerDbConfig是内部数据库配置，
controller接口，dao数据库自动生成，


kael_query表中的tbl_jobs是定时任务相关的表,其中tag是标签名字，暂时是唯一索引，入参，不能改动，sql_note是sql语句内容，operator_no是操作者编号，cron是定时表达式，receiver是收件人，memo是备注内容，task_status是定时任务状态，datasoure_name要执行任务sql的数据源，out_type是输出类型（正文和附件）

增加定时任务的分页查询，包括附加条件的分页查询，比如模糊匹配名字，接收者的分页查询，固定时间范围内的分页查询,把分页查询都写到一个接口里，根据传进来的参数进行不同的查询。

增删改查完本地库，去Quartz 完成增删改查定时任务。用@Autowired注解注入

cn.creditease.std.query.kael.core.task.TaskManager

kaelContext.setEmailMode(tblJobsEntity.getOutType());

根据枚举的邮件输出类型设置emailMode为0或1

tbl_notes是笔记相关的表，tag是笔记名字，暂时不能改动，后面可以置为能改动的列，根据id来查，两张表都是tag和operator_no不能改动，其它可以改动

tbl_sql_warning 是SQL语句对应的表，

完成两张表到增删改查功能，在DatasourceInfoController中查询并返回可用的数据源列表。

中秋节前，9.12：
CMS业务分享PPT：
常用页面操作分享，
核心流程分享：进件流程，状态机修改流程，有哪些限制，
CMS常用表例如transport，history分享

# 常见问题
## 出现找不到主类的错误

首先将porm文件右键选择Add to the maven project添加到右侧到maven库中，然后再将工程文件右键Mark Directory as->Sources Root

## java:程序包XXXX不存在

可以先删去导入的该包然后用快捷键Alt+Enter选择add to library搜索导入该包


不能全写到一个UserService里，要有相应的区分

ServiceImpl参数不能直接传实体，不利于可读性和可维护性，用哪些传哪些，里面只写数据库相关操作，涉及EntityMapper的，逻辑内容写到biz文件夹中,Service里返回Entity而不是DTO，biz中返回DTO

mybatis-config不用加到resources.local里面，在resources里面就行

config文件夹内还要有相应的数据库自动化配置和Mybatis配置文件

如果出现循环依赖的问题，例如query.inner和query.core相互依赖，则在两个子项目的porm.xml中去掉彼此的依赖项目

Maven执行clean和install在生命周期Lifecycle中选择相应的命令执行即可

## Mybatis自动生成实体
pom.xml中的自动生成mapper.xml的依赖后期要注释掉，否则每次install maven都会覆盖掉原先的。

    CharacterEncoding=UTF-8&autoReconnect=true&failOverReadOnly=false
    com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Could not create connection to database server. Attempted reconnect 3 times. Giving up.
    	at sun.reflect.GeneratedConstructorAccessor64.newInstance(Unknown Source)
    	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
    	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)

## Mybatis自动生成实体时generator无法将数据表中TEXT字段和TIMESTAMP映射问题
在mybatis generator对在数据库中表进行映射时发现它无法将表中的TEXT类型和TIMESTAMP的字段映射到实体文件中，

我们需要在配置文件generatorConfig.xml中的 <table></table> 标签中做一定的修改。

比如表rpt_sql_query_task，它的商品描述sql_query是TEXT类型， 那么我们可以在<table> 标签中添加<columnOverride column="sql_query" jdbcType="VARCHAR"></columnOverride> 即可。

同样也可以修改TIMESTAMP格式的字段。

完整代码如下：


    <table tableName="rpt_sql_query_task" domainObjectName="RptSqlQueryTaskEntity"
            enableCountByExample="false" enableUpdateByExample="true" enableDeleteByExample="false"
            enableSelectByExample="true" selectByExampleQueryId="true" enableInsert="true">
         <columnOverride column="sql_query" jdbcType="VARCHAR"></columnOverride>
         <columnOverride column="mail_to" jdbcType="VARCHAR"></columnOverride>
         <columnOverride column="ext_info" jdbcType="VARCHAR"></columnOverride>
         <columnOverride column="created" jdbcType="TIMESTAMP"></columnOverride>
         <columnOverride column="updated" jdbcType="TIMESTAMP"></columnOverride>
     </table>

## 使用mybatis-generator生成新的DTO和mapper.xml时

遇到

    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD FAILURE
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 6.189 s
    [INFO] Finished at: 2019-10-23T17:04:33+08:00
    [INFO] Final Memory: 39M/308M
    [INFO] ------------------------------------------------------------------------
    [ERROR] Failed to execute goal org.mybatis.generator:mybatis-generator-maven-plugin:1.3.5:generate (Generate MyBatis Artifacts) on project kael-inner: Execution Generate MyBatis Artifacts of goal org.mybatis.generator:mybatis-generator-maven-plugin:1.3.5:generate failed: Cannot resolve classpath entry: /Users/liyiye/yiye-project/CreditEase/kael-query/kael-inner/src/main/resources.local -> [Help 1]
    [ERROR]
    [ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
    [ERROR] Re-run Maven using the -X switch to enable full debug logging.
    [ERROR]
    [ERROR] For more information about the errors and possible solutions, please read the following articles:
    [ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginExecutionException

    Process finished with exit code 1

项目文件中只有/Users/liyiye/yiye-project/CreditEase/kael-query/kael-inner/src/main/resources，多了一个.local

改来改去没有太好的方法，所以直接建了个resources.local文件夹然后把相应文件复制过去，就install成功了
## 驼峰命名法
方法名开头字母要小写

# 异常处理

之前处理异常的方法
