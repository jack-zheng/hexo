---
title: Hexo 搜索
date: 2020-05-28 17:10:27
categories:
- 博客
tags:
- hexo
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
