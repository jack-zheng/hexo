---
title: Linux命令行与shell脚本编程大全 第一章 Linux 命令行
date: 2021-05-31 10:04:57
categories:
- Shell
tags:
- Linux命令行与shell脚本编程大全 3rd
- TODO
---

## Chapter 1: Starting with Linux Shells

### What Is Linux

Linux 系统主要由一下 4 部分组成

* The Linux kernel
* The GUN utilities
* A graphical desktop environment
* Application software

#### Looking into the Linux Kernel

Linux 系统的核心就是 kernel，它起到调度硬件软件资源的作用。

kernel 有四个主要的功能

* System memory management
* Software program management
* Hardware management
* Filesystem management

##### System Memory management

下面的内容和操作系统相关，很多概念我都不是很感兴趣，可以先跳过

## Chapter 2: Getting to the Shell

终端介绍，跳过

## Chapter 3: Basic bash Shell Commands

### Interacting with the bash Manual

man page 的结构如下

| Section       | Description                                   |
| :------------ | :-------------------------------------------- |
| Name          | Displays command name and a short description |
| Syopsis       | Shows command syntax                          |
| Configuration | Provides configuration information            |
| Description   | Describes command generally                   |
| Options       | Describes command option(s)                   |
| Exit Status   | Defines command exit status indicator(s)      |
| Return Value  | describes command return value(s)             |
| Errors        | Provides command return value(s)              |
| Environment   | Describes envrionment variable(s) used        |
| Files         | Defines files used by command                 |
| Versions      | Describes command version information         |
| Conforming To | Provides standards followed                   |
| Notes         | Describes additional helpful command material |
| Bugs          | Provides the location to report found buds    |
| Example       | Shows command use examples                    |
| Authors       | Provides information on command developers    |
| Copyright     | Defines command code copyright status         |
| See Also      | Refers similar available commands             |

### Navigating the Filesystem

常见的目录及用途

| Directory | Usage                                                                                       |
| :-------- | :------------------------------------------------------------------------------------------ |
| /         | root of the virtual directory, where normally, no files are placed                          |
| /bin      | binary directory, where GNU user-level utilities are stored                                 |
| /boot     | boot directory, where boot files are stored                                                 |
| /dev      | device directory, where Linux creates device nodes                                          |
| /etc      | system configuration files directory                                                        |
| /home     | home directory, where Linux creates user directories                                        |
| /lib      | library directory, where system and application library files are stored                    |
| /media    | media directory, a common place for mount points used for removable media                   |
| /mnt      | mount directory, another common place for mount points used for removable media             |
| /opt      | optional directory, often used to store third-part software packages and data files         |
| /proc     | process directory, where current hardware and process information is stored                 |
| /root     | root home directory                                                                         |
| /sbin     | system binary directory, where many GNU admin-level utilities are stored                    |
| /run      | run directory, where runtime data is held during system operation                           |
| /srv      | service directory, where local services stre their files                                    |
| /sys      | system directory, where system hardware information files are stored                        |
| /tmp      | temporary directory, where temporary work files can be crated and destroyed                 |
| /usr      | user binary directory, where the bulk of GUN user-level utilities and data files are stored |
| /var      | variable directory, for files that change frequently, such as log files                     |

### Listing Files and Directories

#### Displaying a basic list

展示文件命令 `ls`

```sh
# 如果终端没有配置颜色，使用 -F 区分文件和文件夹,可执行文件后会加 *
ls -F
# test3b.sh*      tmp_folder/

# -a 显示所有文件，包括隐藏文件
ls -a
# . npm  Documents

# -R 循环显示子目录
ls -F -R
# badtest*        nohup.out       search.xml      search.xml.bak  tmp_folder/
#
# ./tmp_folder:
# test1.sh*       test10b.out
```

#### Displaying a long listing

```sh
# Displaying a long list
ls -l
# total 10112
# -rwxr--r--   1 i306454  staff      159 May 30 15:56 badtest
# -rw-------   1 i306454  staff      138 Jun  2 19:12 nohup.out
```

long list 显示格式说明

* The file type, directory(d), regular file(-), linked file(l), character device(c) or block device(b)
* The file permissions
* The number of file hard links
* The file owner username
* The file primary group name
* The file byte size
* The last time file was modified
* The filename or directory name

long list 是一个比较强力的模式，你可以收集到很多信息

#### Filtering listing output

过滤文件

```sh
ls -l bad*
# -rwxr--r--  1 i306454  staff  159 May 30 15:56 badtest
```

可用的过滤符

* ? 单个字符
* \* 多个字符
* \[\] 多选，可以是 [ai], [a-i],[!a]

使用星号的过滤方法也叫做 file globbing

### Handling Files

过一下常用的文件处理命令

#### Creating files

```sh
# touch 创建空文件
touch test_one
ls -l test_one
# -rw-r--r--  1 i306454  staff  0 Jun  4 11:25 test_one

# 可以在不改变文件内容的情况下更新最后改动时间，这个之前倒是不知道
ls -l test_one 
# -rw-r--r--  1 i306454  staff  3 Jun  4 11:27 test_one
touch test_one 
ls -l test_one 
# -rw-r--r--  1 i306454  staff  3 Jun  4 11:29 test_one

# -a 只修改最近访问时间
ls -a test_one
# 不过 Mac 不支持这个参数
```

#### Copying files

format: `cp source destination`, copy 的文件是一个全新的文件

```sh
# -i 当文件已经存在时，询问是否覆盖
cp -i test_one test_two 
# overwrite test_two? (y/n [n]) n
# not overwritten

# -d 只显示文件夹，不显示文件夹内容
ls -Fd tmp_folder
# tmp_folder/
ls -F tmp_folder/
# test1.sh*       test10b.out...
```

#### Linking files

Linux 中你的文件可以有一个物理主体和多个虚拟链接，这种链接即为 links。系统中有两种链接

* A symbolic link
* A hard link

A symbolic link is simply a physical file that points to another file somewhere in the virtual directory structure. The two symnolically linked together files do not share the same contents.

```sh
ln -s test_one  sl_test_one
ls -l *test_one
# lrwxr-xr-x  1 i306454  staff  8 Jun  4 12:30 sl_test_one -> test_one
# -rw-r--r--  1 i306454  staff  3 Jun  4 11:29 test_one

# -i 显示 inode 名字
ls -i *test_one
51540816 sl_test_one    51538439 test_one
```

hard link 是一个虚拟链接，你可以通过它对原文件做修改

```sh
ln test_two hl_test_two

ls -il *test_two
# 51538882 -rw-r--r--  2 i306454  staff  3 Jun  4 11:38 hl_test_two
# 51538882 -rw-r--r--  2 i306454  staff  3 Jun  4 11:38 test_two
```

**Note** 创建 hard link 要求你创建的地方是同一个物理空间，如果是分开的空间，只能创建 symblic link.

查阅下来发现，符号链接和硬链接最主要的区别有

* symbolic link 和原文件有不同的 inode, hard link 和原文件相同
* hard link 启动备份的作用，当所有指向同一个 inode 的文件都删除了文件才删除
* symbolic link 保存原文件路径，当原文件删除了，link 的文件内容就消失了

#### Renaming files

Renaming files is called moving files. `mv` won't change the inode number.

```sh
touch file1
mv file1 file2
ls file*
# file2
```

`mv` 也支持整个文件夹的迁移，且不需要加任何参数

#### Deleting files

```sh
rm -i file2
# remove file2?
```

### Managing Directories

```sh
mkdir New_Dir

# 创建多级文件夹
mkdir -p folder1/folder2/folder3
ls -R folder1
# folder2

# folder1/folder2:
# folder3

# folder1/folder2/folder3:

# rmdir 只能删除空文件夹
rmdir folder1
# rmdir: folder1: Directory not empty

rm -rf folder1
```

### Viewing File Contents

使用 `file` 瞥一眼文件

```sh
file folder1
# folder1: directory
file file2
# file2: empty
ile search.xml
# search.xml: XML 1.0 document text, UTF-8 Unicode text, with very long lines, with overstriking
file badtest
# badtest: Bourne-Again shell script text executable, ASCII text
```

`cat` 全揽文件

```sh
cat tree.txt

# -n 显示行号
cat -n badtest 
    #  1  #!/usr/local/bin/bash
    #  2  # Testing closing file descriptors
    #  3
    #  4  exec 3> test17file
    #  5
    #  6  echo "This is a test line of data" >&3
    #  7
    #  8  exec 3>&-
    #  9
    # 10  echo "This won't work" >&3
    # 11
    # 12
    # 13

# 只显示 non-blank 的行号
cat -b badtest 
    #  1  #!/usr/local/bin/bash
    #  2  # Testing closing file descriptors

    #  3  exec 3> test17file

    #  4  echo "This is a test line of data" >&3

    #  5  exec 3>&-

    #  6  echo "This won't work" >&3

# 使用 ^I 代替 tab, Mac 不支持
cat -T badtest
```

#### Using the more command

`cat` 只能全文显示，`more` 显示一部分并让你自己选择后面的动作

#### Using the less command

别被它的名字骗了，其实他是 more 的增强版 for phrase 'less is more'

#### Viewing parts of a file

`tail` 默认只显示文件的最后 10 行，`-n` 指定显示行数 `tail -n 2 file`

`head` 默认显示开头 10 行，`- 5` 指定行数 `head -3 file`. 格式和 tail 不统一，真是干了

试了下，这两个命令都可以用 `-n 3` 和 `-3` 的格式，没区别

## Chapter 4: More bash Shell Commands

### Monitoring Programs

#### Peeking at the processes

Linux 系统中，用 process 表示运行着的系统。可以用 `ps`(process status) 命令查看.

默认情况下，只显示四个内容，process ID，terminal that they are running from, and the CUP time the process has used.

```sh
ps 
  PID TTY           TIME CMD
  647 ttys000    0:02.15 -zsh
12163 ttys000   11:26.77 /Users/i306454/SAPDevelop/tools/sapjvm_8/bin/java -Dlog4j.co
12183 ttys000    0:01.12 tail -f /Users/i306454/SAPDevelop/workspace/trunk/tomcat-sfs
 1238 ttys001    0:09.65 /bin/zsh -l
 9378 ttys002    0:03.04 /bin/zsh --login -i
```

`ps` command 有三种类型的参数

* Unix style parameters
* BSD style parameters
* GNU long parameters

> Unix-style parameters

简单摘录几个, 而且书上列的只是一部分，主要记住几个常用的就行了

| Parameter | Description                                                                 |
| :-------- | :-------------------------------------------------------------------------- |
| -A        | Shows all processes                                                         |
| -N        | Shows the opposite of the specified parameters                              |
| -a        | Shows all processes except session headers and processes without a terminal |
| -d        | Shows all processes except session headers                                  |
| -e        | Shows all processes                                                         |
| -f        | Displays a full format listing                                              |
| -l        | Displays a long listing                                                     |

```sh
ps -ef | head
#   UID   PID  PPID   C STIME   TTY           TIME CMD
#     0     1     0   0 10:09AM ??         1:23.22 /sbin/launchd
#     0    64     1   0 10:09AM ??         0:03.00 /usr/sbin/syslogd
# ...

ps -l | head
# UID   PID  PPID        F CPU PRI NI       SZ    RSS WCHAN     S             ADDR TTY           TIME CMD
# 501   647   646     4006   0  31  0  5457412   5200 -      S+                  0 ttys000    0:04.03 -zsh
```

* UID: The user responsible for launching the process
* PID: The process ID of the process
* PPID: The PID of the parent process(if a process is start by another process)
* C: Processor utilization over the lifetime of the process
* STIME: The system time when the process started
* TTY: The termnal device from which the porcess was launched
* TIME: The cumulative CUP Time required to run the process
* CMD: The name of the program that wat started
* F: System flags assigned to the process by the kernel
* S: The state of the process. O-running on proecssor; S-sleeping; R-runnable, waiting to run; Z-zombie, process terminated but parent not availale; T-process stopped;
* PRI: The priority of the process(higher numbers mean low priority)
* NI: The nice value, which is used for determining priorites
* ADDR: The memory address of the process
* SZ: Approximate amount of swap space required if the process was swapped out
* WCHAN: Address of the kernel function where the process is sleeping

其他两种我很少用，先留着把，有机会再补全

#### Real-time process monitoring

`ps` 只能显示一个时间点的 process 状态，如果要实时显示，需要用到 `top` 命令

```sh
top
# PID    COMMAND      %CPU TIME     #TH    #WQ  #PORT MEM    PURG   CMPRS  PGRP  PPID STATE    BOOSTS          %CPU_ME %CPU_OTHRS UID  FAULTS     COW     MSGSENT    MSGRECV   SYSBSD     SYSMACH    CSW        PAGEIN IDLEW    POWE INSTRS    CYCLES    USER
# 1841   com.docker.h 36.7 68:12.08 13     0    37    19G    0B     629M   1710  1830 sleeping *0[1]           0.00000 0.00000    501  202373842+ 473     569        335       85444924+  920        48973831+  17     3949456+ 59.4 469486491 786452501 i306454
```

#### Stopping processes

Linux 系统中使用 signals 来和其他 process 交互。常用的 signals 列表

| Signal | Name | Description                                         |
| :----- | :--- | :-------------------------------------------------- |
| 1      | HUP  | Hangs up                                            |
| 2      | INT  | Interrupts                                          |
| 3      | QUIT | Stops running                                       |
| 9      | KILL | Unconditionally terminates                          |
| 11     | SEGV | Produces segment violation                          |
| 15     | TERM | Terminates if possible                              |
| 17     | STOP | Stops unconditionally, but doesn't terminate        |
| 18     | TSTP | Stops or pauses, but continues to run in background |
| 19     | CONT | Resumes execution after STOP or TSTP                |

**The kill command** 只有 process 的 owner 或者 root user 有权限杀死进程。 格式：`kill 3904`

**The killall command** 可以根据名字关闭多个进程 `killall http*`

### Monitoring Disk Space

#### Mounting media

```sh
# 显示当前挂在的设备
mount
# /dev/disk1s1s1 on / (apfs, sealed, local, read-only, journaled)
# devfs on /dev (devfs, local, nobrowse)
# ...
```

显示信息：

* The device filename of the media
* The mount point in the virtual directory where the media is mounted
* The filesystem type
* The access status of the mounted media

手动挂载，你需要是 root 或者用 sudo，格式为 `mount -t type device directory`, sample `mount -t vfat /dev/sdb1 /media/disk`

type 指定了设备的文件类型，如果你想要和 Windows 下共享这个设备，你最好使用下面这些文件类型

* vfat: Windows long filesystem
* ntfs: Windows advanced filesystem used in Windows NT, XP and Vista
* iso9660: The standard CD-ROM filesystem

`unmount [directory | device]` 解绑，如果解绑时有 process 还在这个设备上运行，系统会阻止你

#### Using the df command

当你想要看看磁盘还有多少可用空间时。。。

`df` command allows you to easily see what's happening on all the mounted disks

df - display free disk space

```sh
df -h
# Filesystem       Size   Used  Avail Capacity iused      ifree %iused  Mounted on
# /dev/disk1s1s1  932Gi   14Gi  749Gi     2%  553757 9767424403    0%   /
# devfs           190Ki  190Ki    0Bi   100%     656          0  100%   /dev
# ...
```

#### Using the du command

df 是查看磁盘细心， du 是查看磁盘下的文件信息

The `du` command shows the disk usage or a specific directory(by default, the current directory)

du - display disk usage statistics

```sh
# 说是文件也会显示，问什么我这里看不到。。。
du .
# 104     ./tmp_folder
# 0       ./folder1/folder2/folder3
# 0       ./folder1/folder2
# 0       ./folder1
```

一些可选参数

* -c: 统计结果
* -h: 方便阅读的结果
* -s: Summarizes each argument

### Working with Data Files

列出一些处理大量数据时用到的工具

#### Sorting data

```sh
cat file1
# one
# two
# three
# four
# five
sort file1
# five
# four
# one
# three
# two
cat file2
# 1
# 2
# 100
# 45
# 3
# 10
# 145
# 75
sort file2
# 1
# 10
# 100
# 145
# 2
# 3
# 45
# 75

# 使用 -n 指定数字排序
sort -n file2
# 1
# 2
# 3
# 10
# 45
# 75
# 100
# 145

cat file3
# Apr
# Aug
# Dec
# Feb
# Jan
# Jul
# Jun
# Mar
# May
# Nov
# Oct
# Sep

# 按月份排序
sort -M file3
# Jan
# Feb
# Mar
# Apr
# May
# Jun
# Jul
# Aug
# Sep
# Oct
# Nov
# Dec
```

其他比较常见的参数

* -t 指定分割符
* -k 指定排序的列
* -r 倒序

```sh
# 当前文件夹下的文件倒序排列
du -sh * | sort -nr
#  52K    tmp_folder
# 4.0K    tree.txt
# 4.0K    test_thr
# ...
```

#### Searching for data

`grep [options] patttern [file]`

一些有趣的可选参数

* -v 挑选不 match 的那些
* -n 行号
* -o 只显示配的内容
* -c 显示匹配的数量
* -e 多个匹配 `grep -e t -e f file1`
* 使用正则 grep [tf] file1

#### Compressing data

Linux 系统中的压缩工具

| Utility  | File Extension | Description                                                                         |
| :------- | :------------- | :---------------------------------------------------------------------------------- |
| bzip2    | .bz2           | Uses the Burrows-Wheeler block sorting text compression algorith and Fuffman coding |
| compress | .Z             | Original Unix file compression utility; starting to fade away into obscurity        |
| gzip     | .gz            | The GUN Project's compression utility; uses Lempel-Ziv coding                       |
| zip      | .zip           | The Unix version of the PKZIP program for Windows                                   |

gzip 是 Linux 中使用度最高的压缩工具，它由三部分组成

* gzip for compressing files
* gzcat for displaying the contents of compressed text files
* gunzip for uncompressing files

```sh
gzip file1
ls -l file1*
# -rw-r--r--  1 i306454  staff  50 Jun  4 13:57 file1.gz
gzip file*
# gzip: file1.gz already has .gz suffix -- unchanged
ls -l file*
# -rw-r--r--  1 i306454  staff  50 Jun  4 13:57 file1.gz
# -rw-r--r--  1 i306454  staff  46 Jun  4 13:58 file2.gz
# -rw-r--r--  1 i306454  staff  34 Jun  4 14:02 file3.gz
```

#### Archiving data

虽然 zip 挺好用，但是 Linux 上用的最多的还是 tar command. tar 本来是用来归档到 tape device 的，但是它也能用来归档到文件，后来还变得越来越受欢迎了

`tar function [options] object1 object2`

The tar Command Functions

| Function | Long Name     | Description                                                                                                         |
| :------- | :------------ | :------------------------------------------------------------------------------------------------------------------ |
| -A       | --concatenate | Appends an existing tar archive file to another existing tar archive file                                           |
| -c       | --create      | Create a new tar archive file                                                                                       |
| -d       | --diff        | Checks the differences between a tar archive file and the filesystem                                                |
|          | --delete      | Deletes from an existing tar archive file                                                                           |
| -r       | --append      | Appends files to the end of an existing archive file                                                                |
| -t       | --list        | Lists the contents of an existing tar archive file                                                                  |
| -u       | --update      | Appends files to an existing tar archive file that are newer than a file with the same name in the existing archive |
| -x       | --extract     | Extract files from an existing archive file                                                                         |

The tar Command Options

| Option  | Description                                              |
| :------ | :------------------------------------------------------- |
| -C dir  | Changes to the specified directory                       |
| -f file | Output results to file(or device)                        |
| -j      | Redirects output to the bzip2 command for compression    |
| -p      | Preserves all file permissions                           |
| -v      | Lists files as they are processed                        |
| -z      | Redirects the output to the gzip command for compression |

```sh
# -c create new tar file
# -v list process file
# -f output result to file
tar -cvf test.tar tmp_folder/
# a tmp_folder
# a tmp_folder/test11.sh
# a tmp_folder/test2.sh
# a ...

ls test*
# test.tar 

# 并不会解压，只是看看
# -t list contents in tar
tar -tf test.tar 
# tmp_folder/
# tmp_folder/test11.sh
# tmp_folder/test2.sh
# ...

# 解压
tar -xvf test.tar 
# x tmp_folder/
# x tmp_folder/test11.sh
# ...
```

**Tip** 网上下的包很多都是 `.tgz` 格式的，是 gzipped tar files 的意思，可以用 `tar -zxvf filename.tgz`

## Chapter 5: Understanding the Shell

这章将学习一些 shell process 相关的知识，子 shell 和 父 shell 的关系等

### Exploring Shell Types

你默认启动的 shell 是配置在 `/etc/passwd` 文件中的

```sh
cat /etc/passwd
# ...
# root:*:0:0:System Administrator:/var/root:/bin/sh
# ...
ls -lF /bin/sh 
# -rwxr-xr-x  1 root  wheel  120912 Jan  1  2020 /bin/sh*

# 其他一些自带的 sh
ls -lF /bin/*sh
# -r-xr-xr-x  1 root  wheel  1296704 Jan  1  2020 /bin/bash*
# -rwxr-xr-x  1 root  wheel  1106144 Jan  1  2020 /bin/csh*
# -rwxr-xr-x  1 root  wheel   277440 Jan  1  2020 /bin/dash*
# -r-xr-xr-x  1 root  wheel  2585424 Jan  1  2020 /bin/ksh*
# -rwxr-xr-x  1 root  wheel   120912 Jan  1  2020 /bin/sh*
# -rwxr-xr-x  1 root  wheel  1106144 Jan  1  2020 /bin/tcsh*
# -rwxr-xr-x  1 root  wheel  1347856 Jan  1  2020 /bin/zsh*
```

### Exploring Parent and Child Shell Relationships

```sh
ps -f               
  # UID   PID  PPID   C STIME   TTY           TIME CMD
  # 501   667   665   0 10:10AM ttys000    0:03.74 -zsh
  # 501  1454  1433   0 10:10AM ttys001    0:00.99 /bin/zsh -l
  # 501  2027   637   0 10:11AM ttys002    0:00.38 /bin/zsh --login -i

# 在 zsh 中启动一个 bash
bash

ps -f
  # UID   PID  PPID   C STIME   TTY           TIME CMD
  # 501   667   665   0 10:10AM ttys000    0:03.74 -zsh
  # 501  1454  1433   0 10:10AM ttys001    0:01.04 /bin/zsh -l
  # 501 12146  1454   0  4:07PM ttys001    0:00.01 bash
  # 501  2027   637   0 10:11AM ttys002    0:00.38 /bin/zsh --login -i
# 可以看到新建了一个 bash process, PPID 是 /bin/zsh 的地址
```

上面的例子中，bash 就是 zsh 的子 shell, 他会复制一部分父 shell 的环境变量，这里会导致一些小问题，第6章会介绍。子 shell 也叫 subshell. subshell 可以再建 subshell. `ps --forest` 可以显示树桩结构，不过貌似 mac 不支持

#### Looking at process lists

一行运行多个 cmd, 使用 semicolon 分割 `pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls` 但是它并不是一个 process，将它用括号包裹之后，会启动 subshell 运行它 `(pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls)` 和这个语法相似的还有 `{ command; }` 这个不会启动 subshell. 可以通过打印 `$BASH_SUBSHELL` 变量来验证

```sh
(pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL)
# ...
# 1
pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL
# 0
(pwd; (echo $BASH_SUBSHELL))
# /Users/i306454
# 2
```

#### Creatively using subshells

#### Investigation background mode

`slepp` - 等待 x 秒

```sh
# & 符号设置后台运行
sleep 3000 &
# [1] 12603
ps
# 12391 ttys001    0:00.03 bash
# 12603 ttys001    0:00.00 sleep 3000
```

#### Putting process lists into the background

a process list is a command or series of commands executed within a subshell.

```sh
(sleep 2 ; echo $BASH_SUBSHELL ; sleep 2)
# 1
(sleep 2 ; echo $BASH_SUBSHELL ; sleep 2) &
# [2] 12658
ps
# 12658 ttys001    0:00.00 bash
# 1

# [2]+  Done                    ( sleep 2; echo $BASH_SUBSHELL; sleep 2 )
```

background 运行脚本 not have your terminal tied up with subshell's I/O

sleep 和 echo 的 sample 只是示范，工作中，你可能会后台执行 tar `(tar -cf Rich.tar /home/rich ; tar -cf My.tar /home/christine)&`

#### Looking at co-processing

Co-processing does two thins at the same time. coproc 会起一个后台的 job 运行对应的命令

```sh
coproc sleep 10
# [2] 12746
jobs 
# [1]-  Running                 sleep 3000 &
# [2]+  Done                    coproc COPROC sleep 10
```

默认的 coproc 起的 job 名字为 COPRO, 你也可以指定名字, curly bracket({) 换括号后面要接空格，语法规定。一般用默认的名字就行，只有当你需要和他们通信时，才会特别的取一个名字

```sh
coproc My_Job { sleep 10; }
# [2] 12848
jobs
# [2]+  Running                 coproc My_Job { sleep 10; } &
```

后面还跟了一个 `ps --forest` 的实验，没法做 ╮(￣▽￣"")╭

Just remember taht spawning a subshell can be **expensive and slow**. Creating nested subshells is even more so!

### Understanding Shell Built-In Commands

Built-in commands and non-built-in commands

#### Looking at external commands

external command 也被叫做 filesystem command, 是在 bash shell 之外的，通常放在 /bin, /usr/bin, /sbin 或者 /usr/sbin

`ps` 就是一个 external 的 command

```sh
which ps 
# /bin/ps
type -a ps 
# ps is /bin/ps
```

每当 external command 执行时，都会创建一个 child process, 这种行为叫做 forking.

#### Looking at built-in comands

Built-in commands 不需要 child process 就能执行。他们是 shell 工具集的一部分。

```sh
type exit
# exit is a shell builtin
type cd
# cd is a shell builtin
```

他们不需要 fork 或者运行文件，所以他们更快，效率更高。

有些 cmd 有两个版本，which 只会显示 external command

```sh
type -a echo 
# echo is a shell builtin
# echo is /bin/echo
which echo 
# /bin/echo
```

#### Using the history command

显示 cmd 的历史记录

```sh
history 
  # ...
  #  42  code test19
  #  43  ./test19
  # ...
```

**Tip** 设置环境变量 HISTSIZE 改变数量上限

使用 `!!` 执行上一条命令, bash 的历史记录会存在 `.bash_history` 文件中，当前 shell 的历史存在内存中，退出后存到文件中，通过 `history -a` 强制立刻写入文件

#### Using command aliases

为了简化输入，有了别名(alias)。

```sh
# 显示自带的别名
alias -p

alias li='ls -li'
li
# total 8
#  5091820 drwx------@  3 i306454  staff    96 Aug 20  2020 Applications
```

## Using Linux Environment Variables

Environment variables are set in lots of places on the Linux system, and you should know where these places are.

这章将介绍环境变量存储的位置，怎么创建自己的环境变量，还介绍怎么使用 variable arrays.

### Exploring Environment Variables

bash shell 使用 environment variable 存储 shell session 和 工作环境相关的信息。环境变量分两种

* Gloabl variables
* Local variables

#### Looing at global environment variables

Gloabl variables 是所有 shell 都可见的，Local variables 是当前 shell 才可见的。

```sh
# 查看 gloabl variables
printenv
# SHELL=/bin/zsh
# LSCOLORS=Gxfxcxdxbxegedabagacad
# PIPENV_VENV_IN_PROJECT=1
# ...

# 输出单个变量
printenv HOME
# /Users/i306454

# env 貌似不能输出单个变量
env HOME
# env: HOME: No such file or directory

# 还可以用 echo
echo $HOME
# /Users/i306454
```

#### Looing at local environment variables

Linux 默认为每个 shell 定义基本的 local variables, 当然你也可以自定。系统中并没有输出本地变量的命令，但是有 set 可以输出 global + local

```sh
set
# '!'=0
# '#'=0
# '$'=13377
```

env vs printenv vs set:

* set = global + local + user-defined variables, result is sorted
* env has additional functionality that printenv not have

### Setting User-Defined Variables

#### Setting local user-defined variables

```sh
echo $my_var

my_var=Hello
echo $my_var
# Hello

# 包含空格的，需要用单/双引号包裹
my_var=Hello world
# bash: world: command not found
my_var='Hello world'
echo $my_var
# Hello world

# 新启一个 bash, 访问不到之前定义的 local variable
bash
echo $my_var
#
```

user-defined local varibale 使用小写，global 的使用大写。Linux 中的变量是区分大小写的。

#### Setting global environment variables

创建 gloabl variable 的方法：先创建一个 local variable，然后 export 成一个 global environment

```sh
my_var="I am Gloabl now"
export my_var
bash
echo $my_var
# I am Gloabl now
```

但是，在 child shell 中修改 global variable 并**不会**影响到 parent shell 中的值，这个好神奇, 即使用 export 在 subshell 中修改也不行

```sh
my_var="Null"
echo $my_var
# Null
exit
# exit
echo $my_var
# I am Gloabl now

bash 
export my_var="Null"
echo $my_var
# Null
exit
# exit
echo $my_var
# I am Gloabl now
```

#### Removing Environment Variables

使用 `unset`

```sh
echo $my_var
# I am Gloabl now
unset my_var
echo $my_var
#
```

**Tip** 当你向对变量做什么的时候，不需要加 $, 当你想要用变量做什么的时候，需要加 $. printenv 除外。

和之前的规则一样，当你在 subshell 中 unset 一个 global variable 时，这个 unset 只在 subshell 中生效，parent shell 中变量还是存在的

#### Uncovering Default Shell Environment Variables

Bash shell 除了自己定义一些环境变量外，还从 Unix Bourne shell 那边继承了一下变量过来。

The bash Shell Bourne Variables

| Variable | Description                                                                                                                             |
| :------- | :-------------------------------------------------------------------------------------------------------------------------------------- |
| CDPATH   | A colon-separated list of directories used as a search path for the cd command                                                          |
| HOME     | The current user's home directory                                                                                                       |
| IFS      | A list of characters that separate fields used by the shell to split text strings                                                       |
| MAIL     | The filename fo the current user's mailbox(The bash shell checks this file for new mail.)                                               |
| MAILPATH | A colon-separated list of multiple filenames for the current user's mailbox(The bash shell checks each file in this list for new mail.) |
| OPTARG   | The value of the last option argument processed by the getopt command                                                                   |
| OPTIND   | The index value of the last option argument processed by the getopt command                                                             |
| PATH     | A colon-separated list of directories where shell looks for commands                                                                    |
| PS1      | The primary shell command line interface prompt string                                                                                  |
| PS2      | The sceondary shell command line interface prompt string                                                                                |

除了这些，bash shell 还提供了一些自定义的变量, 太长了，不列了。

### Setting the PATH Environment Variable

当你在终端输入一个 external command 时，系统就会根据 PATH 中的路径找命令. 路径用冒号分割。

```sh
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# 添加路径
PATH=$PATH:/home/jack/Scripts
```

**Tips** 如果 subshell 中也要用到新加的路劲，你就要 export 它。有一个技巧是，可以在 PATH 中添加当前路径 `PATH=$PATH:.`

#### Locating System Environment Variables

前面我们介绍了如何使用这些变量，那么怎么将他们做持久化呢。当你启动一个 shell 的时候，系统会到 setup file or environment files 里面去加载这些变量。

你可以通过三种方式启动一个 bash shell:

* As a default login shell at login time
* As an interactive shell that is started by spawning a subshell
* As a non-inactive shell to run a script

#### Understanding the login shell process

当你登陆系统的时候，bash shell 开启了一个 login shell. login shell 会从一下五个文件中加载配置：

* /etc/profile
* $HOME/.bash_profile
* $HOME/.bashrc
* $HOME/.bash_login
* $HOME/.profile

`/etc/profile` 是所有用户登陆时都会执行的文件，其他的几个就是用户可以自定一的。

```sh
cat /etc/profile
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export PAGER=less
export PS1='\h:\w\$ '
umask 022

for script in /etc/profile.d/*.sh ; do
        if [ -r $script ] ; then
                . $script
        fi
done
```

脚本中循环处理 profile.d 文件夹下的内容，这个文件夹是专门用来放一下 application-specific startup file taht is executed by the shell when you log in.

```sh
ls -lF /etc/profile.d/
# total 8
# -rw-r--r--    1 root     root           295 May 30  2020 color_prompt
# -rw-r--r--    1 root     root            61 May 30  2020 locale.sh
```

自定义位置文件加载顺序如下，第一个被找到后，其他的就不加载了

* $HOME/.bash_profile
* $HOME/.bash_login
* $HOME/.profile

`.bashrc` 不在其中，因为它会在其他 process 中被调用

#### Understanding the interactive shell process

当你在终端输入 bash 时，你会启动一个 interactive shell. 当你启动 interactive shell 的时候，它不会加载 /etc/profile 中的内容。它只会 check .bashrc 中的配置。

.bashrc 做两件事

1. check for a common bashrc file in /etc directory
2. provides a place for user to enter personal command alias + provide script functions

#### Understanding the non-interactive shell process

没遇到过使用场景，先 pass

#### Making environment variables persistent

将自定义的变量存在 $HOME/.bashrc 是一个极好的习惯

### Learning about Variable Arrays

```sh
# 定义数组 
mytest=(one two three four five)
echo $mytest
# one
echo ${mytest[2]}
# three

echo ${mytest[*]}
# one two three four five

mytest[2]=seven
echo ${mytest[*]}
one two seven four five
```

可以通过 unset 移除某个元素，但是移除之后，print 不会显示，但是它位置还是占着的

```sh
unset mytest[2]
${mytest[*]}
# one two seven four five
echo ${mytest[2]}

echo ${mytest[3]}
# four
unset mytest
bash-5.1$ echo ${mytest[*]}

```

有时候 arrays 的使用挺复杂的，一般我们不再脚本中使用它，而且兼容性也不是很好。

## Understanding Linux File Permissions

### Linux Security

Linux 系统的 security 核心是 account 这个概念。每个访问的用户都有一个唯一的账户，权限就是根据账户设置的。下面将介绍一些账户相关的文件和工具包。

#### The /etc/passwd file

/etc/passed 文件中存储这一些 UID 相关的信息，root 是管理员账户，有固定的 UID 0.

```sh
cat /etc/passwd
# root:*:0:0:System Administrator:/var/root:/bin/sh
# daemon:*:1:1:System Services:/var/root:/usr/bin/false
# _uucp:*:4:4:Unix to Unix Copy Protocol:/var/spool/uucp:/usr/sbin/uucico
```

系统还会为一些非用户的 process 创建 account，这些叫做 system accounts.

All services that run in background mode need to be logged in to the Linux system under a system user account.

Linux 将 500 以下的 UID 预留给了 system accounts.

passwd 文件中的信息包括

* The login name
* The password for the user
* The numberical UID of the user account
* The numberical group ID(GID) of the user account
* A text description of the user account(called the comment field)
* The location of the HOME directory for the user
* The default shell for the user

password 字段为 x, 以前放置的还是加密后的 pwd 后来为了安全统一放到 /etc/shadow 下面去了

#### The /etc/shadow file

只有 root user 可以访问 shadow 文件

```sh
# docker bash 中的内容
cat /etc/shadow
# root:!::0:::::
# bin:!::0:::::

# 书上的例子
# rich:$1$.FfcK0ns$f1UgiyHQ25wrB/hykCn020:11627:0:99999:7:::
```

shadow 中的信息包括

* The login name corresponding to the login name in the /etc/passwd file
* The encrypted password
* The number of days since January 1, 1970, that the password was last changed
* The minimun number of days before the password can be changed
* The number of days before the password must be changed
* The number of days before the password expiration that the user is warned to change the password
* The number of days after a password expires before the account will be disabled
* The date(stored as the number of days since January 1, 1970) since the user account was disabled
* A filed reserved for feature use