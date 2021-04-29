---
title: Hexo 保存博客源码
date: 2019-11-13 13:29:29
categories:
- hexo
tags: 
- travis
---
按照之前的教程，虽然 github page 上顺利发布了，但是 blog 的 source code 并没有一起同步过去，还在本地。Hexo 工具只是把你翻译之后的 web 信息同步过去了。search 了一下，想要同步 source 有大概三种路子：

1. hexo 插件: hexo-git-backup
2. 在原来的 blog repo 里面新建分支存储
3. 官方方法，集成 Travis CI，每次 push 自动帮你部署

本文只描述怎么集成 Travis CI, 其他的方案有机会再补，网上教程很多，随便找找就有了。
> 采用 Travis CI 的方案之后，原来的 repo name 需要改变，不然 blog 访问不了, 针对这种情况，其实还有另一种解决方案，将 master 用来存放编译好的 blog, 源码用新的 branch 存放，和前面的那些原理都一样

### Travis CI 集成

1. 新建一个 repo, 这里我用 hexo 作为 repo name
1. clone 到本地，将之前的 blog source copy 进去，添加 `.gitignore` 文件，把 `public\` 添加进 list
1. 注册 Travis 账号，用 github 授权就行，授权 repo, 添加 token. 官方文档都有链接，很方便
1. update `_config.yml` 里的 url 和 root 值
1. 添加 `.travis.yml` 做 CI 集成管理
1. commit + push 这些改动，Travis 自动 build 就会被触发了
1. build 完成后，repo 会有一个新的 branch 叫 gh-pages. 访问 `https://jack-zheng.github.io/hexo` 查看改动结果

国内，第一次访问会比较慢，cache 了文件之后后面访问会快一点

```yml
# _config.yml setting
url: https://jack-zheng.github.io/hexo
root: /hexo/
```

```.gitignore
#.gitignore
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
.vscode
```

```yml
# .travis.yml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
```

### Issue Trace

在按照官方教程走完流程后，repo 的 setting page 会有如下 Warning, 删了 `themes/landscape/README.md` 这个文件再 build 一下就行了

```text
Your site is having problems building: The tag fancybox on line 77 in themes/landscape/README.md is not a recognized Liquid tag. For more information, see https://help.github.com/en/github/working-with-github-pages/troubleshooting-jekyll-build-errors-for-github-pages-sites#unknown-tag-error.
```

### Reference

[Travis CI 集成 - 官方](https://hexo.io/docs/github-pages)
