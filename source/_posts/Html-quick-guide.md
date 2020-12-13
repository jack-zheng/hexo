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

### CSS 的三大特性

### 层叠性

1. 样式冲突，遵循就近原则，执行离的近的那个
2. 样式不冲突，不层叠

### 继承性

子标签会继承父标签的某些样式，比如文字颜色和字号等

* 恰当的使用继承合一简化代码，降低 CSS 复杂性
* 子元素可以继承父元素的样式 (text-, font-, line- 和 color)

```html
<style>
    body {
        /*文字大小/行高 行高可以指定像素，也可以是字号的倍数，这里是 1.5 倍*/
        font: 12px/1.5 'Microsoft YaHei';
    }
</style>
```

### 优先级

1. 选择器相同，则执行层叠性
2. 不同，则根据权重来

权重排行：

| 类型                 | 权重    |
| :------------------- | :------ |
| 继承或者 *           | 0,0,0,0 |
| 元素选择器           | 0,0,0,1 |
| 类选择器，伪类选择器 | 0,0,1,0 |
| id 选择器            | 0,1,0,0 |
| 行内样式 style=""    | 1,0,0,0 |
| !important 重要的    | 无穷大  |

复合选择器会有权重叠加 `.nav li` = 0010 + 0001 = 0011

## 盒子模型 P136

网页布局本质

1. 准备好页面元素，基本都是盒子
2. CSS 设置盒子样式，摆放到相应位置
3. 往盒子里装内容

组成：

1. 边框 border
2. 内容 content
3. 内边距 padding
4. 外边距 margin

### 边框

三部分：边框粗细，样式 和颜色

border-width, border-style, border-color 可以写成复合形式： `border: 5px solid red;`

单条边框设置：border-top/bottom/left/right

border-collapse: collapse; 表格边框线合并

边框会影响盒子实际大小

### 内边距 padding

padding-left/top/right/bottom

复合写法：

| 写法                        | 意义                  |
| :-------------------------- | :-------------------- |
| padding: 5px                | 上下左右都是 5px      |
| padding: 5px 10px           | 上下 5px, 左右 10 px  |
| padding: 5px 10px 20px      | 上，左右，下          |
| padding: 5px 10px 20px 30px | 上，右，下，左 顺时针 |

padding 也撑大盒子

### 外边距 margin

控制盒子与盒子之间的距离

margin-left/top/right/bottom

**块级**盒子实现水平居中：

1. 盒子必须有宽度
2. `margin: 0 auto;`

**行内**元素实现水平居中：给父级元素添加 `text-align: center;`

外边距合并-嵌套快元素塌陷，解决方案：

1. 可以为父元素定义上边框
2. 可以为父元素定义上边距
3. 可以为父元素添加 overflow: hidden 属性

不同的网页元素默认都会带有不同的 内外边距，而且不同 browser 值不同，清除如下

```html
<style>
/*一般都会先写这一句*/
    * {
       margin: 0;
       padding: 0;
    }
</style>
```

行内元素尽量只设置左右的内外边距

### 示例

`li { list-style: none;}` 去掉 list 的原点

### 圆角边框

border-radius: npx 或 百分比;

### 盒子阴影

box-shadow: h-shadow v-shadow blur spread color inset; rgba(0,0,0,.3)

### 文字阴影

text-shadow: h-shadow v-shadow blur color;

## 浮动 float

传统三种布局方式

1. 标准流 - 默认方式
2. 浮动
3. 定位

浮动可以改变元素的默认排列方式

网页布局第一准则：多个块级元素纵向排列找标准流，多个块级元素横向排列找浮动

**float**属性用于创建浮动框，将其移动到一边，知道左边缘或右边缘触及包含块或另一个浮动框的边缘。

### 浮动的特性

设置浮动的元素最重要的特性：

1. 脱离标准普通流的控制，移动到指定位置，俗称 脱标
2. 浮动的盒子不在保留原先的位置

多个盒子设置浮动，则它们会按照属性值一行内显示并且顶端对齐排列

PS: 浮动元素相互没有间隙，父级宽度装不下，另起一行对齐

浮动元素有行内块特性

一浮全浮，如果一个父元素下有一个加了浮动，一般其他所有多要加浮动属性

浮动指挥影响浮动盒子**后面**的标准流，前面的不会受影响

### 清除浮动

为什么要清除浮动：

父盒子在很多情况下不方便给高度，但是盒子浮动之后就不占有位置，最后父级盒子高度为 0 时就会影响像下面的标准流盒子。

本质：清除元素浮动造成的影响

选择器 {clear: 属性值(left/right/both);} , 实际种都用 both

策略：闭合浮动，只让浮动在父盒子内部影响，不影响父盒子外部的其他盒子

方法：

1. 额外标签法，也称为隔墙法，W3C 推荐做法
2. 父级添加 overflow 属性
3. 父级添加 after 伪属性
4. 腹肌添加双伪属性

额外标签法：

在最后一个浮动子元素后面添加一个额外标签，添加清除浮动样式，实际工作可能会遇到但是不常用

* 优点：通俗易懂，书写方便
* 缺点：添加许多无意义的标签，结构差

示例: `<div style="clear:both"></div>`

PS: 最后的这个元素必须是块级元素，行内元素不行

父元素 overflow:

`.body {overflow: hidden;}`

属性可选项： hidden, auto 和 scroll

* 优点： 代码简洁
* 缺点：无法显示溢出部分

:after 伪元素

声明属性并在父盒子内添加，原理和隔墙法一致，只不过使用 CSS 实现的

```css
.clearfix:after {
    /* 内容为“.”就是一个英文的句号而已。也可以不写 */
    content: ".";
    /* 加入的这个元素转换为块级元素 */
    display: block;
    /* 清除左右两边浮动 */
    clear: both;
    /* -可见度设为隐藏。注意它和display:none;是有区别的。仍然占据空间，只是看不到而已 */
    visibility: hidden;
    height: 0;
    font-size:0;
}
.clearfix {
    /* 兼容 IE 6， 7 */
    *zoom: 1;
}
```

* 优点：没有增加标签，结构接单
* 缺点：代码多

双伪元素清除浮动

```css
/* 声明清除浮动的样式 */
.clearfix:before,
.clearfix:after {
content: "";
display: table;
}
.clearfix:after {
clear: both;
}
/* ie6 7 专门清除浮动的样式*/
.clearfix {
*zoom:1;
}
```

## PS 切图

* jpg: 色彩信息保留较好，高清，颜色多。产品类图片经常用 jpg 格式
* gif： 256 色，动态
* png 可以保存透明背景，切透明背景图片
* PSD，PS 的图片格式
  
cutterman 切图神器

### CSS 书写规范

1. 布局定位属性： display/position/float/clear/visibliity/overflow, display 第一个写，关系到模式
2. 自身属性：width/height/margin/padding/border/background
3. 文本属性： color/font/text-decoration/text-align/vertical-align/white-space/break-word
4. 其他属性(CSS3)： context/cursor/border-radius/box-shadow/text-shadow/background:linear-gradient...

### 布局思路

1. 必须确定版面的版心(可视区)，我们测量可得知
2. 分析页面中的行模块，以及每个行模块种的列模块
3. 一行中的列模块经常浮动布局，先确定每个列的大小之后，确定列的位置，页面布局第二准则
4. 制作 html, 我们还是遵循，先有结构，后又样式的原则。结构永远最重要。

实际开发中，我们不会直接用链接 a 而是用 li 包含链接(li+a)的做法。主要是针对 SEO 搜索的优化。

练习先跳过了，后面要用在看

## 定位 P221

自由的定位，标准流和浮动做不出这种效果

定位 = 定位模式(position) + 边偏移

position = static/relative/absolute/fixed

边偏移 = top/bottom/left/right

静态定位，即默认形式，了解即可 `选择器 {position: static;}`

相对定位：

* 自恋型，移动时参照原来的位置
* 相对于浮动，原来的位置继续保留，不脱标
* `选择器 {position: relative;}`

绝对定位 absolute：

相对与祖先元素来说的，拼爹型 `选择器 {position: absolute;}`

1. 如果没有祖先元素或祖先元素没有定位，则以 document 为标准
2. 如果祖先有定位(相对，绝对，固定定位)，则以最近一级有定位祖先元素为参考点移动位置
3. 绝对定位会脱标，不占有原来位置

子绝父相

固定定位 fixed：

元素可以停在浏览器的可视区域的固定位置，滚动不影响  `选择器 {position: fixed;}`, 不占用原先的位置

固定定位小技巧：固定在版心右侧位置

1. 让固定定位的盒子 left: 50% 走到可视区域的一般位置
2. 让固定定位盒子 margin-left: 版心宽度一般距离

就可以实现该效果

粘性定位 sticky:

1. 以可是窗口为参照点移动元素
2. 占有原先位置
3. 必须设置 top, bottom, left, right 其中一个才会生效

IE 不支持

### 定位叠放次序 z-index

`选择器 {z-index: 1;}`

数值可以是正整数，负整数或0，默认是 auto, 数值越大，盒子越靠上

只有定位的盒子才有 z-index 属性

### 绝对定位居中

left: 50% + margin-left: -盒子本身宽度px;

### 定位拓展

绝对定位和固定定位也和浮动类似

1. 行内元素添加绝对定位或者固定定位，可以直接设置宽度和高度
2. 块级元素添加绝对或者固定定位，如果不给宽度或高度，默认大小是内容大小

脱标的元素不会触发外边距合并

浮动元素只会压住标准流的盒子，不会压住文字，定位会压住文字。

浮动最初产生的动机时用来做环绕效果得，所以不会压住文字