---
title: CSS 渲染 ul 为树结构 
date: 2020-12-15 16:11:57
categories:
- 编程
tags:
- css
- tree
---

看到网上有一段代码通过 CSS 把 ul+li 块渲染成目录树结构，很赞，加入收集。源代码链接: [bootsnipp](https://bootsnipp.com/snippets/ypNAe)

PS: 这个 post 最好是在这段代码下面嵌入一个页面显示效果，但是目前没时间做这方面的 re-search，以后如果有很多 html 示例的化可以考虑一下。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <meta name="robots" content="noindex, nofollow">

    <title>My Tree</title>
    <link href="css/tree3.css" rel="stylesheet" id="bootstrap-css">
</head>

<body>
    <ul id="tree1" class="tree">
        <li>Node1
            <ul>
                <li>Node11</li>
                <li>Node12</li>
                <li>Node13</li>
            </ul>
        </li>
        <li>Node2
            <ul>
                <li>Company Maintenance</li>
                <li>Employee
                    <ul>
                        <li>Reports
                            <ul>
                                <li>Report1</li>
                                <li>Report2</li>
                                <li>Report3</li>
                            </ul>
                        </li>
                        <li>Employee Maint</li>
                    </ul>
                </li>
                <li>Human Resources</li>
            </ul>
        </li>
    </ul>
</body>

</html>
```

```css
ul {
    margin: 0;
    padding: 0;
    list-style: none;
}

.tree ul {
    margin-left: 1em;
    /* 画一条最外层 ul 边框的辅助线 */
    /* border: 1px solid red; */
    position: relative
}

 /* 树状结构竖线部分 */
.tree ul:before {
    /* 伪类选择器，会选中 ul 的第一个元素 */
    content: "";
    display: block;
    width: 0;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    border-left: 1px solid;
}

.tree li {
    margin: 0;
    padding: 0 1em;
    line-height: 2em;
    color: #369;
    font-weight: 700;
    position: relative;
    /* font-family: "Helvetica Neue", Helvetica, Arial, sans-serif; */
}

 /* 树状结构横线部分 */
 .tree ul li:before {
    content: "";
    display: block;
    width: 10px;
    height: 0;
    border-top: 1px solid;
    margin-top: -1px;
    position: absolute;
    top: 1em;
    left: 0
}

/*  覆盖最后一个节点多余的半截竖线 */
.tree ul li:last-child:before {
    background: #fff;
    height: auto;
    top: 1em;
    bottom: 0;
}
```