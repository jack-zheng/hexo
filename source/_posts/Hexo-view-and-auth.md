---
title: Hexo 设置阅读数，文章授权
date: 2019-11-20 18:57:02
categories:
- hexo
tags:
- auth
- view count
---
本篇将介绍如何设置统计文章阅读量和文章授权。

## 阅读量统计

Hexo 默认使用'不蒜子'做阅读量统计，而且已经配置好了，如果想要开启它只需要到 `next/_config.yml` 下将 `busuanzi_count:` 下的 `enable:` 设置为 true 即可。重启后访问也看可以看到文章标题下多处一只眼睛标志，旁边就是总阅读量。

[不蒜子](http://ibruce.info/2015/04/04/busuanzi/), 貌似是某程序员建的站，托管在七牛上的，赞！

## 文章授权

Hexo 默认授权是关闭的，可以在 `next/_config.yml` 的 `creative_commons` 模块做设置。默认是 `by-nc-sa` 授权。

[常见授权方式Wiki](https://zh.wikipedia.org/wiki/%E7%9F%A5%E8%AF%86%E5%85%B1%E4%BA%AB%E8%AE%B8%E5%8F%AF%E5%8D%8F%E8%AE%AE)

许可协议|简称
---|:--:|
创作共享 署名|CC BY
创作共享 署名-相同方式共享|CC BY-SA
创作共享 署名-非商业性|CC BY-NC
创作共享 署名-禁止演绎|CC BY-ND
创作共享 署名-非商业性-禁止演绎|CC BY-NC-ND
创作共享 署名-非商业性-相同方式共享|CC BY-NC-SA
创作共享 相同方式共享|CC SA
创作共享 非商业性|CC NC
创作共享 禁止演绎|CC ND
创作共享 非商业性-相同方式共享|CC NC-SA
创作共享 非商业性-禁止演绎|CC NC-ND
