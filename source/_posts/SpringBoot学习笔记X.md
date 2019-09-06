---
title: SpringBoot学习笔记X
date: 2019-8-28 17:18:12
categories: 2019年8月
tags: [SpringBoot，Java]

---

SpringBoot系统学习。

<!-- more -->
# mysql-scheduler
四个项目：
mysql-scheduler
user-referral
std-report
std-query


如果字段有问题例如http://std-report.yxapp.in/bdStat页面中某客户的“能否进件”字段出现错误，

先在Chrome中F12（检查）-》Network->Doc查看，执行页面中的查询选项可以看到Console中出现相应的请求，然后在请求中的data中找到相应字段对应的字段名字，shouldSuijie:"是"

去mysql-scheduler全局搜字段shouldSuijie，根据std-report计算的逻辑找到相应字段，如果查不到相应定时就去user-referral里面去查

mysql-scheduler:
按sql和程序来更新字段，配置文件yxapp.in.conf
routes的找到对应字段的请求

shangtongdai_rpt.application_infos是mysql-scheduler里面跑出来的进件信息

项目主要使用shangtongdai_rpt数据库中的表

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


# 参考资料：
【1】https://blog.csdn.net/suwu150/article/details/52895855
