---
title: Linux 命令小贴士
date: 2020-07-08 18:49:48
categories:
- linux
tags:
- cmd
---

常用 Linux command 备忘录

## sh Vs bash

`sh` 是一种协议 shell command language. 而 `/bin/sh` 和 `/bin/bash` 是对他的两种不同的实现, 早起他们基本是一致的，但是随着 bash 的发展，他们变得不兼容起来。`/bin/sh` 还是标准，`/bin/bash` 则效率更高

## 快捷键

* `ctr + w` 光标处开始删除一个 word
* `ctr + /`, `ctr + _` 撤销删除，具体细节有所不同，但是都能达到目的

## 查看当前目录下文件最近修改时间

两种方式，一种是通过 `ls --full-time` 显示

```sh
ls --full-time
total 60
drwxr-xr-x    2 root     root          4096 2021-04-14 10:24:04 +0000 srv
dr-xr-xr-x   13 root     root             0 2021-07-12 02:52:11 +0000 sys
drwxr-xr-x    2 root     root          4096 2021-07-12 02:53:15 +0000 test
drwxrwxrwt    1 root     root          4096 2021-05-04 17:21:05 +0000 tmp
```

另一种是 `stat file_name`

```sh
stat test
  File: test
  Size: 4096      	Blocks: 8          IO Block: 4096   directory
Device: bbh/187d	Inode: 1055968     Links: 2
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2021-07-12 02:53:32.000000000
Modify: 2021-07-12 02:53:15.000000000
Change: 2021-07-12 02:53:15.000000000
```

## tee

查看信息的同时做写入操作

```sh
ps  | tee info.log      # ps 输出进程信息的同时，将结果导入 info.log 中
#   PID TTY           TIME CMD
# 23438 ttys000    0:59.10 /bin/zsh -l
# 48670 ttys002    0:01.91 /bin/zsh --login -i
# 71565 ttys003    0:02.87 -zsh
# 72395 ttys003    0:00.00 tee info.log
cat info.log            # 查看文本信息
#   PID TTY           TIME CMD
# 23438 ttys000    0:59.10 /bin/zsh -l
# 48670 ttys002    0:01.91 /bin/zsh --login -i
# 71565 ttys003    0:02.87 -zsh
# 72395 ttys003    0:00.00 tee info.log
```

## 重定向

```txt
<: 输入重定向
>: 输出重定向
<<: 截取标准输入
>>: 输出重定向，追加，不覆盖
EOF: 自定义终止符
```

示例：

```bash
# 只能在一条命令中完成，文本过长会很累赘
echo "eeeeecho" >> echo.txt

# 将 aaa 写入 a.txt
cat << EOF > a.txt
aaa
EOF

# a.txt 中追加 bbb
cat << EOF >> a.txt
bbb
EOF

# a.txt 拷贝到 b.txt
cat a.txt > b.txt

# 和 cat 类似不过它还有附带显示内容的效果
# tee << EOF > d.txt 会将显示部分吞掉，文件倒是还是生产
tee c.txt << EOF
ccc
EOF
```

## curl

终端获取资源，Sample: `curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python`

-s: 静默模式，去掉显示进度等信息
-S: 显示错误信息
-L: 自动站点跳转

将 query 结果存到本地文件

```bash
curl url >> ret.json
```

## ping

`ping` 命令不需要带 protocal，如果要指定端口可以加 `-p`

```bash
ping -p 8089 cloudsearch-dc8.cld.ondemand.com
```

## 容量查询

```bash
# 显示系统容量
df -hl

# 显示当前目录下个文件夹大小
du -sh *

# 显示文件大小并倒序排列
du -sh * | sort -hr
```

## ps 命令保留表头

```bash
# 这个命令不是很好，比较繁琐，效率也不高。
# 实现方式是先 ps 一下拿到 head 打印出来，再 ps 一次拿到我们想要的结果
ps | head -1; ps | grep java
```
## 查看文件/夹大小

```bash
# du: disk usage
du -sh *
```

## 链接 SFTP

建立联接

```shell
$ sfpt username@1.1.1.1 # 回车输入密码
```

获取文件下载到指定路径

```shell
sftp> get /export/sftp/test.csv /Users/my/Downloads
Fetching /export/sftp/test.csv to /Users/my/Downloads/test.csv
/export/sftp/test.csv            100%  133     0.3KB/s   00:00
```

上传本地文件到服务器指定路径

```shell
sftp> put /Users/my/Downloads/re-produce.gif /export/sftp
Uploading /Users/my/Downloads/re-produce.gif to /export/sftp/re-produce.gif
/Users/my/Downloads/re-produce.gif            100%  257KB  86.6KB/s   00:02
```

## 统计文件

* 当前目录下的文件个数，不包含文件夹 `ls -l | grep '^-' | wc -l`
* 当前目录下的文件个数，递归 `ls -lR | grep '^-' | wc -l`
* 当前目录下的文件夹个数 `ls -l | grep '^d' | wc -l`

解释：

* `ls -l`: 显示当前目录下所有文件，文件+文件夹
* `grep '^-'`: 删选文件，`grep '^-'` 筛选文件夹。 示例 `-rw-r--r--    1 jack  staff     1061 Aug  3 16:53 LICENSE`
* `wc -l`: 统计行数
