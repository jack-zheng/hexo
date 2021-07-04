---
title: VSCode 小贴士
date: 2019-12-14 12:51:34
categories:
- 小工具
tags:
- vscode
- 快捷键
- 小知识
- 插件
---
记录一些 VSCode 常用快捷键和使用技巧，提高工作效率 (´▽｀)

## 快捷键

* 跳转到定义：CMD + 键盘单击
* 从定义返回：Ctr + _ 或者  Option + CMD + 方向键
* 快速到顶部/底部：CMD + 上方向/下方向
* Ctrl + g: 快速跳到 x 行

## 很酷的操作

> 批量修改字符串，比如第 1，3，5 行 'est' 关键字前添加 'T', 即多光标操作

1. `option + 鼠标左键` 自定义操作锚点
2. `cmd + d` 向下选中相同的部分
3. 选中行 `shift + option + i` 统一相对为止操作

* [很棒的 VSCode 文档](https://geek-docs.com/vscode/vscode-tutorials/vs-code-multi-cursor.html)

> 在 VSCode 中复制代码并黏贴到 Outlook 等客户端时，会把背景颜色也黏贴过去，可以通过如下设置避免

```config
Preference -> settings, 搜索关键字 editor.copyWithSyntaxHighlighting 然后 disable 就行了
```

> 在 VSCode 中写 markdown 时，段落过长会自动换行，有表格的时候就很难看。可以 ctrl + p 然后搜索 word wrap 关闭换行即可

## 插件

* Ascii Tree Generator: 快速生产 Ascii 类型的目录树，在写文档的时候很游泳，喜欢 (´▽｀)
* Markdown All in One: 他的 format 功能简直太赞了！
* VSCode Icons: 为目录树上中的文件添加类型图标
* Bracket Pair Colorizer: 括号色彩标识
* rainbow csv: CSV 文件色彩标识

### PlantUML

此插件用于生成 UML，甚好（≧∇≦）

1. 打开 VSCode, 在插件列表中搜索 PlantUML 并安装
2. 设置 render server, 有两个可选项，一个是用官方的，另一个是用 local 的
   1. local 的话，可以参照 [Git project](https://github.com/plantuml/plantuml-server) 安装 docker 版本的
   2. `cmd + ,` 打开配置页，搜索 plantuml, 更具实际情况，修改 render 和 server 的配置，都有提示的
   3. 如果用的 local 的 server，还需要改一个配置，这个配置会在生产图片后通过 popup 的形式提示，选择第二个即可
