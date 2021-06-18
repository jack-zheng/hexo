---
title: tar 命令使用简介
date: 2021-06-18 15:56:09
categories:
- Shell
tags:
- shell
- tar
---

tar 最开始是用来操作磁带设备的，后来用途越来越广。一开始 Linux 系统是不能同时压缩多个文件的，所以，通常需要用 tar 命令将文件打包成一个文件，然后在压缩

tar 的常用操作我归结为三类，分别是打包，解包和查看。打包和解包的基础上还可以加上压缩的操作。

## 打包

* -c, 对应的关键字是 create
* -f, 表明处理的是文件，不加会抛错
* -v, 可选，打印详细信息
* -z, 压缩

```sh
ls
case-id-mapping  create_ticket.sh sample_body

tar -cvf my.tar case-id-mapping create_ticket.sh sample_body
a case-id-mapping
a create_ticket.sh
a sample_body

ls -l
-rw-r--r--  1 i306454  staff  9728 Jun 18 16:41 my.tar

ar -zcvf my.tar case-id-mapping create_ticket.sh sample_body
a case-id-mapping
a create_ticket.sh
a sample_body

ls -l my.tar
-rw-r--r--  1 i306454  staff  2203 Jun 18 16:44 my.tar

tar -tvf my.tar
-rw-r--r--  0 i306454 staff    4123 Jun 18 16:35 case-id-mapping
-rwxr--r--  0 i306454 staff    1347 Jun 18 16:35 create_ticket.sh
-rw-r--r--  0 i306454 staff     857 Jun 18 16:35 sample_body
```

* 不加 -v 中间的命令是不会有 a xxx 的信息的
* 加上 -z 之后出现压缩效果
* 压缩的的 tar ball 还是可以通过 -t 查看
* -f 一定要在最后，不然会报错，这个可以从参数定义看出来，这个参数后面接你要处理的目标文件 cvfz 则表示你要处理 z 这个文件了
* 如果加了 -z 则最好将你的文件后缀改为 .gz
* 如果想要查看是否压缩过，使用 file 命令 `file my.tar` 返回 `my.tar: POSIX tar archive`

## 解包

* -x, 解包操作, 代表 extract 操作
* -z, 一开始还以为有用，但是测试下来，不需要加这个参数自动就解压缩了

```sh
tar -xvf my.tar                                              
x case-id-mapping
x create_ticket.sh
x sample_body
```

## 查看

之前的压缩已经使用过了，使用 -t 参数

```sh
tar -tvf my.tar
-rw-r--r--  0 i306454 staff    4123 Jun 18 16:35 case-id-mapping
-rwxr--r--  0 i306454 staff    1347 Jun 18 16:35 create_ticket.sh
-rw-r--r--  0 i306454 staff     857 Jun 18 16:35 sample_body
```

## 其他

其他还有一些参数

* -r, 追加
* -u, 如果有改动才追加

