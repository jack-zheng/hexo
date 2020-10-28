---
title: Git 小贴士
date: 2019-11-15 16:35:00
categories:
- 小工具
tags:
- git
---

## 怎么添加 commited file 到 `.gitignore` 中

> [StackOverflow: applying-gitignore-to-committed-files](https://stackoverflow.com/questions/7527982/applying-gitignore-to-committed-files/7528016)

```bash
git rm --cached path/to/file
```

## 怎么把local master branch 还原成和 remote 端一致

> [StackOverflow: reset-local-repository-branch-to-be-just-like-remote-repository-head](https://stackoverflow.com/questions/1628088/reset-local-repository-branch-to-be-just-like-remote-repository-head)

```bash
git fetch origin
git reset --hard origin/master
```

## 移除本地的 commit

```git
git reset HEAD~1
```

## 将 remote 会滚到上个 commit

StackOverflow 上都说 用 reset hard -- commit 来做，实践了一下并不好用，不能创建 PR。。。不知道是不是公司的版本有定制过，最后使用 `git checkout commit .` 完成了重制。

但是这里有个缺陷 checkout 并不能检测出新建的文件，所以还是有一些手工操作的，不知道有没有更好的办法

## 将本地的单个文件还原成 master 版本

```bash
git checkout origin/master -- /path/to/file
```

## 将 git add, commit 合并到一个命令中

> [StackOverflow: git-add-and-commit-in-one-command](https://stackoverflow.com/questions/4298960/git-add-and-commit-in-one-command)

```bash
# config git alias
git config --global alias.add-commit '!git add -A && git commit'

# and use it with
git add-commit -m 'My commit message'
```

## Rename local repo

```bash
git checkout <repo need to re-name>
git branch -m <new name>

# or make sure you are not at renamed repo
git branch -m <old repo name> <new repo name>
```

## 移除 merge 内容

```bash
git reset --hard HEAD

# or
git merge --abort
```

## 国内 git clone 有时会卡住, 有没有 debug 的选项

> [StackOverflow: how-can-i-debug-git-git-shell-related-problems](https://stackoverflow.com/questions/6178401/how-can-i-debug-git-git-shell-related-problems)

```conf
GIT_CURL_VERBOSE=1 GIT_TRACE=1 git pull origin master
```

## 已创建 repo 添加证书

> 跳转到项目页面，添加文件 'create a new file' -> 输入 'license' 会给出提示

## 查看某人的 commit 记录

```bash
git log --author='jack'
```

## 查看 log 反序

```bash
git log --reverse
```

## 对比文件

```bash
git diff <base-commit> <changed-commit> -- <file-path>
```

比如我像比较 8ab244e3b2de31ca 相对于 f31762ada1764 有什么改动可以使用

```bash
git diff f31762ada1764 8ab244e3b2de31ca -- <file-path>

# 如果是相对于 header 的改动，可以省略第一个 commit 内容
```

## 查看被删除文件的历史记录

```bash
git log -- <file path>
```

## 查看文件某一行删除记录

```bash
# -G 直接支持正则，-S 需要添加其他参数来支持正则
git log -S/G'key' /path/to/file
```

## 已经 check in 的文件夹加入 .gitignore

```bash
# 处理文件夹
git rm -r --cached /folder

# 处理文件
git rm --cached /path/to/file
```

## 显示 repo 关联的远端地址

```bash
git remote show origin
```

## cherry pick 提取某一个 commit 和并到目标分支

情景描述：

我自己又一个分之 A， 同时创建了另一个分之 B 并在上面做了改动，commit 为 c1。我对他的这个 commit 有依赖，又不想自己 CV 代码或者以后有 conflict 什么的，这时可以 checkout 到我自己的分支，然后 `git cherry pick c1` 来合并代码。他的代码 merge 之后我也不用解决冲突，美滋滋儿。

## 使用 rebase 来合并自己分支的 commit 记录，强迫症福音

check out 一个测试 branch，修改 readme

```bash
echo 'a' >> README.md
git add-commit -m 'edit01'
echo 'b' >> README.md
git add-commit -m 'edit01'
echo 'c' >> README.md
git add-commit -m 'edit01'
```

git log --oneline 查看 commit 记录

```bash
dc0a087 (HEAD -> testrebase2) edit03
c6feb2a edit02
0436650 edit01
0dcdaac init porject
```

现在通过 rebase 将 edit01-03 合并为一个 commit。这里有一个地方要注意的是如果指定 commit id，start point 是你想要合并的 ID 的前一个

```bash
git rebase -i HEAD~3 或者 git rebase -i 0dcdaac
```

terminal 给出提示

```bash
pick 0436650 edit01
pick c6feb2a edit02
pick dc0a087 edit03

# Rebase 0dcdaac..dc0a087 onto 0dcdaac (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
```

将 edit02, 03 的前缀改成 s，然后 :wq 进入下一个界面需改 commit message

```bash
pick 0436650 edit01
s c6feb2a edit02
s dc0a087 edit03
```

提示信息如下

```bash
# This is a combination of 3 commits.
# This is the 1st commit message:

edit01

# This is the commit message #2:

edit02

# This is the commit message #3:

edit03

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Jul 8 17:10:52 2020 +0800
#
# interactive rebase in progress; onto 0dcdaac
# Last commands done (3 commands done):
```

带 # 号的行不会显示，只需要修改之前我们自己添加的那些行就行了，这里修改为

```bash
merge commit edit01-03

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Wed Jul 8 17:10:52 2020 +0800
#
# interactive rebase in progress; onto 0dcdaac
# Last commands done (3 commands done):
#    squash c6feb2a edit02
#    squash dc0a087 edit03
# No commands remaining.
# You are currently rebasing branch 'testrebase2' on '0dcdaac'.
#
# Changes to be committed:
#   modified:   README.md
#
```

esc + :wq 退出，终端会给出修改成功的提示

```bash
[detached HEAD 6c42812] merge commit edit01-03
 Date: Wed Jul 8 17:10:52 2020 +0800
 1 file changed, 3 insertions(+)
Successfully rebased and updated refs/heads/testrebase2.
```

这是再使用 git log --oneline 查看，可以发现目标 commit 已经合并成功

```bash
6c42812 (HEAD -> testrebase2) merge commit edit01-03
0dcdaac init porject
```

## 怎么避免 branch 上出现很多 merge 的 commit, 强迫症福音 2.0

TODO

## Git SS 加速

修改 .gitconfig 文件，添加配置如下

```gitconfig
# config your `~/.gitconfig` file
[http]
proxy = socks5://127.0.0.1:1080
sslVerify = false

[https]
proxy = socks5://127.0.0.1:1080
```

或者在终端输入

```bash
# or you can config it by typing terminal
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
git config --global http.sslVerify false

# turn off proxy, 开启后 git commit 会受影响
git config --global --unset http.proxy
git config --global --unset https.proxy
```
