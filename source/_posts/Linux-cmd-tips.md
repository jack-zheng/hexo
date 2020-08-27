---
title: Linux cmd 小贴士
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