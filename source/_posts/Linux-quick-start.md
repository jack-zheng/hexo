---
title: Linux 弹射起步
date: 2021-04-28 18:32:23
categories:
- linux
tags:
- 弹射起步
---

## 走进 Linux 系统

开机会启动很多程序，Windows 叫服务(service)，Linux 叫守护进程(daemon)

## 关机

linux 中没有报错，即成功

```sh
sync            # 同步数据

shutdown -h now # 马上关机

reboot          # 重启
```

## 目录结构

1. 一切皆文件
2. 根目录 `/`，所有文件都挂载在这个节点下

```sh
ls
# bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# bin: binary, 存放经常使用的命令
# boot: 启动 Linux 时使用的一些核心文件，包括连接文件和镜像文件
# dev: device, 存放外部设备，Linux 中访问设备和访问文件是一样的
# *etc: 存放所有系统管理所需的配置文件和子目录
# home: 用户主目录，每个用户有一个自己的目录，以自己命名
# lib: 系统最基本的动态连接共享库，类似于 Windows 的 dll 文件
# mnt: 系统提供目录为用户临时挂载文件系统，如光驱等
# lost+found: 一般为空，当系统非正常关机时，这里就会存放一些文件
# media: 媒体设备，U盘，光驱等
# *opt: 额外安装的软件所摆放的位置，比如安装一个数据库什么的，默认为空
# proc: 虚拟目录，系统映射，不管
# *root: 管理员目录
# sbin: s for super user, 村系统管理员使用的管理程序
# srv: 存放一些服务启动后需要提取的数据
# sys: linux2.6之后一个很大的变化，2.6后该目录下新出现一个文件系统 sysfs
# tmp: 临时文件，用完就丢
# *usr: 非常重要的目录，用户很多应用程序和文件都放在这个目录下，类似 Windows 的 program files 目录
# usr/bin: 系统用户使用的应用程序
# usr/sbin: 超级用户使用的应用程序
# usr/src: 内核源代码默认放置位置
# var: 放不断扩充的东西，比如日志
# run: 临时文件系统，存储系统启动以来的信息。重启时，这个目录下的文件应该被删除或清空
# www: 存放服务器网站相关资源，环境，网站目录
```

## 常用基本命令

```sh
mkdir -p f1/f2/f3   # 递归创建文件夹
rmdir -p f1         # 递归删除
rmdir f1            # 如果有子文件，删除会失败
```

## 内容查看

```sh
cat # 顺序
tac # 倒叙
nl  # 带行号

less 比 more 好，功能多，向下查询 /，向上查询 ？
tail -n 20 # 最后20行
```

## 链接

硬链接：A...B, 假设 B 是 A 的硬连接，那么他们指向同一个文件，允许一个文件多个路径，用这种机制可以放置误删
软连接：类似 Windows 下的快捷方式

```sh
touch f1

echo "hello" > f1

ln f1 f2 # 创建硬链接

ln -s f1 f3 # 创建软链接

rm f1

cat f2      # f1, f2 硬连接，删除还存在
# hello

cat f3      # f1, f3 软链接，删除即不见
# cat: f3: No such file or directory
```


