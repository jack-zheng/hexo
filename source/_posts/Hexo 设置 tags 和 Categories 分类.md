---
title: Hexo 设置 tags 和 Categories 分类
date: 2019-11-15 16:44:32
categories:
- 博客
tags:
- hexo
---
默认设置下 Hexo Next 主题是关闭 `tag` 和 `categories` 的，你可以通过一下步骤打开它。

1. 去到 next folder 下，打开 `_config.yml`, 去掉 menu 下的 tags 和 categories 的注释。此时刷新页面，主页上会在 Archive 旁边多两个icon.
1. repo 目录下 run command: `hexo new page categories`, 并向该文件中添加新行 `type: "categories"`。新文件目录 `path/to/blog/source/categories/index.md`
1. repo 目录下 run command: `hexo new page tags`, 并向该文件中添加新行 `type: "tags"`。新文件目录 `path/to/blog/source/tags/index.md`

```config
# categories index.md
---
title: categories
date: 2019-11-15 16:42:15
type: "categories"  <--- 新行
---
```

```config
# tags index.md
---
title: tags
date: 2019-11-15 16:29:40
type: "tags"  <--- 新行
---
```

顺便还可以去 `path/to/blog/scaffolds/post.md`，在 post.md 中添加新行 `categories:`， 这样每次 new post 的时候都会自动带上这个标签了 ♪(´ε｀ )

### Reference

* [linlif-Hexo](https://linlif.github.io/2017/05/27/Hexo%E4%BD%BF%E7%94%A8%E6%94%BB%E7%95%A5-%E6%B7%BB%E5%8A%A0%E5%88%86%E7%B1%BB%E5%8F%8A%E6%A0%87%E7%AD%BE/)
