---
title: Idea 破解
date: 2019-12-26 21:36:14
categories:
- 配置
tags:
- idea
- 破解
---
简单记录一下怎么破解 idea, 主要是记录下破解的文章引用，方便以后查找，引用的文章 po 主说会持续跟新的 (～￣▽￣)～

## Steps

> PS: 预算充足的一定要支持正版啊啊啊啊 (●'◡'●)

1. 去官网下载最新的 Pro 版
1. 下载 JetbrainsCrack.jar 破解包，放到 idea 安装路径的 bin 文件夹下
1. 打开安装好的 idea，选择试用 30 天。 进入界面之后 Help -> Edit Custom VM Options, 如果提示是否创建文件，选择 Yes
1. 拿到刚刚的 jar 文件的绝对路径，添加到末尾，比如我这里是：`-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2019.2.4\bin\JetbrainsCrack.jar`
1. 重启 idea, 再到 Help -> Register, 选择 License server 方式，idea 会自动填入 `http://jetbrains-license-server`，确定
1. 在重启一波，根据提示信息可以看到破解完成

## Refer

* [感谢给出资源和解决方案的 - AlgerFan](https://www.algerfan.cn/articles/2019/03/06/1551868940012.html)
