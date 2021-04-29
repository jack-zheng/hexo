---
title: Hexo 换皮肤
date: 2019-11-15 13:42:26
categories:
- hexo
tags:
- theme
---
今天决定尝试一下给 blog 换皮肤。就拿时下最流行的 Next 主题好了，用的人多，文档齐全，熟悉了之后有需求再发挥。

### 安装过程

安装很简单，官方地址 [hexo-theme-next](https://github.com/theme-next/hexo-theme-next)。

1. `cd hexo`
2. `git clone https://github.com/theme-next/hexo-theme-next themes/next`
3. 到 `themes/next` 目录下把 repo 的 .git, .github 删掉
4. 起 server 验证，打完收工

其他一些比较个人的配置去 next 目录下的 `_config.yml` 里面配置，像什么头像啦，github 三角标什么的都要有的。

### 一些坑

如果你只是将 next clone 下来没有删掉 .git 就 add 的话会有 warning 给出来

```info
git add .gitignore _config.yml themes/warning: adding embedded git repository: themes/nexthint: You've added another git repository inside your current repository.
hint: Clones of the outer repository will not contain the contents of
hint: the embedded repository and will not know how to obtain it.
hint: If you meant to add a submodule, use:
hint:
hint:   git submodule add <url> themes/next
hint:
hint: If you added this path by mistake, you can remove it from the
hint: index with:
hint:
hint:   git rm --cached themes/next
hint:
hint: See "git help submodule" for more information.
```

这是应为 git 是不支持嵌套 repo 管理的，你可以通过 submodules 来管理，不过使用上会有点冲突，按 submodules 的定义来说，它是为那些需要使用子模块但是那些模块不需要更新，或者只需要跟着官方的 branch 走就行了。就 hexo 这种情况，你可以自己 fork 一个，然后作为子模块管理，但是这样你 fork 的 repo 就不能跟进官方的 repo 了 ┑(￣Д ￣)┍， 反正就我的情况来说，用最简单的删 .git 就行了，真有需求以后在研究。
