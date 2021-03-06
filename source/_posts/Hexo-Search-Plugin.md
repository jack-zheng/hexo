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

某一天突然发现部分 Post 不能被 search 出来了。 看这个插件的配置信息，建立的索引应该是放在这个 search.xml 文件里了。排查发现是 Splunk 之后一个都失效了。继续排查，是这片文章中有个 'Steps:' 的节点，在编辑器里面查看是没什么问题的，但是贴到其他工具，比如 idea 或者 browser 里面时，他会带一个 [BS] 的前缀。太神奇了。。。所以之前一直没发现。

在 VSCode 里面看结构还是 `<p>Steps</p>` 但是用 linux cat 时就变成 `<pSteps:</p>` 所以后面的 search 解析就出问题了。查了下 BS 代表的是退格键 0x008, 也解释了为什么 xml p 标签会少一个尖括号了 ╮(￣▽￣"")╭

**2021-05-20**

Docker 弹射起步的那篇文章并没有建立索引，搜索不到，只能通过 tag 导航过去。稍微检查了一下，和之前的文章里面混入了特殊字符的问题还不一样，有待查证

搜索引擎使用技巧的那篇文章也不能搜索出来

**2021-06-02**

jmockit, testng 等关键字也没有建索引，感觉我的忍耐快要到极限了 (#ﾟДﾟ)

**2021-06-03**

本地启动 server，search 了一下，很多关键字都是可以找出来的。。。难道是远端的部署有问题？！

通过 F12 + network 的 tab, 点击 search 看网络加载。remote 的 search.xml 位于 `https://jack-zheng.github.io/hexo/search.xml` 这路径下。加载的时候 terminal 会报错，难道我找到了华点？！！

```txt
This page contains the following errors:
error on line 3463 at column 23: Input is not proper UTF-8, indicate encoding !
Bytes: 0x08 0x20 0xE5 0x88
Below is a rendering of the page up to the first error.
```

灯噔, 密码正确，删掉了对应的文件，search 正常了，但是我反复看了下那个文件，没看出什么问题啊。。。

终于找到问题所在了！！！关键字没有设置对，害我绕了老大一圈才找到答案。如果早点用关键字 `Hexo search.xml Input is not proper UTF-8` 的话，早就出结果了。之前一直纠结在 Chrome 等主题那边

用 hexo 插件等关键字发现几个和我一样的案例，很快的解决了问题。。。

解决步骤：在 vscode 中打开你的项目，点搜索，点正则匹配，根据你的世纪情况搜索你要的关键字。比如我报错是 `0x08 0x20 0xE5 0x88` 就直接搜索 `\x08`, 把搜索出来的地方都改了就行。

选中出问题的字段直接把它复制黏贴到终端，可以看到 `code, ^H如果是` 的显示，好神奇

![Search issue](search_issue.png)

又做了一些深入调查，这种字符好像是控制字符，可能是打字的时候无意间打出来的，你可以通过 vscdoe 里面的设置，将它默认显示出来

Code -> Perferences -> Settings -> 搜索 renderControlCharacters 勾选上即可
