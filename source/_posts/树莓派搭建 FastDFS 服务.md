---
title: 树莓派搭建 FastDFS 服务
date: 2020-03-08 17:28:09
categories:
- 配置
tags:
- pi
- FastDFS
---
树莓派搭建 FastDFS 服务记录

小结写在最前面：此方案可行，稍微有点坑，但是踩踩还是可以过的

## 小知识

FastDFS 分布式文件存储系统，阿里系程序员**余庆**开发完成

FastDFS 只负责存储，不提供web支持，所以一般要搭配 nginx 使用

工作模式： client -> (tracker server + storage server), tracker 信息负责管理， storage 负责存储

树莓派的默认账号： pi | raspberry

可以通过终端输入: ssh pi@host 链接

## 搭建步骤

* [官方文档](https://github.com/happyfish100/fastdfs/wiki)
* [视频教程，不过要翻墙](https://www.youtube.com/watch?v=6Y2NihvPijQ)

依赖关系：

```tree
.
├── FastDFS-nginx-module
├── fastdfs(tracker + storage)
│   ├── GCC
│   ├── libevent
│   └── perl
├── libfastcommon
└── nginx
    ├── pcre-devel
    └── zlib-devel
```

### 一些坑

一些包的名字在 RedHat 和 Debain 下的名字不一样

```bash
apt-get install git gcc make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim

E: Unable to locate package gcc-c++
E: Unable to locate package pcre
E: Unable to locate package pcre-devel
E: Unable to locate package zlib
E: Unable to locate package zlib-devel
E: Unable to locate package openssl-devel

gcc-c++ 可以用 g++ 代替，不过好像 gcc 安装之后自动就装好了

pcre，pcre-devel 用 libpcre3 libpcre3-dev 代替

zlib： zlib1g
openssl-devel: libssl-dev
 ```

按照教程走，输入 ./make.sh && ./make.sh install 还是报错，说 permission denied. 试试 `sudo -i`

> 启动 nginx:

1. cd 到 root@raspberrypi:/usr/local/nginx/sbin# 路径下运行 ./nginx
2. 如果是重启就 ./nginx -s reload

> wget 下载文件并重命名:

wget -c 'url' -O 'rename'

> 查看已经上传的图片  

根据 storage 配置找到路径，各种 cd 进去可以看到你已经上传的文件，我本地测试的时候是在这里 `/home/dfs/data/00/00`

```bash
➜  00 ls -al
total 1072
drwxr-xr-x   2 root root    4096 Mar 14 15:10 .
drwxr-xr-x 258 root root    4096 Mar  8 11:38 ..
-rw-r--r--   1 root root 1024694 Mar  8 11:44 wKgBal5k2qGAQv4CAA-itrfn0m4.tar.gz
-rw-r--r--   1 root root   20002 Mar  8 12:52 wKgBal5k6pmAa4jlAABOIgA5mas34.jpeg
-rw-r--r--   1 root root   20002 Mar  8 15:41 wKgBal5lEiyAfaT6AABOIgA5mas88.jpeg
-rw-r--r--   1 root root   20002 Mar 14 15:10 wKgBal5s8_eAYY_fAABOIgA5mas81.jpeg
```

> 测试图片访问可以在 nginx 启动之后访问 host:8888/group1/M00/00/00/wKgBal5s8_eAYY_fAABOIgA5mas81.jpeg 这样的路径查看
