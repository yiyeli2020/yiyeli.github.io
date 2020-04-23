---
title: SpringBoot学习笔记VIII-MybatisPlus教程
date: 2020-4-17 10:12:12
categories: 2020年4月
tags: [SpringBoot]

---

MybatisPlus教程

<!-- more -->

# 简介
官网地址：https://mybatis.plus/

官网简介：MyBatis-Plus 是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。



下面介绍如何快速上手 mybatis-plus，给出了 mapper 和 service CRUD 的基本示例。

建议在后续开发中，时时翻看官方文档（https://mybatis.plus/guide）。

# 快速开始
## 添加依赖

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.3.0</version>
    </dependency>

## MapperScan 配置

    @SpringBootApplication
    @MapperScan("com.xx.xx.xx.mapper")
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
# Mapper 基本使用
## Entity 定义

    @Data
    @Accessors(chain = true)
    @TableName("t_user") // 表名，默认是类名转下划线
    public class User {
        // 指定主键，IdType.AUTO 表示自增主键
        @TableId(type= IdType.AUTO)
        private Long id;
        private String name;
        private Integer age;
        private String email;
        private Integer address;


        // 不在数据库里的字段
        @TableField(exist = false)
        private Integer count;
    }
## Mapper 定义：继承 BaseMapper

    public interface UserMapper extends BaseMapper<User> {

    }
## 测试用例

    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class MapperTest {

        @Autowired
        private UserMapper mapper;

        @Test
        public void insertTest() {
            // 1. insert
            User user = new User();
            user.setName("小羊");
            user.setAge(3);
            user.setEmail("abc@mp.com");
            assertThat(mapper.insert(user)).isGreaterThan(0);
            // 成功直接拿 ID
            assertThat(user.getId()).isNotNull();
        }


        @Test
        public void deleteTest() {
            // 1. deleteById
            assertThat(mapper.deleteById(3L)).isGreaterThan(0);
            // 2. delete with queryWrapper
            assertThat(mapper.delete(new QueryWrapper<User>()
                .lambda().eq(User::getName, "Sandy"))).isGreaterThan(0);
        }


        @Test
        public void updateTest() {
            // 1. updateById
            assertThat(mapper.updateById(new User().setId(1L).setEmail("ab@c.c"))).isGreaterThan(0);

            // 2. update with entity and updateWrapper
            assertThat(
                mapper.update(
                    new User().setName("mp"),
                    Wrappers.<User>lambdaUpdate()
                        .set(User::getAge, 3)
                        .eq(User::getId, 2)
                )
            ).isGreaterThan(0);
            User user = mapper.selectById(2);
            assertThat(user.getAge()).isEqualTo(3);
            assertThat(user.getName()).isEqualTo("mp");

            // 3. update with entity and queryWrapper
            mapper.update(
                new User().setEmail("miemie@baomidou.com"),
                new QueryWrapper<User>()
                    .lambda().eq(User::getId, 2)
            );
            user = mapper.selectById(2);
            assertThat(user.getEmail()).isEqualTo("miemie@baomidou.com");

            // 4. update with UpdateWrapper
            mapper.update(null, new UpdateWrapper<User>()
                .lambda().eq(User::getId, 2).set(User::getAge, 1));
            // mapper.update(null, new UpdateWrapper<User>()
            //   .eq("id", 2).set("age", 1));
            user = mapper.selectById(2);
            assertThat(user.getAge()).isEqualTo(1);
        }


        @Test
        public void selectTest() {
            // 1. selectById
            assertThat(mapper.selectById(1L)).isNotNull();

            // 2. selectOne with QueryWrapper
            User user = mapper.selectOne(new QueryWrapper<User>().lambda().eq(User::getId, 1L));
            assertThat(user.getAge()).isEqualTo(3);

            // 3. selectList with lambdaQuery
            mapper.selectList(Wrappers.<User>lambdaQuery().select(User::getId))
                .forEach(x -> {
                    assertThat(x.getId()).isNotNull();
                });

            // 4. selectList with QueryWrapper
            mapper.selectList(new QueryWrapper<User>().select("id", "name"))
                .forEach(x -> {
                    assertThat(x.getId()).isNotNull();
                });

            // 5. selectMaps
            List<Map<String, Object>> mapList = mapper.selectMaps(Wrappers.<User>query().orderByAsc("age"));
            assertThat(mapList).isNotEmpty();
            assertThat(mapList.get(0)).isNotEmpty();
            System.out.println(mapList.get(0));

            // 6. selectPage
            IPage<User> page = mapper.selectPage(new Page<>(1, 5),
                Wrappers.<User>query().orderByAsc("age"));
            assertThat(page).isNotNull();
            assertThat(page.getRecords()).isNotEmpty();
            System.out.println(page.getRecords().get(0));

            // 7. selectMaxId
            QueryWrapper<User> wrapper = new QueryWrapper<>();
            wrapper.select("max(id) as id");
            User userMaxId = mapper.selectOne(wrapper);
            System.out.println("maxId=" + userMaxId.getId());
            List<User> users = mapper.selectList(Wrappers.<User>lambdaQuery().orderByDesc(User::getId));
            Assert.assertEquals(userMaxId.getId().longValue(), users.get(0).getId().longValue());
        }

        @Test
        public void orderByTest() {
            List<User> users = mapper.selectList(Wrappers.<User>query().orderByAsc("age"));
            assertThat(users).isNotEmpty();

            List<User> usersLambda = mapper.selectList(Wrappers.<User>lambdaQuery().orderByAsc(User::getAge));
            assertThat(usersLambda).isNotEmpty();
        }

        @Test
        public void groupTest() {
            QueryWrapper<User> wrapper = new QueryWrapper<>();
            wrapper.select("age, count(*)")
                .groupBy("age");
            List<Map<String, Object>> maplist = mapper.selectMaps(wrapper);
            for (Map<String, Object> mp : maplist) {
                System.out.println(mp);
            }
            mapper.selectList( new QueryWrapper<User>().lambda()
                .select(User::getAge)
                .groupBy(User::getAge)
                .orderByAsc(User::getAge))
                .forEach(System.out::println);
        }

        // Mybatis-Plus 没有为 join 操作提供增强接口
        // 在 userMapper 中定义函数，与 Mybatis 的使用一致
        // @Select("select a.id as addressId, t.id as userId, t.name as userName, a.address as address "
        //  + "from t_user t\n"
        //  + "join address a on t.address = a.id\n"
        //  + "where t.id = #{userId}")
        // UserAddress selectUserAddress(Long userId);
        @Test
        public void joinTest() {
            UserAddress userAddress = mapper.selectUserAddress(7L);
            System.out.println(userAddress);
        }
        // UserAddress 定义
        // @Data
        // public class UserAddress {
        //   Integer addressId;
        //   Integer userId;
        //   String address;
        //   String userName;
        // }
    }

# Service 基本使用
## Service 定义

    public interface UserService extends IService<User> {

    }


    @Service
    public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    }
## 测试用例

    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class ServiceTest {

        @Autowired
        private UserService userService;

        @Test
        public void saveTest() {
            User user = new User();
            user.setId(10L);
            user.setName("小羊");
            user.setAge(3);
            user.setEmail("abc@mp.com");
            // 1. save: 相当于 mapper.insert
            assertThat(userService.save(user)).isTrue();
            // 2. saveOrUpdate: 先根据 id 进行 select，如果存在则 update，否则 insert
            assertThat(userService.saveOrUpdate(user)).isTrue();
            System.out.println(user);
        }

        @Test
        public void removeTest() {
            User user = new User();
            user.setName("miemie");
            // 1. removeById
            userService.save(user);
            assertTrue(userService.removeById(user.getId()));

            // 2. remove with QueryWrapper
            userService.save(user);
            assertTrue(userService.remove(new QueryWrapper<User>().lambda().eq(User::getId, user.getId())));

            // 3. removeByMap
            userService.save(user);
            Map<String, Object> columnMap = new HashMap<>();
            columnMap.put("id", user.getId().toString());
            assertTrue(userService.removeByMap(columnMap));

            // 4. lambdaUpdate
            userService.save(user);
            assertTrue(userService.lambdaUpdate().eq(User::getId, user.getId()).remove());
        }

        @Test
        public void updateTest(){
            // 1. updateById
            User user = new User();
            user.setId(1L);
            user.setName("mm");
            assertTrue(userService.updateById(user));

            // 2. update with updateWrapper
            UpdateWrapper<User> updateWrapper = new UpdateWrapper<User>().eq("id", 1L).set("name", "mm");
            System.out.println(updateWrapper.getCustomSqlSegment());
            assertTrue(userService.update(updateWrapper));

            // 3. lambdaUpdate
            boolean update = userService.lambdaUpdate().eq(User::getAge, 18).set(User::getAge, 31).update();
            System.out.println(update);
        }

        @Test
        public void getTest(){
            Long userId = 1L;
            // 1. getById
            User u1 = userService.getById(userId);
            assertThat(u1).isNotNull();

            // 2. getOne
            User u2 = userService.getOne(new QueryWrapper<User>().lambda().eq(User::getId, userId));
            assertThat(u2).isNotNull();
            assertSame(u1.getId(), u2.getId());

            // 3. getList
            List<User> list1 = userService.list(new QueryWrapper<User>().lambda().eq(User::getId, userId));
            assertThat(list1.isEmpty()).isFalse();

            // 4. lambdaQuery
            List<User> list = userService.lambdaQuery().eq(User::getId, userId).list();
            list.forEach(System.out::println);
        }
    }



# 参考资料
