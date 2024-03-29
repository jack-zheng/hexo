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

> 替换成换行

`cmd + f` 唤出搜索看，点击 `.*`, 然后，查找框输入 `,` 替换框输入 `\n` 即可

> 重复一行

光标停留在目标行，opt + shit + 上/下 即可，666

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
* vscode-input-sequence: 输入顺序数字，好用到飞起