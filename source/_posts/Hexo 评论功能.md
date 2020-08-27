---
title: Hexo 评论功能
date: 2019-11-18 13:55:13
categories:
- 博客
tags:
- hexo
---
为博客增加评论功能，参考 next 的配置文件，截止 2019-11-18 号为止，next 已经默认支持了 changyan | disqus | disqusjs | gitalk | livere | valine 这些评论系统。这里出于兼容性和可靠性的原则，选择 gitalk 作为评论系统。

Steps:

1. github 创建一个新的 repo 用于存放 comments，比如叫做 hexo-comments
1. 去到 github 账号的配置页面新建一个 Oauth 授权，[点这里快速跳转](https://github.com/settings/applications/new)
1. 填写授权信息 Homepage URL 和 Authorization callback URL 都写自己的博客地址就行了
1. 确认后跳转到授权信息页面，记下他的 app id 和 secret
1. 配置 next 的 _config.yml 如下
1. 提交代码测试

测试评论，成功。新添加的评论会出现在 hexo-comments 的 issues tab 下面，按这样的操作的话，我觉的可能都不需要自建创建 comments repo 了，直接放在一个 repo 下面就完事了

```config
gitalk:
  enable: true
  github_id: jack-zheng # GitHub repo owner
  repo: hexo-comments # 新建的用于存放评论的repo
  client_id: d44xxxxxxe3a # GitHub Application Client ID
  client_secret: 9b3c4xxxxxb708ef # GitHub Application Client Secret
  admin_user: jack-zheng # GitHub repo owner and collaborators, only these guys can initialize gitHub issues
  distraction_free_mode: true # Facebook-like distraction free mode
  # Gitalk's display language depends on user's browser or system environment
  # If you want everyone visiting your site to see a uniform language, you can set a force language value
  # Available values: en | es-ES | fr | ru | zh-CN | zh-TW
  language:
```

PS: 用这种方案的话，默认只有 github 的用户才能评论，不过看这种文章的应该都是github用户，所以问题不大
PPS: 网上很多文章都会要你去配置 swig 文件，最新版的 next 已经不需要这个步骤了

### Reference

* [gitalk](https://github.com/gitalk/gitalk)
* [简书-Jonzzs](https://www.jianshu.com/p/b5f509f25872)
