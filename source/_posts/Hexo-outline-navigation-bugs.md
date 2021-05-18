---
title: Hexo outline navigation bugs
date: 2021-05-18 10:41:25
categories:
- Hexo
tags:
- 导航
- bug
---

发现一个 Hexo 的 bug. 使用版本

```txt
hexo version 
INFO  Validating config
hexo: 5.0.0
hexo-cli: 4.1.0
os: Darwin 20.4.0 darwin x64
node: 15.10.0
v8: 8.6.395.17-node.25
uv: 1.41.0
zlib: 1.2.11
brotli: 1.0.9
ares: 1.17.1
modules: 88
nghttp2: 1.42.0
napi: 7
llhttp: 2.1.3
openssl: 1.1.1j
cldr: 38.1
icu: 68.2
tz: 2020d
unicode: 13.0
```

问题描述：

Outline 中的中文导航会失败，英文的可以正常工作

![Outline display issue](outline.png)

怎么修还不清楚，简单搜索了一下并没有查到解决方案，有机会的话可以看看 Hexo 源码再提一个 fix 的 PR ╮(￣▽￣"")╭