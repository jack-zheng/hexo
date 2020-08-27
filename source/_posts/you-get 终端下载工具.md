---
title: you-get 终端下载工具
date: 2019-11-27 13:03:43
categories:
- 小工具
tags:
- 终端
- 下载
- python
---
作为一个新世纪的社会主义接班人，怎么能不关注国家大事呢，于是我开始有意识的培养看新闻联播的习惯。但是问题来了，新闻联播要三十分钟，而且不能倍速播放。都 9012 年了啊，官网用的还是 flash, 也是醉了。合计了一下，打算使用工具将视频下载下来后本地用 Potplayer 加速播放。一开始找了 IDM，总的来说，用起来还不错，但是有些时候新闻联播官网抓到的 ts 文件，还得自己合并，不开心 (｡ ́︿ ̀｡)

最后在 Gayhub 上找到了 you-get 很赞 ↖(^ω^)↗ 而且他还支持很多网站的下载 b站，youku 什么的都不在话下，而且有人维护，贡献很积极呦

* [Github you-get](https://github.com/soimort/you-get)

## 安装

Win10 OS

准备工作：安装 python 3.2+, FFmpeg。前者用来下载后者用来合并视频，如果 FFmpeg 没有安装的还，下载还能成功，但是合并就不行了。

```bash
pip install you-get
```

安装 FFmpeg, 前往[官网](https://ffmpeg.org/)下载压缩包，解压后将 bin 文件加到系统配置的 path 中，path 选系统 level 的，user level 的可能会出问题，视屏不能合并，别问我为什么知道的 ┑(￣Д ￣)┍

## 使用

```bash
# 显示可用的下载选项，然后根据提示操作就行了
you-get -i [URL]
```

官方文档写的很详细，不懂就看 you-get 官方文档好了，有[中文版](https://github.com/soimort/you-get/wiki/%E4%B8%AD%E6%96%87%E8%AF%B4%E6%98%8E)呦
