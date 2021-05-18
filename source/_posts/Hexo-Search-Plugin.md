---
title: Hexo 搜索
date: 2020-05-28 17:10:27
categories:
- Hexo
tags:
- search
---
Hexo 提供全区搜索功能很方便，在两个 `_config.yml` 文件下添加配置就行了，一个在 hexo 下，一个在 next 皮肤下。

root -> _config.yml 添加配置

```yml
# Config for search service
search:
    path: search.xml
    field: post
    format: html
    limit: 10000
```

root -> themes -> next -> _config.yml

```yml
local_search:
  enable: true
```

## Issues

某一天突然发现部分 Post 不能被 search 出来了，排查了好久，发现是 Splunk 之后一个都失效了。继续排查，是这片文章中有个 'Steps:' 的节点，在编辑器里面查看是没什么问题的，但是贴到其他工具，比如 idea 或者 browser 里面时，他会带一个 [BS] 的前缀。太神奇了。。。所以之前一直没发现。

在 VSCode 里面看结构还是 `<p>Steps</p>` 但是用 linux cat 时就变成 `<pSteps:</p>` 所以后面的 search 解析就出问题了。查了下 BS 代表的是退格键 0x008, 也解释了为什么 xml p 标签会少一个尖括号了 ╮(￣▽￣"")╭
