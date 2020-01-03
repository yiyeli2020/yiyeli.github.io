---
title: SpringBoot学习笔记IV
date: 2019-7-26 18:30:12
categories: 2019年7月
tags: [SpringBoot]

---

简单总结Java工程师的学习路径，之后还会详细介绍，总结了最近用到的一些知识和注意事项如定时组件，注释，日志，注解等。

<!-- more -->

#  中级Java工程师（十年学习计划）
JVM调优，class如何编译生成
分布式事务
分布式组件/工具zoomkeeper
缓存的使用（redis）
分库分表
微服务，看SpringCloud源码
并发编程
Spring源码（建议看两到三遍）

# 最近半年学习方向（两个方向）：
1.Spring源码，例如bean初始化
2.AQS

## 定时组件学习
## 注释写法
从Controller到DTO再到Service，每个类中的函数都要有注释，包括功能，参数，返回值，格式如下：
Service中的例子：

    /**
     * 创建商户
     * @param tblMerchantEntity
     * @return
     */

DTO中的例子：

    /**
    * @Description: 商户dto
    * @Author: yiye.li
    * @Date: 2019-07-29 10:45
    */

***注释写法，可以先按照步骤写出注释，然后再相应等填充代码***

## 日志
（日志级别分为info，debug，warning，error等），可用于debug。
例程：

    @Override
    public MerchantResponseDTO MybatisSelectByMerchantNo(TblMerchantEntity RequesttblMerchantEntity){
        MerchantResponseDTO merchantResponseDTO= null;
        try {
            merchantResponseDTO = new MerchantResponseDTO();
            if(logger.isDebugEnabled()){
                //debug 级别的日志
                logger.debug("");
            }
            logger.info("query merchant info by merchantN={}",RequesttblMerchantEntity);
            //1。构建商户实体
            TblMerchantEntity tblMerchantEntity=new TblMerchantEntity();
            TblMerchantEntityExample tblMerchantEntityExample=new TblMerchantEntityExample();
            TblMerchantEntityExample.Criteria criteria=tblMerchantEntityExample.createCriteria();
            criteria.andMerchantNoEqualTo(RequesttblMerchantEntity.getMerchantNo());

            // 2。从数据库查询商户信息
            List<TblMerchantEntity> tblMerchantEntities = tblMerchantEntityMapper.selectByExample(tblMerchantEntityExample);
            // 判断如果查询的商户信息为空则抛出异常
            if(CollectionUtils.isEmpty(tblMerchantEntities)){
                throw new RuntimeException("merchant is null");
            }else{
                //如果查询的商户有重复则在日志中写入warn
                if(tblMerchantEntities.size()>1) {
                    logger.warn("query merchant size ={}", tblMerchantEntities.size());
                }
                //获取第一个商户的信息
                tblMerchantEntity=tblMerchantEntities.get(0);
            }

            //3。将entity中的属性set到merchantResponseDTO中去
            merchantResponseDTO.setID(tblMerchantEntity.getId());
            merchantResponseDTO.setCreateTime(tblMerchantEntity.getCreateTime());
            merchantResponseDTO.setLastUpdateTime(tblMerchantEntity.getLastUpdateTime());
            merchantResponseDTO.setMerchantNo(tblMerchantEntity.getMerchantNo());
            merchantResponseDTO.setMerchantName(tblMerchantEntity.getMerchantName());
            merchantResponseDTO.setContactName(tblMerchantEntity.getContactName());
            merchantResponseDTO.setContactPhone(tblMerchantEntity.getContactPhone());
            merchantResponseDTO.setPlatformKey(tblMerchantEntity.getPlatformKey());
            merchantResponseDTO.setStatus(tblMerchantEntity.getStatus());
            merchantResponseDTO.setShortName(tblMerchantEntity.getShortName());
            merchantResponseDTO.setCheckGroup(tblMerchantEntity.getCheckGroup());
            System.out.println("*********Select by MerchantNo="+tblMerchantEntity.getMerchantNo()+"************");
            System.out.println(JSON.toJSONString(tblMerchantEntity));
        } catch (RuntimeException e) {
            logger.error("system error ",e);
            throw e;
        }

        return merchantResponseDTO;
    }

日志技巧.var

## 实现SpringBoot热部署
