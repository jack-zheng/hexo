---
title: DBeaver 设置国内 Maven 源
date: 2020-08-18 15:50:50
categories:
- 小工具
tags:
- DBeaver
---

打开 DBeaver -> Preferences -> 搜索 Maven -> Add, 填入信息 `http://maven.aliyun.com/nexus/content/groups/public/`，调整一下顺序，放到第一位。打完收工～

PS: 设置只有在新建 connection 时生效，所以已经创建的，删了重建即可
