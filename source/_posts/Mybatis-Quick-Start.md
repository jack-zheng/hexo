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