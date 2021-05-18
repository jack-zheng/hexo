---
title: Hexo 中插入 UML
date: 2021-04-29 18:41:22
categories:
- Hexo
tags:
- uml
---

## 安装

1. 安装插件 `npm install hexo-tag-plantuml --save`
2. `_config.yml` 中添加配置

```yml
tag_plantuml:
	type: static
```

UML 测试

{% plantuml %}
    Bob->Alice : hello
{% endplantuml %}

## 参考

* [插件地址](https://github.com/two/hexo-tag-plantuml)