---
title: Mysql 安装教程
date: 2020-09-16 21:29:43
categories:
- 配置
tags:
- mysql
---

## Windows 版本安装

1. 下载安装包 [官方地址](https://dev.mysql.com/downloads/mysql/) 下载比较小的，不到测试套件的版本即可
2. C 盘下新建 Mysql 文件夹，将下载的压缩包解压
3. 进去解压文件夹下，新建一个 my.ini 配置文件并添加配置
4. 将对应的 bin 路径添加到系统的 path 中去，做法和添加 JAVA_HOME 一样
5. 管理员模式打开终端，输入命令 `mysqld --initialize-insecure --user=mysql` 初始化，并且用户密码为空
6. 输入 `mysqld -install` 安装数据库，终端出现 `Service successfully installed` 表示安装成功
7. `net start mysql` 启动服务器
8. 输入 `mysql -u root -p` 不用输入密码直接回车, 出现mysql>表示配置完成
9. 输入 `alter user user() identified by "your-password";` 修改 root 用户密码
10. 输入 `net stop mysql` 关闭数据库

```my.ini
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=C:\Mysql\mysql-8.0.21-winx64
# 设置mysql数据库的数据的存放目录
datadir=C:\Mysql\mysql-8.0.21-winx64\data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
```

## Windows part issues

Windows 运行 `mysqld --initialize-insecure --user=mysql` 配置时报错 `由于找不到vcruntime140_1.dll,无法继续执行代码` 可以去 [官网](https://cn.dll-files.com/vcruntime140_1.dll.html) 下载 dll 文件放到 `C:\Windows\System32` 下即可

Idea 链接 mysql 后报错 `Server returns invalid timezone. Go to 'Advanced' tab and set 'serverTimezone' property manually`，可以通过设置 mysql 时区解决

1. cmd -> mysql -uroot -p 登录 DB
2. `show variables like'%time_zone';` 查看时区， Value 为 SYSTEM 则表示没有设置过
3. `set global time_zone = '+8:00';` 修改时区为东八区
4. 重试链接，问题解决

这只是临时方案，重启 DB 后时区会重置，可以去 my.ini 配置文件中添加配置

```config
[mysqld]
# 设置默认时区
default-time_zone='+8:00'
```

## MacOS 版本安装

```bash
brew install mysql # 使用 homebrew 安装
```

安装完毕的时候，终端回给出提示，最后的那段话比较值得注意

```txt
We've installed your MySQL database without a root password. To secure it run:
    mysql_secure_installation

MySQL is configured to only allow connections from localhost by default

To connect run:
    mysql -uroot

To have launchd start mysql now and restart at login:
  brew services start mysql
Or, if you don't want/need a background service you can just run:
  mysql.server start
```

翻译成人话就是

1. DB 安装成功，但是数据库 root 用户是没有密码的，你直接登陆会失败
2. 运行 mysql_secure_installation 给数据库设置密码
3. 使用命令 `mysql -uroot` 联接数据库
4. 后台启动使用 `brew services start mysql` 前台启动使用 `mysql.server start`

PS: 想要改密码得先启动服务，即运行 `brew services start mysql` 命令

在安全设置脚本中，mysql 会让你进行如重设 root 密码，删除匿名用户等操作，按照提示操作即可。以下是提示样本：

```txt
Jack > ~ > mysql_secure_installation

## 开始进行设置
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

## 是否进行安全设置
Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

## 设置密码复杂度，最低也要 *8* 位密码起步
Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 0
Please set the password for root here.

New password:

Re-enter new password:

Estimated strength of the password: 50
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

## 是否删除匿名用户
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

## 是否开放 root 用户远程访问
Disallow root login remotely? (Press y|Y for Yes, any other key for No) :

 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.

## 是否删除测试表
Remove test database and access to it? (Press y|Y for Yes, any other key for No) :

 ... skipping.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

## 是否重新加载使得配置生效
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done!
```

设置完毕之后就可以使用 `mysql -uroot -p` 登陆测试了。没有遇到其他问题，还挺顺利的 ε-(´∀｀; )
