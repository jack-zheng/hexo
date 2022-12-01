---
title: Java 数据库基础
date: 2020-10-12 17:31:18
categories:
- DB
tags:
- java
- mybatis
---

## 创建 JDBC 链接

项目的 pom 文件中添加驱动引用

```xml
// https://mvnrepository.com/artifact/mysql/mysql-connector-java
implementation group: 'mysql', name: 'mysql-connector-java', version: '8.0.28'
```

在 JDBC 实现中，我们通过类似如下代码得到连接信息

```java
public class JdbcConnectionTest {
	public static final String URL = "jdbc:mysql://localhost:3306/mybatis";
	public static final String USER = "root";
	public static final String PASSWORD = "123456";

	public static void main(String[] args) throws Exception {
		//1.加载驱动程序
		Class.forName("com.mysql.jdbc.Driver");
		//2. 获得数据库连接
		Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
		//3.操作数据库，实现增删改查
		Statement stmt = conn.createStatement();
		ResultSet rs = stmt.executeQuery("SELECT id, name, pwd FROM user");
		//如果有数据，rs.next()返回true
		while (rs.next()) {
			System.out.println(rs.getInt("id") + ";" + rs.getString("name") + ";" + rs.getInt("pwd"));
		}
	}
}
```

在 mybatis 中，这些信息都是写在核心配置文件 xml 中的。样板如下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?..."/>
                <property name="username" value="root"/>
                <property name="password" value="12345678"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/jzheng/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

加载相关的代码

```java
String resource = "mybatis-config.xml";
// 获取文件流
InputStream inputStream = Resources.getResourceAsStream(resource); 
// 构建工厂
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

想要了解的点：

1. mybatis 是如何解析 xml 的 - 写一篇 Builder pattern 的文章，解析的时候重度使用这种模式
2. 在解析的时候都塞了一些什么东西

