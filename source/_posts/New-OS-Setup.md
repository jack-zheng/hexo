---
title: 新系统初始化设置
date: 2020-08-03 14:51:07
categories:
- 小工具
tags:
- 系统
---

记录一下新系统常用配置和软件安装

## Chrome

外网限制，安装 Chrome 扩展极度不便。所有的第一步是安装 SwithyOmega 的扩展来翻墙。

1. 去 [SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega) 官网下载 crx 文件。
2. 重命名，将后缀改为 zip，然后直接拖到浏览器

PS: 直接将 crx 文件拖进去会报错 `CRX_HEADER_INVALID`

## MacOS 显示隐藏文件

`cmd + shift + .`

## Homebrew

官网推荐的通过 curl raw 文件安装，本地没有 proxy 的话 pass，基本不动。可以直接通过 `git clone https://github.com/Homebrew/install` 这个 repo 然后 `cd` 到 install 文件夹下执行 `/bin/bash -c ./install.sh` 来触发任务

### 速度测试

运行如下命令，查看是哪个步骤速度比较慢

```bash
brew update --verbose
```

```bash
jack@PC /usr/local/Homebrew/stable brew update --verbose
Checking if we need to fetch /usr/local/Homebrew...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask...
Fetching /usr/local/Homebrew...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
Checking if we need to fetch /usr/local/Homebrew/Library/Taps/homebrew/homebrew-services...
Fetching /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
Fetching /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask...
remote: Counting objects: 5806, done.
remote: Compressing objects: 100% (2626/2626), done.
remote: Total 5806 (delta 4179), reused 4564 (delta 3087)
Receiving objects: 100% (5806/5806), 1.30 MiB | 203.00 KiB/s, done.
Resolving deltas: 100% (4179/4179), completed with 375 local objects.
From https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew
   7b67ac5e3..a4d7bb64a  master     -> origin/master
 * [new tag]             2.1.10     -> 2.1.10
 * [new tag]             2.1.11     -> 2.1.11
 * [new tag]             2.1.12     -> 2.1.12
 * [new tag]             2.1.13     -> 2.1.13
 * [new tag]             2.1.14     -> 2.1.14
 * [new tag]             2.1.15     -> 2.1.15
 * [new tag]             2.1.16     -> 2.1.16
 * [new tag]             2.1.3      -> 2.1.3
 * [new tag]             2.1.4      -> 2.1.4
 * [new tag]             2.1.5      -> 2.1.5
 * [new tag]             2.1.6      -> 2.1.6
 * [new tag]             2.1.7      -> 2.1.7
 * [new tag]             2.1.8      -> 2.1.8
 * [new tag]             2.1.9      -> 2.1.9
remote: Counting objects: 71830, done.
remote: Compressing objects: 100% (27226/27226), done.
remote: Total 71830 (delta 53303), reused 62922 (delta 44592)
Receiving objects: 100% (71830/71830), 21.95 MiB | 11.70 MiB/s, done.
Resolving deltas: 100% (53303/53303), completed with 4020 local objects.
From https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core
   ea056b500e..6ec9c907ea master     -> origin/master
remote: Enumerating objects: 235987, done.
remote: Counting objects: 100% (215906/215906), done.
remote: Compressing objects: 100% (58941/58941), done.
Receiving objects:  45% (92674/205138), 23.64 MiB | 345.00 KiB/s
```

### 更新 Brew 配置

```bash
# 替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# 替换homebrew-cask.git:
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# 替换 homebrew bottles 源, zsh 用户:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc

# 替换 homebrew bottles 源, bash 用户:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```

如果这个源挂了可以试试清华的

## update warning

brew update 抛 warning

```bash
Updating /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core...
fatal: It seems that there is already a rebase-apply directory, and
I wonder if you are in the middle of another rebase.  If that is the
case, please try
	git rebase (--continue | --abort | --skip)
If that is not the case, please
	rm -fr ".git/rebase-apply"
and run me again.  I am stopping in case you still have something
valuable there.
```

存在 `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/.git/rebase-apply` 这样的备份文件，通过 `rm -rf rebase-apply` 删掉就好了

### Reference

* [RayDBG](https://www.raydbg.com/2019/Homebrew-Update-Slow/)
* [USTC Guide](https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git)

## Iterm2

* 最大化窗口：CMD + Ctrl + F
* Item2 最大化终端：CMD + Enter

可以通过下载 [官方 zip](https://www.iterm2.com/downloads.html) 包离线安装，也可以通过 brew 安装 `brew cask install iterm2`。brew 会比较慢

修改提示符 

```bash
prompt_context () {
    prompt_segment black default "Jack"
}
```

### 配置 solarized 配色方案

最新版的系统已经默认支持这个配色方案了，打开 iterm2 终端，`cmd + ,` 打开配置窗口。 Preferences -> Profiles -> Colors -> Color Presets -> Solarized Dark

## oh-my-zsh

官方 [git](https://github.com/ohmyzsh/ohmyzsh) 地址， 应为网络原因选择 clone repo 安装: `git clone https://github.com/ohmyzsh/ohmyzsh.git` + `sh -c './install.sh`

重启后终端抛出 warning

```bash
Last login: Mon Aug  3 18:08:08 on ttys000
[oh-my-zsh] Insecure completion-dependent directories detected:
drwxrwxr-x  3 jack  admin   96 Aug  3 13:18 /usr/local/share/zsh
drwxrwxr-x  4 jack  admin  128 Aug  3 13:22 /usr/local/share/zsh/site-functions

[oh-my-zsh] For safety, we will not load completions from these directories until
[oh-my-zsh] you fix their permissions and ownership and restart zsh.
[oh-my-zsh] See the above list for directories with group or other writability.

[oh-my-zsh] To fix your permissions you can do so by disabling
[oh-my-zsh] the write permission of "group" and "others" and making sure that the
[oh-my-zsh] owner of these directories is either root or your current user.
[oh-my-zsh] The following command may help:
[oh-my-zsh]     compaudit | xargs chmod g-w,o-w

[oh-my-zsh] If the above didn't help or you want to skip the verification of
[oh-my-zsh] insecure directories you can set the variable ZSH_DISABLE_COMPFIX to
[oh-my-zsh] "true" before oh-my-zsh is sourced in your zshrc file.
```

运行如下 cmd 修复

```bash
chmod 755 /usr/local/share/zsh
chmod 755 /usr/local/share/zsh/site-functions
```

配置命令高亮: `brew install zsh-syntax-highlighting` 并在 .zshrc 中添加配置行 `source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh`

配置命令自动补全提示: `git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions` 添加 .zshrc 配置 `plugins=(zsh-autosuggestions)`

为提示插件绑定快捷键: 在 zshrc 文件中添加配置 `bindkey '^ ' autosuggest-accept`, MacOS 下这个快捷键和系统默认的输入法切换冲突，在 System Preferences -> keyboard -> shortcuts -> input sources 下将 select the previous input source 和 selet the next input soure menue 的勾选去掉就行了

PS: 这个快捷键在 VSCode 的 terminal 上不能 work, 试着把 vscode 自带的 `ctrl + space` 都改掉还是没效果(´Д`) 先凑合着用把，干

PPS: 想要重新绑定 `shift + space` 为补全，不过找不到对应的 zsh code， 擦擦擦。在 linux 下有款终端工具叫 showkey 的貌似可以解决这个问题， 也可以试试终端输入 cat 回车，按键他就会打印出来键符，不过 shift 貌似没给提示。。。

solarized dark 配色和 zsh-autosuggestion 自动提示配色有冲突，会看不到，参考 [issue](https://github.com/zsh-users/zsh-autosuggestions/issues/416#issuecomment-486516333)。我本地直接把配色改成系统自带的 Tango Dark 了

### 设置字体

使用 `agnoster` 主题时需要加载一个字体，不然很多箭头之类的表示符会显示乱码。下载字体：`git clone https://github.com/powerline/fonts.git`, 找到 `fonts/Meslo Slashed/Meslo LG M Regular for Powerline.ttf` 双击安装。 然后打开 iTerm2，按 `Command + ,` 键，打开 Preferences 配置界面，然后Profiles -> Text -> Font -> Chanage Font，选择 Meslo LG M Regular for Powerline 字体。

### VSCode

打开 VSCode, `CMD + SHIFT + P`, 选择 `Shell Command: Install 'code' command in PATH` 命令，应用会自动安装好，在终端输入 `code` 测试

配置完 zsh 之后，VSCode 的终端会显示乱码，`cmd + shift + p` 搜索 'Preferences: Open Settings(JSON)' 添加配置 `{ "terminal.integrated.fontFamily": "Meslo LG M for Powerline" }` 即可修复，保存后可以看到效果。

VSCode 在终端安装 code 命令之后每次重启都会失效，应该是因为我只是把它放在 Document, Download 文件夹下面了。把它放到 Application 下再安装一下 shell 继承命令就可以了。顺带着之前 `zsh-autosuggestion` 不能补全也是这个原因！！！

### 安装 VirtualBox

MacOS 10.15.6 Catalina 安装 VirtualBox 的时候报错，安装失败，是应为 MacOS 默认设置是禁止安装 Oracle 公司产品的，你可以去 System Preference -> Security & Privacy 页面点一下左下方的小锁，允许安装 Oracle 相关软件。再重新安装一下，就行了。

## 使用 IDEA 的快捷键时跳出窗口

窗口内容： "No manual entry for <command>"， Refer to [Official IDEA Support](https://intellij-support.jetbrains.com/hc/en-us/articles/360005137400-Cmd-Shift-A-hotkey-opens-Terminal-with-apropos-search-instead-of-the-Find-Action-dialog)

MacOS since 10.14, 官方定义了这个快捷键，和 IDEA 冲突了，Keyboard -> shortcut -> service -> search man page index in terminal 把这个选项 disable 掉，或者替换掉
