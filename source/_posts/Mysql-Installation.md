---
title: Mysql Installation
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