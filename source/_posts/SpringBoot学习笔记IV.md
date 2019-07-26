---
title: SpringBoot学习笔记IV
date: 2019-7-26 18:30:12
categories: 2019年7月
tags: [SpringBoot]

---

阐述Spring，Spring MVC，Spring Boot 三者的原理和区别。学习笔记II中涉及到对@Resource和@Autowired的区别做了解释。

<!-- more -->

    private static SqlSessionFactory sqlSessionFactory;

        public static void main(String[] args) {
            // Mybatis 配置文件
            String resource = "mybatis-config.xml";

            // 得到配置文件流
            InputStream inputStream = null;
            try {
                inputStream = Resources.getResourceAsStream(resource);
            } catch (IOException e) {
                e.printStackTrace();
            }
            System.out.println(inputStream);
            // 创建会话工厂，传入 MyBatis 的配置文件信息
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            System.out.println(sqlSessionFactory);
    //        for (int i = 0;i<10;i++){
    //            insertUser();
    //        }

    //        insertUser();
            // updateUser();
            // deleteUser();
            // selectUserById();
            // selectAllUser();
    //        findAllUser();
    //        findUserByContactName();
            deleteUserById();

        }

        // 新增用戶

        private static void insertUser() {
            // 通过工厂得到 SqlSession
            SqlSession session = sqlSessionFactory.openSession();

            TblMerchantEntityMapper mapper = session.getMapper(TblMerchantEntityMapper.class);
            TblMerchantEntity user = new TblMerchantEntity();
                user.setContactName("Tom");
                user.setMerchantNo(String.valueOf(Math.random()*Long.MAX_VALUE));
                user.setMerchantName("肯德基");
                user.setStatus("1");
            try {
                mapper.insert(user);

                session.commit();
            } catch (Exception e) {
                e.printStackTrace();
                session.rollback();
            }

            // 释放资源
            session.close();
        }
    //    private static void updateUser(){
    //        SqlSession session = sqlSessionFactory.openSession();
    //        TblMerchantEntityMapper mapper = session.getMapper(TblMerchantEntityMapper.class);
    //
    //        TblMerchantEntityExample tblMerchantEntityExample = new TblMerchantEntityExample();
    //        Criteria criteria = tblMerchantEntityExample.createCriteria();
    //
    //
    //    }
        private static void deleteUserById(){
            SqlSession session = sqlSessionFactory.openSession();
            TblMerchantEntityMapper mapper = session.getMapper(TblMerchantEntityMapper.class);
            int x = mapper.deleteByPrimaryKey((long)30);
            System.out.println("这次删除了"+x+"行数据");
            session.commit();
            session.close();

        }
        private static void findAllUser(){
            SqlSession session = sqlSessionFactory.openSession();
            TblMerchantEntityMapper mapper = session.getMapper(TblMerchantEntityMapper.class);
            TblMerchantEntity entity;
            TblMerchantEntityExample tblMerchantEntityExample = new TblMerchantEntityExample();
            Criteria criteria = tblMerchantEntityExample.createCriteria();
            criteria.getCriteria();
            List<TblMerchantEntity> list = mapper.selectByExample(tblMerchantEntityExample);
            Iterator it = list.iterator();
            while (it.hasNext()){
                entity = (TblMerchantEntity)it.next();
                System.out.println(entity.toString());
            }
            session.close();
        }
        private static void findUserByContactName(){
            SqlSession session = sqlSessionFactory.openSession();
            TblMerchantEntityMapper mapper = session.getMapper(TblMerchantEntityMapper.class);
            TblMerchantEntity entity;
            TblMerchantEntityExample tblMerchantEntityExample = new TblMerchantEntityExample();
            Criteria criteria = tblMerchantEntityExample.createCriteria();
    //        criteria.andContactNameEqualTo("李四").andIdGreaterThan((long)5);
            criteria.andIdBetween((long)2,(long)7);//分页查询
            List<TblMerchantEntity> list = mapper.selectByExample(tblMerchantEntityExample);
            Iterator it = list.iterator();
            while (it.hasNext()){
                entity = (TblMerchantEntity)it.next();
                System.out.println(entity.toString());
            }
        }
