---
title: SpringBoot学习笔记VIII
date: 2019-8-5 11:22:12
categories: 2019年8月
tags: [SpringBoot，Java]

---

CMS系统学习。

<!-- more -->
# 任务
先阅读cn.creditease.bdp.newcms.notice源码，定时相关，其中impl/CmsTaskNoticeHelperImpl/sendMsgByBatch函数是模块入口

## 阅读
cn.creditease.bdp.newcms.cmswrapper.controller.PaymentScheduleController的源码，涉及到customer表，里面是借贷用户的信息

## 阅读
cn.creditease.bdp.newcms.controller.creditreview.TransportController中的public Object create(@RequestBody(required = false) Transport transport)方法重点看FULL_FLOW，cn.creditease.bdp.newcms.service.creditreview.fullFlowfullFlow中的评级部分代码

    CMSResponseCode cmsResponseCode = rateBoth(transport);

和最后一段代码

     dataPrepareCheckAndCheckSuijieRule(transport);

## 需求：

cn.creditease.bdp.newcms.cmswrapper.controller.TransportQueryController中的getAll对应http://std-cms-fe.laincloud.xyz/#/incoming进件查询中的查询按钮，现在需要在标记一栏的返回值中加入新的返回信息。

cn.creditease.bdp.newcms.cmswrapper.service.cms.TransportQueryServiceTest中写测试样例


参见http://wiki.yxapp.in/pages/viewpage.action?pageId=68764493

/api/2.0/application/isDataloan/{transportId} 根据进件号查询是否为数据贷进件
在shangtongdai中site.conf.routes
controllers.api2.InternalApiController.isDataloan

/internal/api/v1/submitDebt 提交负债信息
在shangtongdai中site.conf.api2.routes
controllers.api.InternalApiController.submitDebt

先写submitDebt接口，即siteHelper.submitDebtInfo部分把商通贷中的逻辑写在cms中，scala-》Java

系统调用：Controller->Service->DAO->database
所以逻辑写在Service中，数据库操作写在DAO，一个方法做一件事，增强可读性。he

两个接口原先在shangtongdai中，用scala写成

现在需要迁移到cms中重构为Java


# 系统介绍

最早的site系统，scala语言，编译运行比较慢，一小时左右，包括了application，postloan等所有系统

测试官网网址：http://shangtongdai.yxapp.xyz/newsite/?code=YANGKAI34#/loginReg/login

查询语句：

    select * from shangtongdai.users
        where id in (
          select user_id from shangtongdai.applications
              where id in (
                    select application_id from shangtongdai.loans
                          where loan_status_id=12 ))

application系统：进件系统

巨星系统：数据中转

爬虫系统：爬取商户数据

miner系统：评级系统

postloan系统（贷后系统）

User系统：管理用户

User-refer系统：返还佣金

Report系统：做报表

Report-tool系统：基础查询

综合信贷系统：管理合同签约

新核心系统：管理还款计划表

结算系统：管理还款逻辑

添加店户会触发爬虫系统，爬取商户的数据，使用miner系统进行评级，巨星系统进行数据中转筛除。

评级通过之后，进件将由CMS系统进行审核，审核通过后，所有审核逻辑都在CMS系统中。

审核通过后，由postloan系统（贷后系统）处理，包括了：

1.合同管理：签约，管理状态,综合信贷系统管理合同签约

2.还款：还款计划表,新核心系统管理还款计划表

3.放款：结算系统管理还款逻辑


团队主要负责CMS，postloan和report，report-tool等系统。

application进件系统，爬虫系统，巨星系统，miner评级系统由进件团队负责。

CMS系统里有初审，复审，分发等36个状态。

***shangtongdai数据库表信息：***

transport表：进件表，维护进件状态，

transportlabel拒绝或接受打下标签

transportext 进件附加信息，以json格式存储（最重要）

transport_decisions:重大决策信息放款或其它

transport_assigned_history：客服分单给客户历史的信息



loan :合同

repayment：业务订单，用户还款会生成业务订单

loanrepayment：合同和业务订单的关系，一笔合同会有多笔订单

repayment_type字段，还款类型

repayment_order字段：

repaymenttags:业务订单信息

权限信息的相关表
privilege
role
role_privilege
user_role

任务相关的表
tb_cron_ini
cms_task

# CMS系统分析
## 进件流程
入口create接口
## 状态机流程
很多外界模块都会调用 NewtransportOpmachineServiceimpl/operateTransport
举例审核操作信审3.0系统 ，审核进件，批钱，批产品，校验身份，审核是否符合规定，可能也需要用户补充信息，例如电话核实信息，如果拒绝则调用信审系统，调用cms系统中cn.creditease.bdp.newcms.controller.creditreview.operate接口

目的是变换当前状态到下一个状态，其次是通知操作,label标定，记录备注都会调用状态机，状态不会改变。

## 展示流程
cms有页面，会串联很多系统有交互，会处理很多中间的进件流程。
比如applicatoin进件系统，当application将进件推给cms时，进件信息里很多存在transport_exts表中，CmsDetailController
chrome 右键检查可以看见请求，根据请求找代码

进件流程 ：画流程图
重点是create和fullflow两个


进件分为store检查是否有新进件，没有的话insert，待初审分配，
第二是重新进件


cn.creditease.bdp.newcms.service.creditreview.fullFlow
1.rateBoth full预估和partial预估
2.dataPrepareCheckAndCheckSuijieRule 随借 检查数据完整性是否能够推送到信审3.0，如果不够完整则创建redis任务，



# 参考资料
【1】
https://blog.csdn.net/weixin_34112900/article/details/93630203
