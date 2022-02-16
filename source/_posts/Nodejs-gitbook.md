---
title: Nodejs + gitbook
date: 2022-01-31 15:26:22
categories:
- Nodejs
tags:
- gitbook
---

基于 Nodejs 的 gitbook， 感觉和 hexo 大同小意，不过还有额外的制作 pdf 和 ebook 等功能，值得一试

## 安装

最终各软件版本信息如下

* node: v13.14.0
* CLI version: 2.3.2
* GitBook version: 3.2.3

```bash
# 安装
npm install -g gitbook-cli
# 检测
gitbook-cli -V
```

安装抛错

```bash
No receipt for 'com.apple.pkg.CLTools_Executables' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLILeo' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLI' found at '/'.

gyp: No Xcode or CLT version detected!
/usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
                 ^
```

重新安装 xcode-select, 去 app store 中搜索 xcode 并安装，挺花时间的，后年还要重新签一下协议。。。

折腾一阵子还是失败了，最后搜了下谁是 gitbook 使用的 polyfills.js 还是旧版的，没有跟新，可以直接去这个文件中，将饮用的语句删掉，或者更新对用的库文件即可，我用了后者

根据提示，打开文件，将该文件中的引用语句注释掉, 再试运行，正常

```js
// fs.stat = statFix(fs.stat)
// fs.fstat = statFix(fs.fstat)
// fs.lstat = statFix(fs.lstat)
```

`gitbook init` 又报错了

```bash
TypeError [ERR_INVALID_ARG_TYPE]: The "data" argument must be of type string or an instance of Buffer, TypedArray, or DataView. Received an instance of Promise
```

说是版本过高。。。。草了, 打算使用 nvm 降一下

```sh
# 然而 DNS 被污染了，并不能拿到对应的文件，直接下载过到本地就行了, 然后 bash install.sh 即可
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

source ~/.zshrc

# 查看安装结果，nvm 提示很完善
nvm --version 

nvm ls-remote

nvm install v13.14.0

nvm use v13.14.0
```

切换版本之后，运行正常, 我估计可能 nvm 之后前面的那些改 js 文件的操作都可以省了

## gitbook 使用

* [中文版](https://chrisniael.gitbooks.io/gitbook-documentation/content/)

1. 创建书本文件并 `gitbook init` 生成目录
2. `gitbook serve` 生成本地版电子书，访问 4000 端口阅读

常用规则

常用插件

* gitbook-plugin-summary 可以更具文件夹目录帮你自动生成对应的 summary 文件内容，安装完后输入 `book sm` 自动更新