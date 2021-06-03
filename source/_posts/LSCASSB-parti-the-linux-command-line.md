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



## Chapter 4: More bash Shell Commands

### Monitoring Programs

> Peeking at the processes

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

> Real-time process monitoring

`ps` 只能显示一个时间点的 process 状态，如果要实时显示，需要用到 `top` 命令

```sh
top
# PID    COMMAND      %CPU TIME     #TH    #WQ  #PORT MEM    PURG   CMPRS  PGRP  PPID STATE    BOOSTS          %CPU_ME %CPU_OTHRS UID  FAULTS     COW     MSGSENT    MSGRECV   SYSBSD     SYSMACH    CSW        PAGEIN IDLEW    POWE INSTRS    CYCLES    USER
# 1841   com.docker.h 36.7 68:12.08 13     0    37    19G    0B     629M   1710  1830 sleeping *0[1]           0.00000 0.00000    501  202373842+ 473     569        335       85444924+  920        48973831+  17     3949456+ 59.4 469486491 786452501 i306454
```

> Stopping processes

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

