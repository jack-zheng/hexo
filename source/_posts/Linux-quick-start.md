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

## 账户管理

```sh
useradd -m jack002 # 创建 user 并在 home 下创建对应的文件夹
# root@945e2e63f891:/home# ls
# f2  f3  jack  jack002
# Linux 一切皆文件，添加用户其实就是往某一个文件中写入用户信息 /etc/passwd

userdel -r jack002 # 删除 user 并清空文件夹

usermod -d /home/233 jack002 # 修改用户目录, 233 并不存在，但是对应的信息还是设置到 passwd 中去了，所以修改前必须先手动创建文件夹
# root@945e2e63f891:/home# cat /etc/passwd
# jack002:x:1001:1001::/home/233:/bin/sh
# root@945e2e63f891:/home# ls
# f2  f3  jack  jack002

root@945e2e63f891:/home#
# root: 当前用户
# 945e2e63f891: 主机名
# /home: 当前路径
# '#' 超级用户

su username # 切换用户
# $: 提示符也会跟着改

exit # 退出用户切换

hostname # 查看主机名
hostname xxx # 修改主机名，需要重联生效

passwd username # 修改密码

passwd -l username # 锁用户
passwd -d username # 用户密码清空，也不能登陆
```

## 用户组管理

本质是对 /etc/group 进行修改

```sh
# -g        # 指定 gid, 不指定就自增
groupadd jackgroup
# root@945e2e63f891:/# cat /etc/group
# jackgroup:x:1002:
# jack520:x:520:

groupdel jackgroup # 删除 group

# -n    # 修改 id
# -G    # 设置用户组
groupmod -g 555 -n jack555 jack520

newgroup root # 切换组
```

## 拓展

/etc/passwd

```txt
jack002:x:1001:1001::/home/jack002:/bin/sh
用户名：密码（不可见，所以是x）：用户标识号（自增）：组标识号：注释性描述：主目录：登陆 shell
```

密码放在 /etc/shadow 加密过的 jack002:$6$/me.SpanXkOPad04$XO/jFHniPIenQQXFZhSOfwL7eQ0hQ..X5EWNigGrfh8sqZ6KA8wAFQtzCPpwgf.Ov9RIVp8hr9GcXB3un4Oax1:18746:0:99999:7:::

## 磁盘管理

```sh
df # 列出整体磁盘使用量
# root@945e2e63f891:/# df -h
# Filesystem      Size  Used Avail Use% Mounted on
# overlay          59G   21G   35G  38% /
# tmpfs            64M     0   64M   0% /dev
# tmpfs           7.9G     0  7.9G   0% /sys/fs/cgroup
# shm              64M     0   64M   0% /dev/shm
# /dev/vda1        59G   21G   35G  38% /etc/hosts
# tmpfs           7.9G     0  7.9G   0% /proc/acpi
# tmpfs           7.9G     0  7.9G   0% /sys/firmware
du # 当前磁盘使用量
# root@945e2e63f891:/home# du -a
# 4       ./jack002/.bashrc
# 4       ./jack002/.profile
# 4       ./jack002/.bash_logout
# 16      ./jack002
# 4       ./f2
# 4       ./jack/f1/f2/f3
# 8       ./jack/f1/f2
# 12      ./jack/f1
# 16      ./jack
# 0       ./f3
# 40      .

du -sm /*       # 系统根目录下每个文件夹占用空间
# 5       /bin
# 1       /boot
# 0       /dev

mount /dev/jack /mnt/jack # 将外部设备 jack 挂载到 mnt 下，实现访问

umount -f /mnt/jakc # 强制卸载
```

## 进程管理

1. 每个进程都有父进程
2. 两种运行方式：前台，后台
3. 服务基本都是后台运行，程序都是前台运行
4. 每个进程都有一个 id 号

```sh
# -a 当前终端运行的所有进程信息
# -u 一用户的信息显示进程
# -x 显示后台运行进程的参数
ps # 查看当前正在运行的进程信息
# ps -aux       
# ps -ef 可以查看父进程

# -p 显示父 id
# -u 显示组信息
pstree -pu # 进程树

kill -9 pid # 结束进程
```