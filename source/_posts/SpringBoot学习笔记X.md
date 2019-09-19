---
title: SpringBoot学习笔记X
date: 2019-8-28 17:18:12
categories: 2019年8月
tags: [SpringBoot，Java]

---

SpringBoot系统学习。

<!-- more -->
邮件网页版申请会议室，新建-》会议请求

# 写tc文件
http://jira.yxapp.in/projects/STD/summary
上面的代号：STD-4545 report-tool项目重构
用户使用的查询网址：
http://std-report-fe.laincloud.xyz/#/reportool/notes
要有用户名，有地址，把定时模块和笔记模块写上去，将该网址中的操作进行截图然后介绍如何使用

## 遇到的问题
http://std-report-fe.laincloud.xyz/#/reportool/notes
登陆时失败，去lain中的日志中去找登陆失败的原因
cn.creditease.bdp.std.authorization.client.token

## 同步定时
线上查询网址：

http://std-report-tool.yxapp.in/tool

查询列表将定时同步到kael-query的tbl_jobs表中,先将所有定时任务status都置为disable状态，然后第二次时将需要同步都都打开

    SELECT * from shangtongdai_rpt.rpt_sql_query_task
TODO：
自己写一个JDBC来建立连接，从resultSet中获取结果然后找到要查询的内容
可供参考：
cn.kael.query.core.executer.SqlExecuterWrapper.execute
cn.kael.query.core.executer.impl.execute
仿照：

    @Override
     public ResultSet execute(long maxWaitMills) {
         try {
             //1.获取链接
             //TODO:2019.9.17 自己写一个JDBC来建立连接，从resultSet中获取结果然后找到要查询的内容
             connection = ((DataSourcePoolWrapper) dataSrource).getConnection(isSlowSql, maxWaitMills);
             statement = connection
                 .prepareStatement(sql, ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);

             //2.执行实现方法
             resultSet = doExuecte(statement);

             return resultSet;


         } catch (Exception e) {
             //4.异常控制
             logger.error("执行sql异常", e);
             throw new KaelException(e.getMessage());
         }

     }

查询 kael_query.tbl_jobs中的相应内容：

      SELECT * from kael_query.tbl_jobs
      where tag='33'


删除 kael_query.tbl_jobs中的相应内容：

    DELETE FROM kael_query.tbl_jobs
    WHERE tag='3'

在kael_query.tbl_jobs加入一个字段用于记录迁移前的数据id,位置在tag之后

    ALTER TABLE kael_query.tbl_jobs ADD old_id int(11) AFTER tag

从shangtongdai_rpt.rpt_sql_query_task中复制相应的列到kael_query.tbl_jobs中：

    INSERT INTO kael_query.tbl_jobs(tag,old_id,sql_note,operator_no,cron,receiver,memo,task_status,datasource_name,out_type)
    SELECT id,id,sql_query,created_user,cron_desc,mail_to,description,status,query_type,ext_info
    from shangtongdai_rpt.rpt_sql_query_task
    where shangtongdai_rpt.rpt_sql_query_task.id=3

试验：
复制时先将所有定时任务status都置为disable状态且不复制status为“deleted”状态的数据（先复制id=33的数据）：

    INSERT INTO kael_query.tbl_jobs(tag,old_id,sql_note,operator_no,cron,receiver,memo,task_status,datasource_name,out_type)
    SELECT id,id,sql_query,created_user,cron_desc,mail_to,description,'disable',query_type,ext_info
    from shangtongdai_rpt.rpt_sql_query_task
    where shangtongdai_rpt.rpt_sql_query_task.id=33 and shangtongdai_rpt.rpt_sql_query_task.status!='deleted'

task_status为了和kael_query.tbl_jobs保持一致，复制时改为off状态


现在开始复制所有数据：

    INSERT INTO kael_query.tbl_jobs(tag,old_id,sql_note,operator_no,cron,receiver,memo,task_status,datasource_name,out_type)
    SELECT id,id,sql_query,created_user,cron_desc,mail_to,description,'off',query_type,ext_info
    from shangtongdai_rpt.rpt_sql_query_task
    where shangtongdai_rpt.rpt_sql_query_task.status!='deleted'

发现出现问题，要复制的shangtongdai_rpt.rpt_sql_query_task表中sql_query,receiver,out_type过长，所以要去kael_query.tbl_jobs中修改字段类型

    ALTER TABLE kael_query.tbl_jobs
    MODIFY sql_note text NOT NULL

    ALTER TABLE kael_query.tbl_jobs
    MODIFY receiver text

    ALTER TABLE kael_query.tbl_jobs
    MODIFY out_type text


第二次时将需要status状态同步的数据都打开，置为enable数据

    UPDATE kael_query.tbl_jobs
    INNER JOIN shangtongdai_rpt.rpt_sql_query_task
    ON kael_query.tbl_jobs.old_id=shangtongdai_rpt.rpt_sql_query_task.id
    SET kael_query.tbl_jobs.task_status=shangtongdai_rpt.rpt_sql_query_task.status

task_status为了和kael_query.tbl_jobs保持一致，同步时改为on状态

    UPDATE kael_query.tbl_jobs
    SET task_status='on'
    WHERE task_status='enable'

    UPDATE kael_query.tbl_jobs
    SET task_status='off'
    WHERE task_status='disable'

重置所有记录的status为off
    UPDATE kael_query.tbl_jobs
    SET kael_query.tbl_jobs.task_status='off'

out_type中到csv和xls复制时都转为attachment
    UPDATE kael_query.tbl_jobs
    SET out_type='attachment'
    WHERE out_type='{"emailType":"csv"}' OR out_type='{"emailType":"xls"}'

    UPDATE kael_query.tbl_jobs
    SET out_type='content'
    WHERE out_type='{"emailType":"content"}'


query_type 对应的mysql属性需要到线上到std-report-tool查询应该转成什么数据源,xyz后缀的是测试，in后缀的是生产

执行sbt run命令
在localhost:9000 中进入

首先插入一条测试的数据对数据源进行测试：

    INSERT INTO kael_query.tbl_jobs
    (tag,old_id,sql_note,operator_no,cron,receiver,memo,task_status,out_type,datasource_name)
    VALUES
    ('CronTest',888,"",'yiyeli','*/10 * * * * ?','yiyeli@creditease.cn','测试数据源','enable','attachment','shangtongdai')
在sql_note中复制入不同的sql来测试

    UPDATE kael_query.tbl_jobs
    SET datasource_name='shangtongdai'
    WHERE datasource_name='mysql'

在测试环境中只需要选择shangtongdai进行测试，把mysql改成shangtongdai，hive在测试环境也没法测，数据源配好上线测试，dianxiao在线上测试

# mysql-scheduler
四个项目：

mysql-scheduler 计算进件信息的逻辑代码

user-referral 计算进件信息的逻辑代码

std-report 查询进件

std-query


如果字段有问题例如http://std-report.yxapp.in/bdStat页面中某客户的“能否进件”字段出现错误，

先在Chrome中F12（检查）-》Network->Doc或All查看，执行页面中的查询选项可以看到Console中出现相应的请求，然后在请求中的data中找到相应字段对应的字段名字，shouldSuijie:"是"。
再在All->transports->Headers中找到Request URL: http://std-report.yxapp.in/internal/bd/transports
在std-report中的conf/routes中找到与URL对应的接口，如果查不到相应定时就去user-referral里面去查

    POST        /internal/bd/transports                    controllers.TransportController.allTransports

如果需要寻找更新逻辑则
去mysql-scheduler或者std-report全局搜字段shouldSuijie或suijie或should_suijie，因为在不同系统中同一个字段可能有不同字段名字，如果查不到相应定时就去user-referral里面去查,

例如查找更新信息则查到updateSuijieInfoById，再找到使用其的updateByLoans等三个接口，再找到使用其的方法，最后查到updateSuijieInfo接口

    GET         /updateSuijieInfo                       controllers.HomeController.updateSuijieInfo

再发送带有对应参数的GET请求来更新用户信息，逻辑修改到相应的底层代码去改。

mysql-scheduler:
按sql和程序来更新字段，配置文件yxapp.in.conf
routes的找到对应字段的请求

shangtongdai_rpt.application_infos是mysql-scheduler里面跑出来的进件信息

项目主要使用shangtongdai_rpt数据库中的表

在http://std-report-tool.yxapp.in/tool查询工具中查询对应用户的身份证号或者信息

    select * from shangtongdai.user_authentications where user_name='李侠'

    select * from shangtongdai.users where id=540721

举个例子：姓名为田伟的用户需要查询

    select *
    from shangtongdai.user_authentications
    where user_name='田伟'
查询结果：

    id	user_id	user_name	identification	card_number	source_type	addition_loan	is_deleted	deleted_at	created	updated
    4546	526765	田伟	610422197407190017	6228450226018907664	WEB	0	0	null	2018-10-26 12:04:20.0	2018-10-26 12:04:20.0
    8506	146069	田伟	51013219830125661X	6222024402055205128	WEB	1	0	null	2018-12-15 20:07:20.0	2019-08-30 00:23:45.0
    12195	475021	田伟	522730198601080314	6214835102106232	WEB	1	0	null	2019-01-21 13:03:45.0	2019-08-30 00:33:22.0

多个重名，根据user_id 查询到编号475021的用户是我们要找的田伟

    select *
    from shangtongdai.users
    where id=475021

    id	email	email_confirmed	name	mobile	mobile_account	has_set_pwd	code	tracking	hashed_password	salt	signup_ip	created	updated
    475021	null	1	null	150****1468	150****1468	1	QDCYL674	null	9e286c2c3f2954070f0fd12067f7e7d4b351abc5	1928a1d1be554c8a9fb00faff215c881	null	2018-01-26 11:49:21.0	2018-08-15 10:44:42.0

查询该用户的进件信息
    select id,transport_id,user_id,identification,should_suijie
    from shangtongdai_rpt.application_infos
    where identification =522730198601080314 and user_id=475021

查询结果

    id	transport_id	user_id	identification	should_suijie
    309722805	250658	475021	522730198601080314	0
    344227885	272763	475021	522730198601080314	0
    358453685	282782	475021	522730198601080314	0
    377322417	289761	475021	522730198601080314	0


查询记录中should_suijie为NULL的条数

    select count(id)
        from shangtongdai_rpt.application_infos
        where should_suijie IS NULL

查询对应的记录中有身份证信息的条数：

    select count(id)
        from shangtongdai_rpt.application_infos
        where should_suijie IS NULL and identification IS NOT NULL

进件数据库生产环境在credit_audit,测试环境在creditreview

根据进件id在credit_audit.transports中查询should_suijie为NULL，身份证信息不为空的相应记录的更多信息

    select count(*)
    from shangtongdai_rpt.application_infos
    INNER JOIN credit_audit.transports
    ON shangtongdai_rpt.application_infos.transport_id=credit_audit.transports.id
    where should_suijie IS NULL and identification IS NOT NULL

常用网址：
测试环境网址（暂时无权限）：

http://std-report-old.laincloud.xyz/partner

线上环境（scala）：

http://std-report.yxapp.in/index

parnter界面：

http://std-report.yxapp.in/partner

运营界面：

http://std-report.yxapp.in/stdTask

bd界面：

http://std-report.yxapp.in/bdStat

新的report测试网址（java），对应项目std-query：

http://std-report-fe.laincloud.xyz/#/
常见问题：数据不全

上午10：10前不要重启lain中的std-report-tool-pro
里面有重发接口

final_area,final_bd 今天更新昨天进件，本周更新上周进件，刷新code

定时卡住去查：

    SELECT *
    FROM shangtongdai_rpt.rpt_fetch_times ;
里面有定时最后更新时间
如果查不到相应定时就去user-referral里面去查

# 调试定时任务管理器TaskManager时要注意定时表达式满足条件时才会执行

# 使用lain在测试环境部署
登陆：
http://kael-query-test.laincloud.xyz/auth/login/do?loginName=kai.yang&password=123123
测试环境网址：
http://console.laincloud.xyz/
登陆后选择kael-query-test


先点击详情-》构建，然后在镜像选项中点击部署，进行项目的部署，
部署后选择SHELL，在/lain/logs/kael.log中查看日志

    tail -f

# 配置文件application.yml解读
有必要解读一下SpringBoot项目中关于application.yml配置的一些问题：

首先看kael-starter中的pom.xml



    <profiles>
        <profile>
            <id>prod</id>
            <properties>
                <env>prod</env>
            </properties>
        </profile>
        <profile>
            <id>local</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <env>local</env>
            </properties>
        </profile>
        <profile>
            <id>staging</id>
            <properties>
                <env>staging</env>
            </properties>
        </profile>
    </profiles>

三个环境分别是生产环境，本地环境和测试环境，其中

      <activation>
          <activeByDefault>true</activeByDefault>
      </activation>  

代表使用的是本地环境。

在这个pom.xml中还有：

      <resources>
           <resource>
               <directory>src/main/resources</directory>
               <includes>
                   <include>**/*.*</include> <!-- 此配置不可缺，否则mybatis的Mapper.xml将会丢失 -->
               </includes>
               <filtering>false</filtering>
           </resource>
           <resource>
               <directory>src/main/resources.${env}</directory>
           </resource>
       </resources>


在进行加载时首先加载resources文件夹中的application.yml
其中的内容是：

    spring:
      profiles.active: env

而在resources.local文件夹中的application-env.yml中

      application.mode: local
      spring:
        zipkin:
          base-url: http://std-zipkin.laincloud.xyz/
        datasource:
          url: jdbc:mysql://10.143.248.78:3306/kael_query?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&failOverReadOnly=false
          driver-class-name: com.mysql.jdbc.Driver
          username: root
          password: 123456
      kael:
        datasource:
          druidDataSourceList:
            #  前期通过yml配置，后期有需要可以迁移出去配置
            - name: kael_query
              type: com.alibaba.druid.pool.DruidDataSource
              driver-class-name: com.mysql.jdbc.Driver
              url: jdbc:mysql://10.143.248.78:3306/kael_query?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&failOverReadOnly=false
              username: root
              password: mojiti
              #      filters: stat,wall,log4j,config
              max-active: 20
              initial-size: 10
              max-wait: 1000
              min-idle: 1
              max-idle: 10
              time-between-eviction-runs-millis: 60000
              min-evictable-idle-time-millis: 300000
              validation-query: select 'x'
              test-while-idle: true
              test-on-borrow: false
              test-on-return: false
              pool-prepared-statements: true
              max-open-prepared-statements: 50
              max-pool-prepared-statement-per-connection-size: 20
            - name: shangtongdai
              type: com.alibaba.druid.pool.DruidDataSource
              driver-class-name: com.mysql.jdbc.Driver
              url: jdbc:mysql://10.143.248.78:3306/shangtongdai?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&failOverReadOnly=false
              username: root
              password: mojiti
                #      filters: stat,wall,log4j,config
              max-active: 20
              initial-size: 10
              max-wait: 1000
              min-idle: 1
              max-idle: 10
              time-between-eviction-runs-millis: 60000
              min-evictable-idle-time-millis: 300000
              validation-query: select 'x'
              test-while-idle: true
              test-on-borrow: false
              test-on-return: false
              pool-prepared-statements: true
              max-open-prepared-statements: 50
              max-pool-prepared-statement-per-connection-size: 20
            #  前期通过yml配置，后期有需要可以迁移出去配置
      # Actuator 配置
      management.endpoints.web:
        exposure.include: "*"
        exposure.exclude: env,beans
      eureka:
        client:
          register-with-eureka: true
          fetch-registry: true
          serviceUrl:
            defaultZone: http://std-eureka.laincloud.xyz/eureka/
      swagger:
        enable: true

其中指明了application.mode: local所以在加载的时候也要加载它，而

    spring:
      profiles.active: env

则说明在加载时读取的application-env.yml文件名中可以加上-env


# 多条件模糊分页查询遇到的问题
当一些查询条件为NULL或空字符串“”

    /**
      * 分页查询
      * 分页参数pageNo(页码） ，pageSize(每页查询数目）
      */
     @Override
     public PageInfo<TblJobsEntity> queryTblJobByPage(int pageNo, int pageSize, TblJobsEntity tblJobsEntity,
         Date createTimeBegin, Date createTimeEnd, Date updateTimeBegin, Date updateTimeEnd) {
         //查询
         String tag = tblJobsEntity.getTag();
         String receiver = tblJobsEntity.getReceiver();

         PageHelper.startPage(pageNo, pageSize);
         TblJobsEntityExample tblJobsEntityExample = new TblJobsEntityExample();
         TblJobsEntityExample.Criteria criteria = tblJobsEntityExample.createCriteria();
         //使用StringUtil.isEmpty()而不是==来判断，因为==判断的是地址
         if (!StringUtils.isEmpty(tag)) {
             //模糊匹配的通配符
             tag = "%" + tag + "%";
             criteria.andTagLike(tag);

         }
         //如果存在该属性为NULL或空字符串""的时候，需要进行判断，之前出现的错误是只判断该属性是否为NULL而未判断是否为""空字符串
         if (!StringUtils.isEmpty(receiver)) {
             receiver = "%" + receiver + "%";
             criteria.andReceiverLike(receiver);
         }
         if (createTimeBegin != null && createTimeEnd != null) {

             criteria.andCreateTimeBetween(createTimeBegin, createTimeEnd);
         }
         if (updateTimeBegin != null && updateTimeEnd != null) {
             criteria.andUpdateTimeBetween(updateTimeBegin, updateTimeEnd);
         }

         List<TblJobsEntity> tblJobsEntityList = tblJobsEntityMapper.selectByExample(tblJobsEntityExample);

         //用PageInfo对结果进行包装
         PageInfo<TblJobsEntity> pageInfo = new PageInfo<TblJobsEntity>(tblJobsEntityList);

         return pageInfo;
     }

# 通过日志调试定时任务执行情况
在resources.local中添加logback.xml文件，改动 <logger name="cn.kael.query.inner.dao" level="DEBUG" />中的level

      <?xml version="1.0" encoding="UTF-8"?>
      <configuration>

          <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
              <encoder>
                  <pattern>%d{yyyy.MM.dd HH:mm:ss.SSS}|%X{TRACE_ID}|%thread|%p|%c:%line|%m%n</pattern>
              </encoder>
          </appender>

          <root level="INFO">
              <appender-ref ref="CONSOLE"/>
          </root>

          <!--<logger name="org.springframework.amqp" level="info" additivity="false">-->
          <!--<appender-ref ref="CONSOLE"/>-->
          <!--</logger>-->

          <!--<logger name="org.mybatis" level="DEBUG" additivity="false">-->
          <!--<appender-ref ref="CONSOLE"/>-->
          <!--</logger>-->
          <!--<logger name="org.apache.zookeeper" level="ERROR" additivity="false">-->
          <!--<appender-ref ref="CONSOLE"/>-->
          <!--</logger>-->
          <!--<logger name="java.sql.Connection" level="DEBUG" additivity="false">-->
          <!--<appender-ref ref="CONSOLE"/>-->
          <!--</logger>-->
          <!--<logger name="java.sql.Statement" level="DEBUG" additivity="false">-->
          <!--<appender-ref ref="CONSOLE"/>-->
          <!--</logger>-->
          <!--<logger name="java.sql.PreparedStatement" level="DEBUG" additivity="false">-->
          <!--<appender-ref ref="CONSOLE"/>-->
          <!--</logger>-->
          <logger name="org.springframework.data.redis" level="INFO" additivity="false">
              <appender-ref ref="CONSOLE"/>
          </logger>
          <logger name="redis.clients" level="INFO" additivity="false">
              <appender-ref ref="CONSOLE"/>
          </logger>

          <logger name="cn.kael.query.inner.dao" level="DEBUG" />

      </configuration>

# Mybatis中自动生成主键

## 遇到的问题
在使用例如

    tblJobsEntityMapper.insertSelective(tblJobsEntity);

进行数据插入时，由于数据库的ID为自动插入：

    `id` bigint(20) NOT NULL AUTO_INCREMENT,

所以RequestDTO里往往没有传入id，这时会导致后面的在Quartz中添加定时任务时获取到的id为null

    //在Quartz中添加定时任务
    taskManager.addJob(tblJobsEntity.getId(), tblJobsEntity.getCron(), tblJobsEntity.getMemo());

## 解决的方法
可以在数据库中查询再获取id，但为了减小数据库查询的压力，所以要使用Mybatis的自带配置来进行自动插入id，

在TblJobsEntityMapper.xml中添加配置，在INSERT语句中，我们为可以自动生成（auto-generated）主键的列 id 插入值。

我们可以使用useGeneratedKeys和keyProperty属性让数据库生成auto_increment列的值，并将生成的值设置到其中一个输入对象属性内，如下所示：     


    </insert>
      <insert id="insertSelective" parameterType="cn.kael.query.inner.entity.TblJobsEntity" useGeneratedKeys="true" keyProperty="id">
      ......
    </insert>

 这里id列值将会被数据库自动生成(如mysql)，并且生成的值会被设置到tblJobsEntity对象的id属性上

## Mybatis 查询自动生成的多个数据库中的表
配置文件是config/MybatisConfig
扫描的类是sqlSessionFactory，对应的实现是config/DbAutoConfiguration中的SqlSessionFactoryBean，其数据源配置来自dataSourceProperties

# 参考资料：
【1】https://blog.csdn.net/suwu150/article/details/52895855
