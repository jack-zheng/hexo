---
title: SQL store procedure and function
date: 2021-11-12 18:49:15
categories:
- SQL
tags:
- store procedure
---

看了视屏，过了一个周末，只剩一个大概的映像，其他都忘了，果然还是要实际操作一下印象才会深刻。mysql 使用 docker 版本的，参考 mysql 安装那篇教程

## 数据准别

视屏下方留言区有对应的 SQL 文件可以下载，按照上面的教程，安装完 docker 版本的 mysql 之后，默认会在 tmp 文件夹下创建共享文件。将 sql 文件复制到共享文件中，然后运行一下命令倒入数据

```bash
docker exec -it mysql-test /bin/bash
# 登陆 docker 中的 mysql 终端
mysql -h localhost -P 2999 -u root -p
# 导入数据
source /etc/mysql/conf.d/employees.sql
source /etc/mysql/conf.d/girls.sql
```

## 变量声明

* 系统变量
  * 全局变量 - 系统级别的改动，跨会话有效，重启重置
  * 回话变量 - 新建回话(链接)会重置
* 自定义变量
  * 用户变量
  * 局部变量

系统变量

```sql
-- 查看所有的系统变量
SHOW GLOBAL | [SESSION] VARIABLES;

-- 查看满足条件的部分系统变量
show global | [session] variables like '%char%';

-- 查看置顶的某个系统变量的值
select @@global | [session].系统变量名;

-- 赋值
set global | [session] 系统变量名 = 值;
set @@global | [session] .系统变量名 = 值;
```

自定义变量

```sql
-- 使用方式：声明，赋值，使用
-- 用户变量：
    -- 作用域： 针对于当前会话有效，同会话变量作用域。可以放在任何地方，begin/end 内部或外部

-- 声明并初始化
SET @VARIABLE_NAME=VALUE;
SET @VARIABLE_NAME:=VALUE;
SELECT @VARIABLE_NAME:=VALUE;

-- 赋值
    -- 方式一，同声明
    -- 方式二， select into, 要求必须是**一个**值，不能是一组
select 字段 into 变量名 from 表;

-- 使用(查看用户变量值)
select @VARIABLE_NAME;

-- e.g.
set @count=1;
select count(*) into @count from employees;
SELECT @count;

-- 局部变量，作用域：仅仅在定义他的 begin/end 内部, 并且必须为第一句话

-- 声明
declear 变量名 类型;
declear 变量名 类型 default 值;
-- 赋值
    -- 方式一
    set 局部变量名=值;
    set 局部变量名:=值;
    select @局部变量名:=值;
    -- 方式二：select into
    select 字段 into 局部变量 from 表;

-- 使用
select 局部变量名;
```

| 类型     | 作用域      | 定义和使用位置                 | 语法                                      |
| :------- | :---------- | :----------------------------- | :---------------------------------------- |
| 用户变量 | 当前回话    | 会话中的任意位置               | 必须加@，不限定类型                       |
| 局部变量 | begin/end中 | 必须在 begin/end中，且为第一句 | 一般不用@，除了 select 语句，需要限定类型 |

## Store Procedure

存储过程优点：

* 代码重用
* 简化操作
* 预编译，减少联接次数

```sql
CREATE PROCEDURE 存储过程(参数列表)
BEGIN
    存储过程体(一组合法的 SQL 语句)
END
```

参数列表包含三部分：参数模式，参数名，参数类型, e.g. IN stuname VARCHAR(20)

参数模式：

* IN: 该参数作为输入
* OUT: 该参数作为输出
* INOUT: 该参数即作为输入又可以作为输出

存储过程体如果仅含有一句话，BEGIN END 可以省略

存储过程体中的每条 SQL 语句的结尾要求必须加分好

存储过程的结尾可以使用 DELIMITER 重新设置，`DELIMITER 结束标志`: e.g. DELIMITER $

调用：CALL 存储过程名(实参列表);

```sql
-- 1. 空参列表
-- 貌似只能在终端执行
DELIMITER $
CREATE PROCEDURE myp1()
BEGIN 
	INSERT INTO ADMIN (USERNAME, `PASSWORD`)
	VALUES ('john1', 0000), ('rose', 0001), ('jack', 0002), ('tom', 0003), ('lin', 0004);
END $

CALL myp1()$

-- 2. 创建带 in 模式参数的存储过程
-- 根据女神名查询对应的男神信息
CREATE PROCEDURE myp2(IN beautyName VARCHAR(20))
BEGIN 
	SELECT bo.*
    FROM boys bo
    RIGHT JOIN beauty b ON bo.id = b.boyfriend_id
    WHERE b.name = beautyName;
END $

CALL myp2('小昭')$

-- 传入双参数，验证登陆
CREATE PROCEDURE myp4(IN username VARCHAR(20), IN PASSWORD VARCHAR(20))
BEGIN 
	DECLARE result VARCHAR(20) DEFAULT '';
    
    SELECT COUNT(*) INTO result
    FROM admin
    WHERE admin.username = username
    AND admin.password = PASSWORD;

    -- SELECT result;
    SELECT IF(result>0, 'success', 'failed');
END $

CALL myp4('lin', 4)$

-- 3. 创建带 out 模式的存储过程
-- 根据女神名返回男神名
CREATE PROCEDURE myp5(IN beautyName VARCHAR(20), OUT boyName VARCHAR(20))
BEGIN 
	SELECT bo.boyname INTO boyName
    FROM boys bo
    INNER JOIN beauty b ON bo.id = b.boyfriend_id
    WHERE b.name=beautyName;
END $

# 调用
CALL myp5('小昭', @bName)$
SELECT @bName$

-- 根据女神名返回对应男神名和魅力值
CREATE PROCEDURE myp6(IN beautyName VARCHAR(20), OUT boyName VARCHAR(20), OUT userCP INT)
BEGIN 
	SELECT bo.boyname, bo.userCP INTO boyName, userCP
    FROM boys bo
    INNER JOIN beauty b ON bo.id = b.boyfriend_id
    WHERE b.name=beautyName;
END $

# 调用
CALL myp6('小昭', @bName, @usercp)$
SELECT @bName$, @userCP$

-- 4. 带 inout 模式的存储过程
-- 传入 a,b 返回对应的翻倍值
CREATE PROCEDURE myp7(INOUT a INT, INOUT b INT)
BEGIN 
	SET a=a*2;
    SET b=b*2;
END $

SET @m=10$
SET @n=20$
call myp7(@m, @n)$

-- 删除
DROP PROCEDURE 存储过程名称;
-- 查看
SHOW CREATE PROCEDURE myp2;

-- 传入日期，返回日期字符串
CREATE PROCEDURE test_pro4(IN mydata DATETIME, OUT strDate VARCHAR(20))
BEGIN 
	SELECT DATE_FORMAT(mydata, '%y-%m-%d') INTO strDate;
END $

CALL test_pro4(now(), @strDate)$
SELECT @strDate$
```

## Function

区别：存储过程可以有任意个返回，函数有且仅有一个返回

创建语法：

CREATE FUNCTION 函数名(参数列表) RETURENS 返回类型
BEGIN
    函数体
END

参数列表：参数名 参数类型

函数体：肯定有 return，没有报错，可以不放在最后，但是不建议

函数体为空，可以省略 begin end

使用 delimiter 语句设置结束标记

调用语法： SELECT 函数名(参数列表)

```sql
-- 1. 无参有返回
-- 返回公司员工个数
CREATE FUNCTION myf1() RETURNS INT
BEGIN
    DECLARE c INT DEFAULT 0;
    SELECT COUNT(*) INTO c
    FROM employees;
    RETURN c;
END$
SELECT myf1()$

-- 报错：ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL
-- 设置：SET global log_bin_trust_function_creators=TRUE;

-- 2. 有参数返回
-- 根据员工名返回工资
CREATE FUNCTION myf2(empName VARCHAR(20)) RETURNS DOUBLE
BEGIN
    SET @sal=0;
    SELECT salary INTO @sal
    FROM employees
    WHERE last_name = empName;

    RETURN @sal;
END$
select myf2('Kochhar') $

-- 根据部门名返回平均工资
CREATE FUNCTION myf3(deptName VARCHAR(20)) RETURNS DOUBLE
BEGIN
    DECLARE sal DOUBLE;

    SELECT AVG(salary) INTO sal
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    WHERE d.department_name=deptName;

    RETURN sal;
END$
select myf2('Kochhar') $

-- 查看和删除，同存储过程
SHOW CREATE FUNCTION myf3();
DROP FUNCTION myf3;
```

## 流程控制

分支结构：

### if 函数

select if(expr1, expr2, expr3). expr1 成立则返回 expr2 否则 expr3

### case 结构

方式一：

CASE 变量|表达式|字段
WHEN 要判断的值 THEN 返回值 1 或语句 1;
WHEN 要判断的值 THEN 返回值 2 或语句 2;
...
ELSE 要返回的值 n 或语句 2;
END CASE;

方式二：

CASE
WHEN 要判断的条件1 THEN 返回值 1
WHEN 要判断的条件2 THEN 返回值 2
...
ELSE 要返回的值 n
END

可以作为表达式，嵌套在其他语句中使用，比如 BEGIN END 中/外

可以作为独立的语句去使用，只能放在 BEGIN/END 中

如果 WHEN 中条件成立，则执行 THEN 然后结束，如果都不满足，执行 ELSE。ELSE 可以省略。如果没有 ELSE 并且 WHEN 都不满足，返回 NULL

```sql
-- 存储过程，显示成绩
CREATE PROCEDURE test_case(IN score INT)
BEGIN
    CASE
    WHEN score >= 90 AND score <= 100 THEN SELECT 'A';
    WHEN score >=80 THEN SELECT 'B';
    WHEN score >=60 THEN SELECT 'C';
    ELSE SELECT 'D';
    END CASE;
END$

CALL test_case(95)$
```

## if 结构

if 条件1 then 语句1;
elseif 条件2 then 语句2;
...
[else 语句n;]
end if;

只能放在 begin/end 中

```sql
-- 根据传入成绩返回等级
CREATE FUNCTION test_if(score INT) RETURNS CHAR
BEGIN
    IF score >= 90 AND score <=100 THEN RETURN 'A';
    ELSEIF score >= 80 THEN RETURN 'B';
    ELSEIF score >= 60 THEN RETURN 'C';
    ELSE RETURN 'D';
    END IF;
END $
```

### 循环

分类：while, loop, repeat

循环控制：iterate, 对应 continue; leave 对应 break;

[标签:] loop 循环条件 do
    循环体
end while [标签];

[标签:] loop
    循环体
end loop [标签];

可用于模拟死循环

[标签:] repeat
    循环体
until 结束循环条件 [标签];

```sql
-- 批量插入 admin
CREATE PROCEDURE pro_while2(IN insertCount INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    WHILE i<insertCount DO
        INSERT INTO admin (username, `password`) VALUES (CONCAT('jjjj', i), '666');
        SET i=i+1;
    END WHILE;
END $

-- 添加 leave 控制
-- 清空并重置 index
TRUNCATE TABLE admin$

CREATE PROCEDURE pro_while3(IN insertCount INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    a: WHILE i<insertCount DO
        INSERT INTO admin (username, `password`) VALUES (CONCAT('jjjj', i), '666');
        IF
            i>= 20 THEN LEAVE a;
        END IF;
        SET i=i+1;
    END WHILE a;
END $
call pro_while3(100)$

-- iterate, 只插入偶数次
CREATE PROCEDURE pro_while4(IN insertCount INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    a: WHILE i<insertCount DO
        SET i=i+1;
        IF
          MOD(i, 2) != 0 THEN ITETATE a;
        END IF;
        INSERT INTO admin (username, `password`) VALUES (CONCAT('jjjj', i), '666');
    END WHILE a;
END $
call pro_while3(100)$
```