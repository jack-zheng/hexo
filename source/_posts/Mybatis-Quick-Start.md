---
title: Mybatis 快速上路笔记
date: 2020-09-16 23:08:01
categories:
- 弹射起步
tags:
- java
- mybatis
---

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

解释成白话：这是一个操作数据库的框架，就是把操作简化了，你之前用 JDBC 时的那些配置什么还是少不了只不过用起来更好使罢了。比如使用数据库你得配联接吧，得配驱动把，得写 SQL 把，mybatis 也需要你做这个，只不过人家帮你把这些事情总结出了一个套路，你用这个套路就可以少很多冗余代码，但是也增加了你自己学习这个框架的成本，少了自由度。当然就大部分人的编程水平，肯定是收益大于损失的 ╮(￣▽￣"")╭

* [视频教程](https://www.bilibili.com/video/BV1NE411Q7Nx)
* [练习项目地址](https://github.com/jack-zheng/mybatis-note)
* 练习版本：mybatis 3.5.5

## 原型 JDBC 操作数据库

1. 导入 mysql 包
2. 编写实体类
3. 编写驱动类
4. 编写 Dao 类
5. 测试

```java
@Data
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

```java
public class DBUtils {

    private static final String URL = "jdbc:mysql://localhost:3306/mybatis";
    private static final String NAME = "root";
    private static final String PASSWORD = "12345678";

    private static Connection conn = null;

    static {
        //1.加载驱动程序
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        //2.获得数据库的连接
        try {
            conn = DriverManager.getConnection(URL, NAME, PASSWORD);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    public static Connection getConnection() {
        return conn;
    }

    public static void main(String[] args) throws Exception {
        //3.通过数据库的连接操作数据库，实现增删改查
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("select id, name, pwd from user");//选择import java.sql.ResultSet;
        while (rs.next()) {//如果对象中有数据，就会循环打印出来
            System.out.println("Result: [" + rs.getInt("id") + ", " + rs.getString("name") + ", " + rs.getString("pwd") + "]");
        }
    }
}
```

```java
public class UserDao {

    List<User> getUsers() throws SQLException {
        List<User> users = new ArrayList<>();
        Connection connection = DBUtils.getConnection();
        Statement statement = connection.createStatement();
        ResultSet rs = statement.executeQuery("select id, name, pwd from user");

        while (rs.next()) {
            User tmp = new User();
            tmp.setId(rs.getInt("id"));
            tmp.setName(rs.getString("name"));
            tmp.setPwd(rs.getString("pwd"));
            users.add(tmp);
        }
        return users;
    }
}
```

```java
public class UserDaoTest {
    public static void main(String[] args) throws SQLException {
        UserDao dao = new UserDao();
        List<User> users = dao.getUsers();
        for (User user : users) {
            System.out.println(user);
        }
    }
}
```

mybatis 为我们做的只不过是把上面的这些步骤简化了，通过配置文件管理连接信息，通过 factory， SqlSession 等来管理 SQL 执行等。按照这样的思路去理解记忆应该会更加有效率。

## 搭建环境 mybatis-01-setup

对照官方文档的入门篇

创建测试表

```SQL
-- 创建测试数据库
CREATE DATABASE mybatis;
USE mybatis;

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

### 最简项目树

```txt
.
├── java
│   └── com
│       └── jzheng
│           ├── dao
│           │   ├── UserMapper.java
│           │   └── UserMapper.xml
│           └── pojo
│               └── User.java
└── resources
    └── mybatis-config.xml
```

### 新建测试项目

1. 新建 maven 项目
2. 删除 src 目录，通过 module 的方式管理，条理更清楚
3. 配置依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jzheng</groupId>
    <artifactId>mybatis-note</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>mybatis-01-setup</module>
    </modules>

    <!-- java 8 compiler 配置，和下面的 build plugin 配合使用 -->
    <properties>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
    </properties>

    <!-- mybatis 基础包，包括 DB 驱动，连接，测试的 jar 包 -->
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
        <!-- 在 build 的时候将工程中的配置文件也一并 copy 到编译文件中，即 target 文件夹下 -->
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

</project>
```

配置 idea 链接本地 mysql 报错 `Server returns invalid timezone. Go to 'Advanced' tab and set 'serverTimezone' property manually.`

时区错误，MySQL默认的时区是UTC时区，比北京时间晚8个小时。在mysql的命令模式下，输入 `set global time_zone='+8:00';` 即可

连接后点击扳手图标可以拿到 url 信息

mybatis 核心配置文件，这个文件中配置 DB 连接，驱动等信息，算是 mybatis 的入口配置文件了。

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
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTime=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/jzheng/mapper/UserMapper.xml"/>
    </mappers>
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

生成实体类 pojo

```java
public class User {
    private int id;
    private String name;
    private String pwd;

    // ...
    // 省略构造函数和 getter/setter
}
```

定义 Dao 接口

```java
public interface UserMapper {
    // CURD user
    int addUser(User user);
    int deleteUser(int id);
    int updateUser(User user);
    User getUserById(int id);

    // First sample
    List<User> getUsers();
}
```

配置 Mapper xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jzheng.mapper.UserMapper">
    <insert id="addUser" parameterType="com.jzheng.pojo.User">
        insert into mybatis.user (id, name, pwd) values (#{id}, #{name}, #{pwd})
    </insert>

    <delete id="deleteUser">
        delete from mybatis.user where id=#{id};
    </delete>

    <update id="updateUser" parameterType="com.jzheng.pojo.User">
        update mybatis.user set name=#{name}, pwd=#{pwd} where id=#{id};
    </update>

    <select id="getUserById" resultType="com.jzheng.pojo.User">
        select * from mybatis.user where id=#{id};
    </select>

    <!-- 查询所有用户 -->
    <select id="getUsers" resultType="com.jzheng.pojo.User">
        select * from mybatis.user;
    </select>
</mapper>
```

编写测试类

```java
public class UserMapperTest {
    @Test
    public void test_official_sample() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession session = sqlSessionFactory.openSession();
        UserMapper mapper = session.getMapper(UserMapper.class);
        List<User> users = mapper.getUsers();
        for (User user : users) {
            System.out.println(user);
        }
        session.close();
    }

    @Test
    public void test_util() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        List<User> users = sqlSession.getMapper(UserMapper.class).getUsers();
        for (User user : users) {
            System.out.println(user);
        }
        sqlSession.close();
    }

    @Test
    public void test_add() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        User user = new User(5, "t0928", "pwd");

        int ret = sqlSession.getMapper(UserMapper.class).addUser(user);
        System.out.println(ret);
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void test_delete() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        int ret = sqlSession.getMapper(UserMapper.class).deleteUser(5);
        System.out.println(ret);
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void test_update() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        User user = new User(2, "change", "pwdchange");
        int ret = sqlSession.getMapper(UserMapper.class).updateUser(user);
        System.out.println(ret);
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void test_getUserById() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        User ret = sqlSession.getMapper(UserMapper.class).getUserById(1);
        System.out.println(ret);
        sqlSession.close();
    }
}
```

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

**[Attention]:**

1. 当进行增删改操作时需调用 commit 方法将修改提交才能生效
2. namespace 中的包名要和 Dao/mapper 保持一致

### 万能 map

如果实体类的属性过多，可以考虑使用 map 传递参数, 这是一种可定制性很高的用法

```java
// Mapper interface
User getUserByMap(Map map);
```

```xml
<!-- 通过 map 查询 -->
<select id="getUserByMap" parameterType="map" resultType="com.jzheng.pojo.User">
    select * from mybatis.user where id=#{id};
</select>
```

测试用例

```java
@Test
public void test_getUserByMap() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    Map<String, Object> map = new HashMap<>();
    map.put("id", 1);
    User ret = sqlSession.getMapper(UserMapper.class).getUserByMap(map);
    System.out.println(ret);
    sqlSession.close();
}
```

### 分页功能 limit

通过 map 来实现分页功能

```sql
select * from table limit startIndex, size;
```

```java
// Limit query
List<User> getUsersWithLimit(Map map);
```

```xml
<!-- 分页 -->
<select id="getUsersWithLimit" parameterType="map" resultType="com.jzheng.pojo.User">
    select * from mybatis.user limit #{startIndex}, #{pageSize};
</select>
```

### 常用变量的作用域

**SqlSessionFactoryBuilder:** 一用完就可以丢了，局部变量

**SqlSessionFactory:** 应用起了就要应该存在，所以应用作用域(Application)最合适。而且只需要一份，使用单列或者静态单列模式

**SqlSession:** 线程不安全，不能共享。最佳作用域是请求或方法层。响应结束后，一定要关闭，所以最佳时间是把它放到 finally 代码块中，或者用自动关闭资源的 try block。


### 疑问记录

1. 项目中我即使把 pojo 的构造函数和 getter/setter 都注视掉了，值还是被塞进去了，和 spring 不一样，他是怎么实现的？
2. 核心配置文件中的 mapper setting，resource tag 不支持匹配符？类似 `com/jzheng/mapper/*.xml` 并不能生效
3. mapper.xml 中 resultType 怎么简写，每次都全路径很费事
4. mybatis 中是不支持方法重载的

### 疑问解答

1. mybatis 会通过 DefaultResultSetHandler 处理结果集，applyAutomaticMappings 就是进行映射的地方，这个方法下面会通过反射对 field 进行赋值，并没有调用 set 方法，别和 spring 搞混了。
2. TBD
3. 参见 配置 -> typeAlias

## Lombok 偷懒神器

Lombok 可以省去你很多冗余代码，在测试项目的时候很好用。是否使用看个人，但是就个人小项目来说我还是很愿意使用的。

1. Idea 安装 lombok 插件
2. 安装依赖的 jar 包
3. 在 pojo 类中添加注解使用

调试技巧：在 pojo 上添加注解后，你可以在 idea 的 Structure tab 里看到新生产的方法

## 配置解析 mybatis-02-configuration

对应 配置 章节

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

尽管可以配置多个环境，但每个 SqlSessionFactory 实例**只能选择一种**环境。如果想连接两个数据库就需要创建两个 SqlSessionFactory 实例。

**事务管理器(transactionManager)**有 JDBC 和 MANAGED 两种，默认使用 JDBC，另一种几乎很少用，权作了解。

**数据源(dataSource)**用来配置数据库连接对象的资源，有 [UNPOOLED|POOLED|JNDI] 三种。JNDI 是为了支持 EJB 应用，现在应该已经过时了。

DB Pool 的常见实现方式：jdbc，c3p0, dbcp

### properties 属性

引用配置文件，可以和 `.properties` 文件交互

文件目录如下：

```txt
resources
├── db.properties
└── mybatis-config.xml
```

db.properties

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=false&useUnicode=true&characterEncoding=UTF-8&serverTime=UTC
```

mybatis-config 配置如下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="db.properties">
        <property name="uname" value="root"/>
        <!-- priority rank: parameter > properties file > property tab -->
        <property name="url" value="tmp_url"/>
    </properties>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${uname}"/>
                <property name="password" value="12345678"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/jzheng/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

xml 中的 properties tag + resource 属性可以将配置文件加载进来。另外还有一种属性配置方式是直接在构建 session factory 或者 factory builder 的时候通过参数的形式传入。

```java
sqlSessionFactoryBuilder.build(reader, props);
// ... or ...
new SqlSessionFactoryBuilder.build(reader, environment, props);
```

三种属性添加方式优先级：parameter > properties 文件 > property 标签

### typeAlias 类型别名

设置短的名字，减少类完全限定名的冗余

```xml
<typeAliases>
    <typeAlias type="com.jzheng.pojo.User" alias="User"/>
</typeAliases>

<typeAliases>
    <package name="com.jzheng.pojo"/>
</typeAliases>
```
也可以在实体类上添加 Alias 注解

```java
@Alias("user")
public class User {}
```

三种添加别名的方式 typeAliases+typeAlias, typeAliases+package 和 类名+@Alias。想要使用缩写必须在配置文件中加上 typeAliases 的 tag 直接在类上使用注解是不会生效的。

typeAliases 使用时，是忽略大小写的，官方提倡使用首字母小写的命名方式。一旦类傻上加了注解，则**严格**匹配类注解

### setting 设置

比较常用的设置为：

* cacheEnabled：开启缓存配置
* logImpl：开启日志配置

### mapper 映射器

映射器用来告诉 mybatis 到哪里去找到映射文件

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

{% asset_img SessionFactory_Session.PNG Factory_Session 关系图 %}

每个 Mapper 代表一个具体的业务，比如 UserMapper。

### 解决属性名和字段名字不一样的问题

将 User 的 pwd 改为 password, 和 DB 产生歧义

```java
@Data
public class User {
    private int id;
    private String name;
    private String password;
}
```

解决方案01, 在 Sql 中使用 as 关键字重新指定 column name 为 property name(pwd as password)。

```xml
<select id="getUserById" parameterType="int" resultType="user">
    select id, name, pwd as password from mybatis.user where id = #{id};
</select>
```

解决方案02, 使用 resultMap 映射结果集

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

### 疑问记录

1. 在测试属性和数据库名字不一样的案例的时候发现，就算不一样，但是如果有构造函数的话，还是会被赋值，但是顺序会被强制指定，如果我构造为 User(id,password) 则 User 的 name 会被赋值成 pwd, 应该和底层实现有关系

## 日志 mybatis-03-logging

支持的 log framework 类型

* SLF4J [Y]
* LOG4J 
* LOG4J2 [Y]
* JDK_LOGGING
* COMMONS_LOGGING
* STDOUT_LOGGING [Y]
* NO_LOGGING

STDOUT_LOGGING 是自带的 log 包，直接 enable 就能使用，使能后可以在 log 中看到运行的 SQL。

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

### 开启 log4j 支持

log4j 是一个比较常用的日志框架，有很多功能，比如定制格式，指定存到文件等

1. 导包
2. 添加 log4j.properties
3. 添加配置到核心配置文件

```properties
# 全局日志配置
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/mybatis-03-logging.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

使能配置

```xml
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

## 基于注解开发

基于注解开发，在应对简单的需求时还是很高效的，但是不能处理复杂的 SQL。

面向接口编程：

* 接口定义和实现分离
* 反映出设计人员对系统的抽象理解

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

PS: 注解和 xml 中对同一个接口只能有一种实现，如果重复实现，会抛异常

```bash
Caused by: java.lang.IllegalArgumentException: Mapped Statements collection already contains value for com.jzheng.mapper.UserMapper.getUserById. please check com/jzheng/mapper/UserMapper.xml and com/jzheng/mapper/UserMapper.java (best guess)
```

注解模式的实现**机制**：反射 + 动态代理

注解和配置文件是可以共存的，只要命名相同，并且实现方法没有冲突就行。

### 注解版 CRUD

工具类自动提交事务可以通过 Utils 类中，指定参数实现。注解版的 CRUD 基本上和 xml 版本的一样，只不过在注解版中，他的参数类型通过 @Param 指定。

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

    // 当参数是对象时，直接传入即可，保证属性名一致
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

## 高级结果映射

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

1. 新建表，准备测试数据
2. 新建 teacher/student 实体类
3. 创建 mapper 接口
4. 创建 mapper xml 文件
5. 核心配置类注册接口或 xml
6. 测试查询

### 多对一 mybatis-05-resultmap

在这里采用多个学生对应一个老师的情况作为案例，为了更好的面向对象 Student pojo 需要做一些修改，我们把 teach id 用对象来代替

```java
@Data
public class Student {
    private int id;
    private String name;
    private Teacher teacher;
}
```

我们想要实现的效果其实就是子查询 `SELECT st.id, st.name, te.name as tname from student st, teacher te where st.tid = te.id;`

关键点：使用 **association** tag 作为连接键

#### 按照查询嵌套处理

1. 直接写查询所有学生信息的语句，结果集自定义
2. 根据自定义的结果集，将 teacher 对象和 tid 绑定
3. 定义根据 tid 查询 teacher 的语句
4. Mybatis 会自动将查询到的 teacher 对象整合到学生的查询结果中

```xml
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

#### 按照结果嵌套处理

这种方法的查询更加直接，和上面给出的 SQL 基本一致，就是 association 部分的匹配看着有点懵，大概是 mybatis 底层都会根据 column name 做匹配的，但是这里查询的时候 teacher 的 name 字段重命名为 tname 了所以要显示的重新匹配一下。

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

### 一对多  mybatis-05-resultmap02

一个老师对应多个学生为案例, 代码和思路和上面的多对一其实没什么区别，就是关键字变了一下。。。

关键字：**collection** tag

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
<select id="getTeachers" resultMap="TeacherStudent">
    select * from mybatis.teacher;
</select>
<resultMap id="TeacherStudent" type="Teacher">
    <collection property="students" ofType="Student" column="id" javaType="ArrayList" select="getStudents"/>
</resultMap>

<select id="getStudents" resultType="Student">
    select * from mybatis.student where tid=#{id}
</select>
```

### 小结：

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

## 动态 SQL mybatis-06-dynamic-sql

根据不同的条件生成不同的 SQL 语句

* if
* choose (when, otherwise)
* trim (where, set)
* foreach

### 搭建环境

```sql
CREATE TABLE `blog`(
`id` VARCHAR(50) NOT NULL COMMENT '博客id',
`title` VARCHAR(100) NOT NULL COMMENT '博客标题',
`author` VARCHAR(30) NOT NULL COMMENT '博客作者',
`create_time` DATETIME NOT NULL COMMENT '创建时间',
`views` INT(30) NOT NULL COMMENT '浏览量'
)ENGINE=INNODB DEFAULT CHARSET=utf8
```

1. 导包
2. 编写配置
3. 编写实体类
4. 编写 mapper + 测试

```java
@Data
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date createTime;
    private int views;
}
```

### if

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from mybatis.blog where 1=1
    <if test="title != null">
        and title=#{title}
    </if>
    <if test="author != null">
        and author=#{author}
    </if>
</select>
```

### choose (when, otherwise), 这种判断语句更贴近 java 中的 switch-case，在 if 中，所有符合 test 判断的条件都会被添加进去，但是在 choose 中，他只会从众多条件中选择一种

```xml
<select id="queryBlogChoose" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <choose>
            <when test="title != null">
                title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
```

trim (where, set), where 可以对 xml 中定义的 and + where 冗余情况进行判断，只在需要的时候才添加这个关键字，同理 set 会处理 set + ，的情况

PS: 添加 set 标签的时候 `,` 是一定要加的，多余的 `,` 框架会帮你去掉，少了可不行。

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <if test="title != null">
            and title=#{title}
        </if>
        <if test="author != null">
            and author=#{author}
        </if>    
    </where>
</select>

<update id="updateBlog" parameterType="map">
        update  mybatis.blog
        <set>
            <if test="title != null">
                title=#{title},
            </if>
            <if test="author!=null">
                author = #{author}
            </if>
        </set>
        where id=#{id}
    </update>
```

### foreach 可以用来处理类似 `SELECT * from blog where id in ("1", "2", "3");` 的 SQL

```xml
<select id="queryBlogs" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <foreach collection="ids" item="id" open="and (" close=")" separator="or">
            id=#{id}
        </foreach>
    </where>
</select>
```

所谓的动态 SQL，本质还是 SQL 语句，只是我们可以在 SQL 层面去执行一个逻辑代码

### SQL片段

1. 将公共部分抽取出来
2. 通过 include 标签引用

```xml
<sql id="if-title-author">
    <if test="title != null">
        and title=#{title}
    </if>
    <if test="author != null">
        and author=#{author}
    </if>
</sql>

<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <include refid="if-title-author"></include>
    </where>
</select>
```

* 最好基于单表来定义 SQL 片段
* 不要存在 where 标签

## Cache 缓存 - mybatis-07-cache

在 DB 操作中连接数据库是非常消耗资源的，所以有了缓存机制来减少重复的查询操作消耗

缓存：一次查询的结果，给他暂存在内存中，再次查询的时候直接走取结果

### 一级缓存

一级缓存默认开启，且不能关闭，只在一次 SqlSession 中有用

1. 开启日志
2. 测试一次 session 中查询两次相同结果
3. 查看日志输出

缓存失效的几种情况：

1. 查询不同的东西
2. 增删改可能会改变原来的数据，所以必定要刷新缓存
3. 查询不同的 mapper.xml
4. 手动清理缓存

#### 测试 p1

```java
@Test
public void getUsers() {
    SqlSession session = MybatisUtils.getSqlSession();
    System.out.println("-----> query user1 the first time <-----");
    session.getMapper(UserMapper.class).getUserById(1);
    System.out.println("-----> query user1 the second time <-----");
    session.getMapper(UserMapper.class).getUserById(1);
    System.out.println("-----> query user2 the second time <-----");
    session.getMapper(UserMapper.class).getUserById(2);

    session.close();
}
```

输出 log

```txt
-----> query user1 the first time <-----
Opening JDBC Connection
Created connection 1866875501.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@6f46426d]
==>  Preparing: select * from mybatis.user where id=?; 
==> Parameters: 1(Integer)
<==    Columns: id, name, pwd
<==        Row: 1, jack, 123
<==      Total: 1
-----> query user1 the second time <-----
-----> query user2 the second time <-----
==>  Preparing: select * from mybatis.user where id=?; 
==> Parameters: 2(Integer)
<==    Columns: id, name, pwd
<==        Row: 2, change, pwdchange
<==      Total: 1
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@6f46426d]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@6f46426d]
Returned connection 1866875501 to pool.
```

user1 在第一次 query 的时候有访问 DB，第二次则直接从内存拿，在同一个 session 中访问 user2 也会从 DB 拿

#### 测试 p4

```java
@Test
public void getUsers() {
    SqlSession session = MybatisUtils.getSqlSession();
    System.out.println("-----> query user1 the first time <-----");
    session.getMapper(UserMapper.class).getUserById(1);
    session.clearCache(); // 手动清 cache !!!
    System.out.println("-----> query user1 the second time <-----");
    session.getMapper(UserMapper.class).getUserById(1);
    session.close();
}
```

输出 log

```txt
-----> query user1 the first time <-----
Opening JDBC Connection
Created connection 1936722816.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@73700b80]
==>  Preparing: select * from mybatis.user where id=?; 
==> Parameters: 1(Integer)
<==    Columns: id, name, pwd
<==        Row: 1, jack, 123
<==      Total: 1
-----> query user1 the second time <-----
==>  Preparing: select * from mybatis.user where id=?; 
==> Parameters: 1(Integer)
<==    Columns: id, name, pwd
<==        Row: 1, jack, 123
<==      Total: 1
```
添加了清理 cache 的语句后，第二次访问同一个 user 也会从 DB 拿

### 二级缓存

1. 开启全局缓存 cacheEnabled -> true
2. 在 mapper.xml 中加入 <cache/> 标签

为了支持 <cache/> 标签需要 pojo 类实现序列化接口不然会报错 ` Cause: java.io.NotSerializableException: com.jzheng.pojo.User`

* 一级缓存作用域太低了，所以诞生了二级缓存
* 基于 namespace 级别的缓存，一个命名空间对应一个二级缓存
* 工作机制
  * 一个会话查询一条数据，数据被存放在一级缓存中
  * 当前会话关闭，对应的一级缓存就没了，一级缓存中的数据会被保存到二级缓存中
  * 新会话查询信息，会从二级缓存中获取内容
  * 不同 mapper 查出的数据会放在自己对应的缓存中

#### 测试用例

```java
@Test
public void getUsers_diff_session() {
    SqlSession session1 = MybatisUtils.getSqlSession();
    System.out.println("-----> query user1 the first time <-----");
    session1.getMapper(UserMapper.class).getUserById(1);
    session1.close();

    SqlSession session2 = MybatisUtils.getSqlSession();
    System.out.println("-----> query user1 the second time <-----");
    session2.getMapper(UserMapper.class).getUserById(1);
    session2.close();
}
```

当 mapper 中没有添加 <cache/> 标签时，输出如下，两个 session 查询同一个 user 的时候都进行了 DB 访问

```txt
-----> query user1 the first time <-----
Opening JDBC Connection
Created connection 1936722816.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@73700b80]
==>  Preparing: select * from mybatis.user where id=?; 
==> Parameters: 1(Integer)
<==    Columns: id, name, pwd
<==        Row: 1, jack, 123
<==      Total: 1
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@73700b80]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@73700b80]
Returned connection 1936722816 to pool.
-----> query user1 the second time <-----
Opening JDBC Connection
Checked out connection 1936722816 from pool.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@73700b80]
==>  Preparing: select * from mybatis.user where id=?; 
==> Parameters: 1(Integer)
<==    Columns: id, name, pwd
<==        Row: 1, jack, 123
<==      Total: 1
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@73700b80]
```

当 mapper 中添加 <cache/> 标签时，输出如下，第二次查询 user 时是从 cache 中查找的

```txt
-----> query user1 the first time <-----
Cache Hit Ratio [com.jzheng.mapper.UserMapper]: 0.0
Opening JDBC Connection
Created connection 379645464.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@16a0ee18]
==>  Preparing: select * from mybatis.user where id=?; 
==> Parameters: 1(Integer)
<==    Columns: id, name, pwd
<==        Row: 1, jack, 123
<==      Total: 1
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@16a0ee18]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@16a0ee18]
Returned connection 379645464 to pool.
-----> query user1 the second time <-----
Cache Hit Ratio [com.jzheng.mapper.UserMapper]: 0.5
```

小结：

* 只要开启二级缓存，在同一个 Mapper 下就有效
* 所有的数据都会先放在一级缓存中
* 只有当会话提交或者关闭，才会提交到二级缓存中

## 缓存原理

1. 先看二级缓存中有没有
2. 再看一级缓存中有没有
3. 最后才查DB

## 自定义缓存 ehcache

一种广泛使用的开源 Java 分布式缓存，主要面向通用缓存

使用：

1. 导包
2. config 中配置 type

不过这样的功能现在都用类似 redis 的工具代替了，应该不是主流用法了

