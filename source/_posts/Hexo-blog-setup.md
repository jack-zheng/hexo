---
title: Hexo 博客 Setup 
date: 2019-11-12 17:58:11
categories:
- Hexo
tags: 
- setup
---
Hexo setup 笔记。网上有好多 setup 的教程，这里就不赘述了。记录一下我 setup 时候用到的命令，作为备忘。

这里使用的 Next 版本 **V7.5.0**

### Commands

1. 安装 node/npm, `brew install node`, type `node -v`, `npm -v` to check if install successfully.
1. run command: `npm install -g hexo-cli`, 安装 hexo 工具, 安装完成，type `hexo` to check
1. Setup 博客基础架构 `hexo init <folder>`, cd \<folder\>, run command: `hexo server` 就可以得到一个本地可访问的 hello world 博客模版
1. `hexo new post_name`, 在 source 文件夹下面会创建一个新的 post_name.md 文件作为新博客的载体
1. 为你的博客新建一个git repo, repo name 必须是**你的Git用户名.github.io**, 如果已经创建了, rename 一下
1. 编辑 \<folder\>\_config.yml 关联 git repo
1. `npm install hexo-deployer-git --save` 安装 git 集成工具
1. `hexo g` 生成工程目录及相关文件
2. `hexo s` 启动本地 server 验证
3. `hexo d` 部署发布到 github, 等一两分钟访问 `https://<你的Git用户名>.github.io` 就可以看到你的作品了 (^з^)-☆

```config
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/<username>/<username>.github.io.git
  # SSH 格式的也OK, 简单理解就是去 github repo 页面, 把你的 repo 地址复制一下
  branch: master
```

### 异地环境 setup

#### Win10

在其他机子上面重新 setup 环境只需要安装 git 和 nodejs, 把项目 clone 到本地之后 cd 到博客根目录下运行命令

```bash
npm install -g hexo-cli
```

就行了，在 Windows 下使用 VS Code 的默认命令行时还遇到另外一个问题，hexo 命令不能执行，抛出 Exception:

```bash
PS C:\Users\lanmo\gitStore\hexo> hexo
hexo : 无法加载文件 C:\Users\lanmo\AppData\Roaming\npm\hexo.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 https:/go.microsoft
.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
所在位置 行:1 字符: 1
+ hexo
+ ~~~~
    + CategoryInfo          : SecurityError: (:) []，PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

这是由于 powershell 的默认脚本执行策略把这个 command 阻塞了，可以执行

```bash
# 允许本地脚本执行
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

来开放权限，其他可用命令还有

```bash
# 查看可用策略
Get-ExecutionPolicy -List

# 查看当前策略
Get-ExecutionPolicy
```

更多可以参考官方文档 [About Execution Policies](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-6)

#### Win10 WSL

WSL 默认已经安装了 git, 所以只需要额外安装 nodejs 就行了。

```bash
sudo apt-get install nodejs

# 如果速度慢可以使用 taobao 源加速
npm --registry https://registry.npm.taobao.org install nodejs
# 配置永久源
npm config set registry https://registry.npm.taobao.org
```

安装完后运行 `node -v` 和 `npm -v` 查看是否安装成功。我本地安装完后，node 可以正常调用，但是 npm 不行，报错

```bash
jack@DESKTOP-9TGTFK1:~$ npm -v
: not foundram Files/nodejs/npm: 3: /mnt/c/Program Files/nodejs/npm:
: not foundram Files/nodejs/npm: 5: /mnt/c/Program Files/nodejs/npm:
/mnt/c/Program Files/nodejs/npm: 6: /mnt/c/Program Files/nodejs/npm: Syntax error: word unexpected (expecting "in")

# 运行 which npm 查看路径
jack@DESKTOP-9TGTFK1:~$ which npm
/usr/bin/npm
```

是应为路径有问题，修改 WSL 下的 `~/.profile` 文件，添加 npm 执行路径

```bash
PATH="$HOME/bin:$HOME/.local/bin:/usr/bin:$PATH"
```

然后 `source ~/.profile` 在运行 `npm -v`, 成功。

该问题可以参考 [VSCode Git Issue](https://github.com/microsoft/WSL/issues/1512)

### Reference

* [Hexo 官方文档](https://hexo.io/docs/github-pages)
* [Juejin-最全Hexo博客搭建+主题优化+插件配置+常用操作+错误分析](https://juejin.im/post/5bebfe51e51d45332a456de0)
* [Zhihu-GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
