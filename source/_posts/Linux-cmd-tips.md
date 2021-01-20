---
title: Linux 命令小贴士
date: 2020-07-08 18:49:48
categories:
- 配置
tags:
- linux
- cmd
---

常用 Linux command 备忘录

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
* 当前目录下的文件个数，递归 `ls -l | grep '^-' | wc -l`
* 当前目录下的文件夹个数 `ls -l | grep '^d' | wc -l`

解释：

* `ls -l`: 显示当前目录下所有文件，文件+文件夹
* `grep '^-'`: 删选文件，`grep '^-'` 筛选文件夹。 示例 `-rw-r--r--    1 jack  staff     1061 Aug  3 16:53 LICENSE`
* `wc -l`: 统计行数
