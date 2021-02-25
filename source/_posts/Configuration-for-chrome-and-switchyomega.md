---
title: Configuration for chrome and switchyomega
date: 2021-02-25 16:20:40
categories:
- 配置
tags:
- 代理
- Chrome
---

Chrome + SwitchyOmega 已经配置了无数遍了，抽出来单独写一篇详细教程

## 配置步骤

外网限制，安装 Chrome 扩展极度不便。所有的第一步是安装 SwithyOmega 的扩展来翻墙。

1. 去 [SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega) 官网下载 crx 文件。
2. 重命名，将后缀改为 zip，然后直接拖到浏览器 - **直接将 crx 文件拖进去会报错 `CRX_HEADER_INVALID`**
3. 左键点击插件 icon -> Options -> New profile 添加 Profile Proxy 类型配置文件
4. Protocol, Server, Port 一次填入 SOCKS5, 127.0.0.1, 1086
5. 选择 auto switch, `Rule List Config` 下选择 AutoProxy 类型
6. `Rule List URL` 填入 `https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt` 点击下载更新文件
7. Apply Change, 打完收工

PS: 公司里面有提供 PAC 解决方案，很方便，规则都给你定好了 New Profile 选择 PAC 类型，填入指定的 URL, 再点击 download 即可

## Resources

* [GFWList 配置教程 - 官方](https://github.com/FelisCatus/SwitchyOmega/wiki/GFWList)
* [GFW 规则主页](https://github.com/gfwlist/gfwlist)