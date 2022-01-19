---
title: Webssh2 quick start
date: 2021-12-13 15:49:18
categories:
- Webssh
tags:
- node
- web socket
- ssh
---

搭建 Webssh2 实验平台，熟悉工具使用模式，等后期试着拆解一下这个项目的细节，用到自己的小项目上。感觉道路有点崎岖。。。。

这个东西本地部分挺简单的，clone 下来之后直接启动就行了，但是作为目标对象，需要一个测试用虚拟机，打算用 docker 起一个

## Docker 测试对象创建

之前还打算用 Centos 镜像自己搞一个的，结果各种换源出问题，然后找找 Ubuntu 的资源，直接发现一个配置好的镜像资源，6 啊，直接拿来用，香

```bash
# 作者在官方镜像基础上搭建的 https://hub.docker.com/r/rastasheep/ubuntu-sshd/
docker pull rastasheep/ubuntu-sshd

docker run -d -P --name test_sshd rastasheep/ubuntu-sshd

# 显示本地映射的端口
docker port test_sshd 22
# 0.0.0.0:55000

# 本地测试，默认密码 root, 成功
ssh root@localhost -p 55000
```

## webssh2 测试

```bash
git clone git@github.com:billchurch/webssh2.git

cd webssh2/app

npm install --production

npm start
```

然后开启 browser 并访问 http://localhost:2222/ssh/host/127.0.0.1?port=55000&header=My%20Header&headerBackground=red 即可访问网站。而且貌似还能记住密码，登陆一次之后就不需要再次输入了
