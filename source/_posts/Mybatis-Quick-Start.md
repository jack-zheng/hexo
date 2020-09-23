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