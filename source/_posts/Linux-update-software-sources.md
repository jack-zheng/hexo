---
title: Linux 换源
date: 2019-12-02 22:27:18
categories:
- linux
tags:
- 换源
---
Linux 配置国内源加速，以 Ubuntu 为例子

## Steps

运行 command

```bash
# 替换 sources.list 中的源信息
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list

# 更新索引
sudo apt-get update
```

如果配置不生效，查看 sources.list 文件中的源信息，可能不是 `archive.ubuntu.com` 所以更新失败，比如我的 WSL环境中，原始的源就 `security.ubuntu.com` 需要把上面的命令改为, 使之生效

```bash
sudo sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```

## Reference

* [Ubuntu 源使用帮助](https://mirrors.ustc.edu.cn/help/ubuntu.html)
