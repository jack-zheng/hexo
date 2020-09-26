---
title: Mybatis 快速上路笔记
date: 2020-09-16 23:08:01
categories:
- 编程
tags:
- java
- mybatis
---

* [视频教程](https://www.bilibili.com/video/BV1NE411Q7Nx)

## 搭建环境

```SQL
-- 创建测试表
CREATE TABLE user (
id INT(20) NOT NULL PRIMARY KEY,
name VARCHAR(30) DEFAULT NULL,
pwd VARCHAR(30) DEFAULT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8;

-- 插入数据
INSERT INTO user (id, name, pwd) VALUES 
(1, 'jack', '123'), (2, 'jack02', '123');
```

新建项目

1. 新建 maven 项目
2. 删除 src 目录
3. 配置依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 父工程 -->
    <groupId>com.jzheng</groupId>
    <artifactId>mybatis-study</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>mybatis-01</module>
    </modules>

    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/junit/junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

</project>
```

配置 idea 链接本地 mysql 报错 `Server returns invalid timezone. Go to 'Advanced' tab and set 'serverTimezone' property manually.`

时区错误，MySQL默认的时区是UTC时区，比北京时间晚8个小时。在mysql的命令模式下，输入 `set global time_zone='+8:00';` 即可

连接后点击扳手图标可以拿到 url 信息

mybatis 配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 核心配置文件 -->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTime=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

编写工具类

```java
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        String resource = "mybatis-config.xml";
        InputStream inputStream = null;
        try {
            inputStream = Resources.getResourceAsStream(resource);
        } catch (IOException e) {
            e.printStackTrace();
        }
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }

}
```

生成实体类 Pojo

定义 Dao 接口

配置 Mapper xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 绑定 dao/mapper 接口 -->
<mapper namespace="com.jzheng.dao.UserDao">
    <select id="getUserList" resultType="com.jzheng.pojo.User">
        select * from mybaties.user;
    </select>
</mapper>
```

编写测试类

常见错误

```bash
org.apache.ibatis.binding.BindingException: Type interface com.jzheng.dao.UserDao is not known to the MapperRegistry.

-- 核心配置文件没有配置 mapper 路径

Caused by: java.io.IOException: Could not find resource com/jzheng/dao/UserMapper.xml
	at org.apache.ibatis.io.Resources.getResourceAsStream(Resources.java:114)
	at org.apache.ibatis.io.Resources.getResourceAsStream(Resources.java:100)
	at org.apache.ibatis.builder.xml.XMLConfigBuilder.mapperElement(XMLConfigBuilder.java:372)
	at org.apache.ibatis.builder.xml.XMLConfigBuilder.parseConfiguration(XMLConfigBuilder.java:119)
	... 27 more

-- maven 约定大于配置，默认指挥将 resources 下面的 xml 导出到 target, 如果需要将 java 下的配置文件到处需要再 pom.xml 下的 build tag 里加点配置

<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>

java.security.cert.CertPathValidatorException: Path does not chain with any of the trust anchors

-- 链接配置问题，可以把 useSSL 改为 false

```

## CRUD

### mapper

namespace 中的包名要和 Dao/mapper 保持一致

### select/insert/update/delete

选择，查询语句

* id: 对应 namespace 中的方法名
* resultType: sql 执行的返回值
* parameterType: 参数类型

1. 编写接口
2. 配置 mapper
3. 测试

怎删改需要提交事务

```code
int addUser(User user);

<insert id="addUser" parameterType="com.jzheng.pojo.User">
    insert into mybatis.user (id, name , pwd) values (#{id}, #{name}, #{pwd});
</insert>

@Test
public void addUser() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    // 方式一：getMapper
    UserMapper userDao = sqlSession.getMapper(UserMapper.class);
    System.out.println(userDao.addUser(new User(4, "haha", "123123")));

    sqlSession.commit();
    sqlSession.close();
}
```

## 万能 map

如果实体类的属性过多，可以考虑使用 map 传递参数

## 模糊查询

## 配置解析

核心配置文件：mybatis-config.xml

```xml
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

### environments 环境变量

可以配置多套环境，但使用时只能选择一种

默认事务管理器 JDBC，默认 dataSource - Pooled

### properties 属性

引用配置文件，可以和 `.properties` 文件交互

db.properties

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=false&useUnicode=true&characterEncoding=UTF-8&serverTime=UTC
username=root
password=root
```

也可以和 xml 混合使用，如果属性有重名，优先使用外部 properties 中的属性

### Alias 别名

设置短的名字，减少类完全限定名的冗余

```xml
<!-- 给实体类起别名 -->
<typeAliases>
    <typeAlias type="com.jzheng.pojo.User" alias="User"/>
</typeAliases>

<!-- 或者包别名，使用时直接用 bean name, 推荐首字母小写 -->
<typeAliases>
    <package name="com.jzheng.pojo"/>
</typeAliases>
```

也可以在实体类上添加 Alias 注解

### setting 设置

cacheEnabled, lazyLoadingEnabled, logImpl

## mapper 映射器

方式一：资源文件

```xml
<mappers>
        <mapper resource="com/jzheng/dao/UserMapper.xml"/>
</mappers>
```

方式二：使用 class 绑定

```xml
<mappers>
    <mapper class="com.jzheng.dao.UserMapper"/>
</mappers>
```
限制：

1. 接口和 mapper 必须重名
2. 接口和 mapper 必须要同意路径下

方式三：包扫描

```xml
<mappers>
    <package name="com.jzheng.dao"/>
</mappers>
```

缺陷也是要在同一路径下

## 生命周期

生命周期和作用域是至关重要的，因为错误的使用会导致非常严重的并发问题

**SqlSessionFactoryBuilder**

* 一旦创建了 SqlSessionFactory 就不在需要他了
* 局部变量

**SqlSessionFactory**

* 说白了就是可以看作数据库连接池
* 一旦创建就一直存在，没有理由丢弃它或者重新创建一个新的
* 因此 SqlSessionFactory 最佳作用域应为 应用作用域
* 最简单的是使用单例模式或静态单例模式

**SqlSession**

* 链接到连接池的一个请求
* 不能被共享
* 最佳作用域是请求或方法作用域
* 用完之后需要赶紧关闭，否则资源被占用

![Factory_Session关系图](SessionFactory_Session.PNG)

每个 Mapper 代表一个具体的业务

## 解决属性名和字段名字不一样的问题

将 User 的 pwd 改为 password, 和 DB 产生歧义

解决方案

1. 起别名

```xml
<select id="getUserById" parameterType="int" resultType="user">
    select id,name,pwd from mybatis.user where id = #{id};
</select>
```

2. resultMap, 结果集映射

```xml
<resultMap id="UserMap" type="User">
    <!-- column: db 字段， property: 实体类属性 -->
    <result column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="pwd" property="password"/>
</resultMap>
<select id="getUserById" parameterType="int" resultMap="UserMap">
    select * from mybatis.user where id = #{id};
</select>
```

ResultMap 的设计思想是，对于简单的语句根本不需要配置显示的结果集映射，对于复杂的语句只需要描述他们的关系就行了。

上面的方案还可以将 id, name 的描述简化掉，框架会帮你处理，只保留不一致的即可

## 日志

### 日志工厂 logImpl

数据库操作异常排错

* SLF4J [Y]
* LOG4J 
* LOG4J2 [Y]
* JDK_LOGGING
* COMMONS_LOGGING
* STDOUT_LOGGING [Y]
* NO_LOGGING

STDOUT_LOGGING sample:

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

```txt
Opening JDBC Connection
Created connection 477376212.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@1c742ed4]
==>  Preparing: select * from mybatis.user where id = ?; 
==> Parameters: 1(Integer)
<==    Columns: id, name, pwd
<==        Row: 1, jack, 123
<==      Total: 1
User{id=1, name='jack', password='123'}
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@1c742ed4]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@1c742ed4]
Returned connection 477376212 to pool.
```

### Log4j

1. 导包
2. 添加 log4j.properties
3. 添加配置到核心配置文件

## 分页

减少数据的处理量

### 使用 limit 分页

```sql
select * from table limit startIndex, size;
```

### RowBounds

稍作了解

## 注解开发

面向接口编程：

* 接口定义和实现分离
* 反应设计人员对系统的抽象理解

接口有两类：一类是对一个个体的抽象，可以对应为一个抽象个体，另一类是对一个个体的某一方面抽象，即形成一个抽象面

个体可能有多个抽象面，抽象提与抽象面是有区别的

1. 在接口方法上添加注解
2. 在核心配置文件中添加配置

```java
public interface UserMapper {
    @Select("select * from user")
    List<User> getUsers();
}
```

```xml
<mappers>
    <mapper class="com.jzheng.dao.UserMapper"/>
</mappers>
```

反射 + 动态代理

## Mybatis 执行流程解析

1. Resources 获取加载全局配置文件
2. 实例化 SqlSessionFactoryBuilder 构造器
3. 解析配置文件流 XMLConfigBulder
4. Configuration 所有的配置信息
5. SqlSessionFactory 实例化
6. Transaction 事务管理器
7. 创建 executor 执行器
8. 创建 SQLSession
9. 实现 CRUD
10. 查看是否成功

## 注解 CRUD

工具类自动提交事务可以通过 Utils 类中，指定参数实现

```java
public static SqlSession getSqlSession() {
    return sqlSessionFactory.openSession(true);
}
```

实现

```java
public interface UserMapper {

    @Select("select * from user")
    List<User> getUsers();

    // 方法存在多个参数，所有参数前面必须加上 @Param
    @Select("select * from user where id=#{id}")
    User getUserById(@Param("id") int id);

    @Insert("insert into user (id, name, pwd) values (#{id}, #{name}, #{password})")
    int addUser(User user);

    @Update("update user set name=#{name}, pwd=#{password} where id=#{id}")
    int updateUser(User user);

    @Delete("delete from user where id=#{id}")
    int deleteUser(@Param("id") int id);
}
```

关于 @Param 注解

* 基本类型 + String 类型需要加
* 引用类型不需要
* 如果只有一个基本类型，可以不加，但还是建议加上
* Sql 中引用的属性名和 Param 中的名字保持一致

'#' 前缀可以防注入，'$' 不行

## Lombok

感觉可以起飞，稍微有点缺点，自行斟酌

1. 安装 Idea 插件
2. 导入 jar 包
3. 实体类加注解

支持的方法

* @Getter and @Setter
* @FieldNameConstants
* @ToString
* @EqualsAndHashCode
* @AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
* @Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
* @Data - 无参构造，getter/settter, toString, equals
* @Builder
* @SuperBuilder
* @Singular
* @Delegate
* @Value
* @Accessors
* @Wither
* @With
* @SneakyThrows
* @val
* @var
* experimental @var
* @UtilityClass
* Lombok config system
* Code inspections
* Refactoring actions (lombok and delombok)

## 多对一

多对一 - 关联 - association

一对多 - 集合 - collection

创建测试表

```sql
CREATE TABLE `teacher` (
                           `id` INT(10) NOT NULL,
                           `name` VARCHAR(30) DEFAULT NULL,
                           PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO teacher(`id`, `name`) VALUES (1, '秦老师');

CREATE TABLE `student` (
                           `id` INT(10) NOT NULL,
                           `name` VARCHAR(30) DEFAULT NULL,
                           `tid` INT(10) DEFAULT NULL,
                           PRIMARY KEY (`id`),
                           KEY `fktid` (`tid`),
                           CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('1', '小明', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('2', '小红', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('3', '小张', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('4', '小李', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('5', '小王', '1');
```

测试环境搭建

1. 导入 lombok
2. 新建 teacher/student 实体类
3. 创建 mapper 接口
4. 创建 mapper xml 文件
5. 核心配置类注册接口或 xml
6. 测试查询

按照查询嵌套处理

```xml
<!-- 
    1. 查询所有学生信息
    2. 根据查询出来的tid 属性寻找对应的老师
    效果上类似子查询
 -->
<select id="getStudent" resultMap="StudentTeacher">
    select * from student;
</select>
<resultMap id="StudentTeacher" type="Student">
    <!-- obj use association, collection use collection -->
    <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
</resultMap>

<select id="getTeacher" resultType="Teacher">
    select * from teacher where id=#{id}
</select>
```

按照结果嵌套处理

```xml
<select id="getStudent2" resultMap="StudentTeacher2">
    select s.id sid, s.name sname, t.name tname from student s, teacher t
    where s.tid = tid;
</select>

<resultMap id="StudentTeacher2" type="Student">
    <result property="id" column="sid"/>
    <result property="name" column="sname"/>
    <association property="teacher" javaType="Teacher">
            <result property="name" column="tname"/>
    </association>
</resultMap>
```

对应 SQL 的子查询和联表查询

## 一对多

一个老师对应多个学生

实体类

```java
@Data
public class Teacher {
    private int id;
    private String name;

    private List<Student> students;
}

@Data
public class Student {
    private int id;
    private String name;
    private int tid;
}
```

按照结果嵌套处理

```xml
<select id="getTeachers" resultType="Teacher">
    select * from teacher;
</select>

<select id="getTeacher" resultMap="TeacherStudent">
    select s.id sid, s.name sname, t.name tname, t.id tid from student s, teacher t
    where s.tid = t.id and t.id=#{tid};
</select>

<resultMap id="TeacherStudent" type="Teacher">
    <result property="id" column="tid"/>
    <result property="name" column="tname"/>
    <collection property="students" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="tid"/>
    </collection>
</resultMap>
```

按照查询嵌套处理

```xml
<select id="getTeacher2" resultMap="TeacherStudent2">
    select * from mybatis.teacher where id=#{tid};
</select>
<resultMap id="TeacherStudent2" type="Teacher">
    <collection property="students" javaType="ArrayList" ofType="Student" select="getStudentByTeacherId" column="id"/>
</resultMap>

<select id="getStudentByTeacherId" resultType="Student">
    select * from mybatis.student where tid = #{tid};
</select>
```

小结：

* 关联 - 一对多 - associate
* 集合 - 多对一 - collection
* javaType & ofType
  * javaType 指定实体类中的属性
  * ofType 指定映射到集合中的 pojo 类型，泛型中的约束类型

注意点：

* 保证SQL可读性，尽量通俗易懂
* 注意一对多和多对一属性名和字段的问题
* 排错时善用 log

面试高频

* Mysql 引擎
* InnoDB 底层原理
* 索引
* 索引优化