---
title: Idea 常用配置
date: 2019-12-26 22:24:12
categories:
- 配置
tags:
- idea
- maven
- 配置
- 快捷键
---
Idea 中关于 Maven 的一些配置

## Configurations

### 避免 import *

默认设置下，同一个包下 import 数量超过 5 个就会用 * 来代替，可以去 Setting -> editor -> code style -> java, 然后右边选择 Imports tab, 修改 'Class count to use import *' 的值即可

### Maven 下载仓库配

1. Shift + Ctrl + A -> 搜索 `Settings.xml`, Open/Create 这个文件 -> 添加仓库地址  
1. localRepository 这个变量的地址应该是对应到本地的 `.m` folder 下的 repository 文件夹  
1. Settings.xml 路径可以在 'Build, Excutations, Deployment' 下的 maven tag 下查看

### 设置 Maven 自动下载包源码

1. Build, Excutations, Deployment -> Maven -> Importing -> Automatically download: source, documentation 打勾  
1. 回到主界面，在侧边栏的 Maven 里面会出现 'Download source and/or documentation' 的按钮

### Win10 下 Idea/NVIDIA 快捷键冲突

1. NVIDIA Graphic 开启的时候 Ctrl + Alt + 方向键会变成调整显示方向的设置，和 Idea 的代码跳转冲突
1. 右键桌面 -> 图形属性 -> 选项和支持 -> 禁用快捷键

### Idea 查看 JDK 源码

File -> project setting -> SDKs -> 右边有个 Sourcepath -> 导航到 JDK 文件目录下找到 src.zip 就行了

### 设置条件断点

添加断点之后，在断点上右键输入你想要的条件，比如： a==10

### 复制代码段的时候，取消格式复制

cmd+shift+A 打开搜索框，输入关键字 'copy as rich text', 关闭对应的开关

### IDEA中显示空格

cmd+shift+A 打开搜索框，输入关键字 'show withspace', 操作对应的开关

### 快速实现 tab <-> space 转化

1. cmd+shift+A 打开搜索框，输入关键字 'convert indents', 选择 'To Spaces'
1. 或者在输入关键字的时候直接选择 'To Spaces'

### 查看类继承关系

Navigate -> Type Hierarchy 或者 Ctrl + H

### 关闭 Idea 自动更新提示

快捷搜索 `Automatically check update for` 然后将更新选项去掉

### Class 生成 Enter就会提示自动创建serialVersionUID

1. Setting->Inspections->Serialization issues->Serializable class without ’serialVersionUID’ 
1. 选上以后，在你的class中：Alt+Enter就会提示自动创建serialVersionUID了。

## 插件

* 查看字节码：安装 jclasslib，重启。选中文件，选择导航栏上的 view -> Show byte code with jclasslib 选项即可

## 快捷键

| 功能       |       Mac       |  Win  |
| :--------- | :-------------: | :---: |
| 万能快捷键 | CMD + Shift + A | TODO  |
| 查找类     |     CMD+ O      | TODO  |
