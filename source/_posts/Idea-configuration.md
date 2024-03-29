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

### Debug 显示设置

debug 时一些值比如 `byte[]` 想要看具体的值时多少，可以右键 -> Evaluate Expression... 输入表达式 `new String(dmBytes)` 查看，也可以通过 add to watch 输入同样的表达式

### Debug 时查看过滤变量

e.g. debug 窗口中，变量存在集合中，比如 List，而且这个变量是个复杂变量，我想要检测某一个变量是否在这个集合中，可以右键 list -> Filter, 输入删选条件，比如 `this.name.startWith("ja")` 即可，贼方便

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

### 设置注释文字不顶格子

搜索 comment code 并将 Line comment at first column 和 Block comment at frist column 的选项 disable 掉

![comment code](comment_code.png)

### Class 生成 Enter就会提示自动创建serialVersionUID

1. Setting->Inspections->Serialization issues->Serializable class without ’serialVersionUID’ 
1. 选上以后，在你的class中：Alt+Enter就会提示自动创建serialVersionUID了。

### 设置终端 log size

默认终端数量有限，稍微多点就把前面的给冲掉了，可以设置 Preferences > Editor > General > Console, 勾选 Override console cycle buffer size (1024 KB)，并把值调大就行

## 插件

* 查看字节码：安装 jclasslib，重启。选中文件，选择导航栏上的 view -> Show byte code with jclasslib 选项即可

## 快捷键

| 功能       |       Mac       |  Win  |
| :--------- | :-------------: | :---: |
| 万能快捷键 | CMD + Shift + A | TODO  |
| 查找类     |     CMD+ O      | TODO  |

## Spring 中 Autowired warning

Settings -> Editor -> Code Style -> Inspections -> Spring Core -> Code -> Field injection warning 选项 disable 掉

## maven-assembly-plugin not found

For newer versions of IntelliJ, enable the use plugin registry option within the Maven settings as follows:

1. Click File -> Settings.
2. Expand Build, Execution, Deployment -> Build Tools -> Maven. Check Use plugin registry.
3. Click OK or Apply.

For IntelliJ 14.0.1, open the preferences---not settings---to find the plugin registry option:

1. Click File -> Preferences. Regardless of version, also invalidate the caches:
2. Click File -> Invalidate Caches / Restart.
3. Click Invalidate and Restart.

When IntelliJ starts again the problem should be vanquished.

## 默认注释格式

默认注释格式会把双斜杠放到最前面，和习惯很不搭配，可以通过 Perferences -> settings -> Editor -> Code style -> java 跳出设置界面

到 Code generic tab 下面，将 Comment Code 选项下的 `Line comment at first coumn` 去掉，下一级的 `Add a space at comment start` 选上即可

## 2021-04-15

升级 Idea 之后，原来的项目在 Idea 里面编译失败，但是终端却可以。由此断定项目肯定是好的。Google 了一下，可以通过 File -> Invalidate caches -> Invalidate and Restart 重启 Idea 解决问题