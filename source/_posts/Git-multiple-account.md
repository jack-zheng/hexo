---
title: Git 多账户设置
date: 2021-08-17 12:56:20
categories:
- 小工具
tags:
- git
---

公司使用 GitHub 企业版管理代码，我自己也时常会需要在公用版 Github 上更新一些代码，经常要交互使用。2021-08-13 的时候，GitHub 官方静止了 password 类型提交代码，这不只能找找怎么本地配置双账号了，解决方案如下

前置，初始化配置：

1. 删除 ～/.gitconfig 中的账户信息
2. 删除 ～/.ssh 下的配置信息

配置：

1. cd 到 ～/.ssh 目录下运行 `ssh-keygen -t rsa -C "your_email@example.com"` 生成公私密码，之前参考了官方文档指定类型 ed25519, 不知道为啥，配置不生效
2. 在提示 `Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]` 时添加后缀指定环境, 我本地的配置情况 `id_rsa_github id_rsa_github.pub id_rsa_sap id_rsa_sap.pub`
3. 后面直接回车到文件生成成功。
4. 运行 `eval "$(ssh-agent -s)"` 启动代理并运行 `ssh-add ~/.ssh/id_rsa_github` 和 `ssh-add ~/.ssh/id_rsa_sap` 将私钥添加到密钥链中。通过 `ssh-add -l` 可以查看已经添加的 key 信息. PS: 如果你是 Mac 系统需要加一个 -K 不然重启之后 key 就没了 `ssh-add -K ~/.ssh/id_rsa_github`
5. 在 ~/.ssh/config 文件中配置账号，网站的对应关系, 示例如下
6. 打开 github 网站，将 id_rsa_github.pub 这个公钥信息添加到目标网站 头像 -> Settings -> SSH and GPG keys ->  New SSH key
7. 测试配置是否成功，终端输入 `ssh -T git@github.com` 和 `ssh -T git@github.wdf.sap.corp` 看提示信息是否正确
8. 配置 commit 账号，如果没有设置，commit 的时候, git 会用电脑主机号作为提交账号。我本地是将常用的公司账号信息通过 git config --global 配置为全局信息，在自己的 repo 中通过 git config --local 单独配置

```txt
Host github
HostName github.com
User jack-zheng
IdentityFile ~/.ssh/id_rsa_github

Host gitlab
HostName github.wdf.sap.corp
User Ixxxx
IdentityFile ~/.ssh/id_rsa_sap
```

## 参考

* [官方文档](https://docs.github.com/cn/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
