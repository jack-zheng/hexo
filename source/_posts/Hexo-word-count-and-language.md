---
title: Hexo 添加字数显示，更改语言
date: 2019-11-19 17:58:31
categories:
- Hexo
tags:
- word count
- language
---
本篇包含两个配置

1. 文章字数，阅读时间显示
1. 语言设置，显示中文

## 配置字数

参靠 repo: [hexo-symbols-count-time](https://github.com/theme-next/hexo-symbols-count-time)

1. 到根目录下执行 `npm install hexo-symbols-count-time` 安装插件
1. 到根目录下的 _config.yml 中添加配置

```config
symbols_count_time:
  symbols: true
  time: true
  total_symbols: true
  total_time: true
  exclude_codeblock: false
```

## 配置中文显示

查看 Next 主题下面的 language 文件夹，找到其中的中文显示文件名，把根目录下的 _config.yml 里的 language 改为这个名字就行了。我这边文件名为 `zh-CN.yml`，将 yml 中 language 改为 `zh-CN`
