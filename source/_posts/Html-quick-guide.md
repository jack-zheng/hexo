---
title: Html quick guide
date: 2020-12-05 15:25:55
categories:
- 编程
tags:
- html
---

HTML + CSS + JS = 骨 + 肉 + 灵

[Pink 前端视频教程](https://www.bilibili.com/video/BV14J4114768) 学习笔记

## VSCode

在 html 文件中 `! + tab` 自动生成网页骨架

插件：

* open in browser
* auto rename tag
* JS-CSS-HTML Formatter
* CSS Peek

## Html 常用标签 p12-60

`<!DOCTYPE html>` 文档类型声明标签，表明时 html5 格式。

`<html lang="en">` 显示语言，只起提示作用，浏览器遇到不同语言会给你翻译提示

`<meta charset="UTF-8">` 字符集

`<hn>` n=1-6 总共支持 6 级标签

`<p>` 段落

`<br/>` 换行，单标签

文本效果：

* `<strong>`, `<b>` 加粗
* `<em>`, `<i>` 倾斜
* `del`, `s` 删除线
* `<ins>`, `<u>` 下划线

`<div>` 盒子标签，没有语义，div 分割，分区，占一整行

 `<span>` 盒子标签，跨度，跨距

 `<img src='' alt='' title='' width='' height='' border='边框，一般通过 css 改'/>` 图像

 <a href='' target='_self/_blank'> 超链接， _blank 新标签页打开

 锚点链接: <a href='#someid'></a> + <link id='someid'>

 注释 <!-- something -->

 特殊字符：

| name | value  |
| :--- | :----- |
| 空格 | &nbsp; |
| 小于 | &lt;   |
| 大于 | &gt;   |

**表格**

作用：显示，展示数据，table->tr->th/td

属性：align(left/center/right) 对齐；border 边框；cellpadding 单元格和内容之间的距离；cellspacing 单元格之间的距离；width 表宽；

table 可以分成 theader + tbody, theader 包含 tr + th，tbody 包含 tr + td 内容。

合并单元格：rowspan="合并单元格数" 跨列合并，左侧的为目标单元格， colspan="合并单元格数" 跨行合并 上面的为目标单元格

**列表**

作用：布局

分类：无序列表，有序列表，自定义列表

无序：ul + li, ul 下第一级只能包含 li，li 下可以包含任何标签

有序：ol + li

自定义：dl + dt + dd，dl 下只能包含 dt 和 dd, 适用于小标题加说明的情况

```html
<dl>
    <dt>关注我们</dt>
    <dd>微信</dd>
    <dd>微博</dd>
</dt>
```

**表单**

作用：收集用户信息

组成

* 表单域：<form> 将域范围内的内容提交到后台
* 提示信息
* 表单控件(元素)：input/select/textarea

`<input type='类型'/>`, 类型有很多种，text, boolean, button, radio， reset, file, image 等, 还有 name, value, checked 和 maxlength 四个属性

label 标签：绑定页面元素，点击 label 光标会自动聚焦到绑定元素，增加用户体验 -- 这个之前一直没注意(～￣▽￣)～ 

示例：

```html
<!-- 通过 label 里面的 for 属性生效 -->
<label for="text"> 用户名：</label> <input type="text" id="text"> 
```

下拉列表：多选一

```html
<select>
    <option selected="selected">xxx</option>
    <option>aaa</option>
</select>
```

文本域 textarea: cols, rows

## CSS P60-

css=选择器 + 一条或多条声明, 放在 head 下的 `<style/>` 下

```html
<style>
    p {
        color: red;
        font-size: 12px;
    }
</style>
```

选择器分为 基础选择器 和 复合选择器

基础选择器：由单个选择器组成，包括：标签选择器，类型选择器， id 选择器 和通配符选择器

```html
<style>
/* 标签选择器 */
/* 标签名做选择标的 */
    p {
        color: green;
    }

    div {
        color: pink;
    }
</style>
```

```html
<style>
/* 类型选择器, 样式(class)做选择标的 */
    .red {
        color: green;
    }
</style>
```

```html
<style>
/* id选择器, id 做选择标的 */
    #pink {
        color: green;
    }
</style>
```

```html
<style>
/* 通配符选择器, 通配符做选择标的 */
    * {
        color: green;
    }
</style>
```

### 字体属性

* font-family: "Microsoft YaHei";
* font-size: 20px;
* font-weight: blod; 文字效果，加粗，也可以用数字 blod=700 normal=400, 范围 100-900
* font-style: normal/italic; 斜体

字体的复合属性

```html
<style>
    div {
        /* font: font-style font-weight font-size/font-height font-family; */
        /* font-size 和 font-family 必须有，其他可以省略，不然不起作用 */
        font: italic 700 16px 'Microsoft YaHei';
    }
</style>
```

### 文本属性 P72-83

* color, 表示方式，blue，16进制(#)或者 RGB(RGB(X,X,X))
* text-align, 对齐， left, right, center
* text-decoration 文本装饰，下划线之类的效果 none(取消超链接的下划线效果), underline, overline, line-through
* text-indent: 2em; 段落首行缩进
* line-height：26px 行间距, 行间距=上间距+文字高度+下间距

测量行高小工具 FSCapture.exe

### CSS 引入方式

1. 内部样式表
2. 外部样式表
3. 行内样式表

内部样式：放到 header 下的 style tag 下

行内样式： `<p style="color: pink;">fense</p>`

外部样式：

1. 新建 .css 文件
2. 使用 <link> 标签引入到 thml `<link rel="stylesheet" href="css_file_path">`

### Emment 语法

前身时 Zen coding， 使用缩写提高 html/css 编写速度，VSCode 已经内置了

1. 标签 + tab 自动生成
2. tag*n + tab 生成多个
3. ul>li 生成父子级关系 tag
4. div+p 兄弟级 tag
5. p.one 生成 `<p class="one">`, `#` 生成 id
6. `.demo$*5` 带序号的 div
7. `div{123}` -> `<div>123</div>`

### CSS 复合选择器

即基础选择器的组合形式，有后代选择器，子类选择器，并集选择器，伪类选择器等。。。

后代选择器影响所有后代

```html
<style>
    /*父子级中间加空格*/
    ol li {
        /* 后代选择器 */
        color: pink;
    }

    .nav ul li {
        /* 后代选择器 */
        color: blue;
    }
</style>
```

子选择器，只对亲儿子起作用(第一层)，其他不起作用, 父子间不用空格

```html
<style>
    /*父子级中间加空格*/
    ol > li {
        /* 后代选择器 */
        color: pink;
    }
</style>
```

并集选择器，使用都好分割, 用于删选多组数据

```html
<style>
    /*父子级中间加空格*/
    div,
    p {
        /* 后代选择器 */
        color: pink;
    }
</style>
```

伪类选择器，冒号(:), 比如 :hover, :first-child

链接伪类选择器：

1. a:link 选择所有未被访问的链接
2. a:visited 选择所有已被访问过的链接
3. a:hover 选择鼠标指针位于其上的链接
4. a:active 选择活动链接，鼠标按下未弹起

注意点:

1. 一定要按照 LVHA 顺序写，不然会胡问题 (Love Hate / LV 包包 hao)
2. a 需要单独指定样式，直接指定 body 没用
3. 实际使用一般指定 a + a:hover 即可

:focus 伪类选择器，选择表单中光标作用的元素

```html
<style>
    /*父子级中间加空格*/
    input:focus {
        /* 后代选择器 */
        background-color: pink;
    }
</style>
```

### CSS 的元素显示模式

HTML 一般分为块元素, 占一行(比如 div)和行内元素(比如 span)，一行多个

#### 常见块元素

h1~h6, p, div, ul, ol, li 等

特点：

1. 霸道，独占一行
2. 高，宽，外边距以及内边距都可控
3. 宽度默认时容器(父级宽度)的 100%
4. 是一个容器及盒子，里面可以放行内或者块级元素

PS: 文字类元素，比如 h, p 种不能再放块级元素

#### 常见行内/内联元素

a, strong, b, em, i, s, u, span

特点：

1. 相邻元素在一行上，一行可以显示多个
2. 高宽直接设置是无效的
3. 默认宽度是它本身内容的宽度
4. 行内元素只能容纳文本或其他行内元素

PS: 链接里面不能套链接；特殊情况下链接里面可以放块级元素，但是给 a 转换一下块级模式最安全

#### 行内块元素

img, input, td

特点：

1. 一行可以多个 (行内元素特点)
2. 默认宽度是它本身内容的宽度 (行内元素特点)
3. 高度，行高，外边距以及内边距都可以控制 (块级元素特点)

### 元素显示模式转换

特殊情况下，需要元素模式转换，即一个模式元素需要另一种模式的特性，比如想要增加 <a> 的触发范围。

做法在样式种加入 display:block, 或 display:inline, display: inline-block


```html
<style>
    a {
        width: 150px;
        height: 50px;
        background-color: pink;
        display: block;
    }

    div {
        background-color: blue;
        display: inline;
    }

    span {
        width: 150px;
        height: 50px;
        background-color: yellow;
        display: inline-block;
    }
</style>
```

### 小工具介绍 Snipaste

微软商城可以直接下载，貌似免费，mac 上类似的是 snappy

1. F1 截图，同时测量大侠，设置箭头，文字等功能
2. F3 桌面置顶
3. 点击图片 alt 拾取颜色
4. esc 取消图片显示
5. alt + c 直接复制拾取的颜色

### 单行文字垂直居中

行高=上间隙 + 字高 + 下间隙，设置 行高=盒子高度 `line-height: 行高` 实现居中

行高 < 盒子高度， 偏上
行高 > 盒子高度， 偏下

### CSS 的背景 P115-126

背景属性可以给页面元素添加背景样式，比如背景颜色，背景图片，背景平铺，背景图片为止，背景图片固定等。

background-color: transparent | color

background-image: none | url(), 适用于 logo, 装饰性小图片，超大背景图片，优点是便于控制位置

background-repeat: repeat | no-repeat | repeat-x | repeat-y

background-position: x y; **重点**，x, y 可以是像素值，也可以是方位名词(left, center, right)

background-attachment: scroll | fixed 背景图片固定/附着，是否更正页面滚动变化，视差滚动效果

复合写法，直接在 background: 属性1， 属性2 ... 没有顺序要求

#### CSS3 半透明效果

`background: rgba(x, x, x, 0.3)` 最后一位就是透明度， `0.3` 可以直接写 `.3`