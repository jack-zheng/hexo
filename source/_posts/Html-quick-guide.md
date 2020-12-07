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