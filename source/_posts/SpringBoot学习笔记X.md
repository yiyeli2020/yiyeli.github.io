---
title: SpringBoot学习笔记X
date: 2019-8-28 17:18:12
categories: 2019年8月
tags: [SpringBoot，Java]

---

SpringBoot系统学习。

<!-- more -->

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



# 参考资料：
【1】
