---
title: Chrome 插件记事
date: 2020-07-22 18:39:41
categories:
- 小工具
tags:
- Chrome
- 插件
---

公司日常维护过程中，同事自己写的一个 Chrome 小插件很精巧，符合我小而美的审美，很适合处理某些需求，特此记录一下 Chrome 插件的小知识和一个阅读源码的收获

## 插件目录结构

```bash
Root
├── README.md
├── background.js <- 定义一些 js 脚本
├── content.js
├── doc
│   └── images
│       ├── extend_all.png
│       ├── extend_status.png
│       ├── extension_icon.png
│       ├── extension_loaded.png
│       └── load_unpacked_extension.png
├── icon.png <- icon 定义
├── images
│   ├── icon128.png
│   ├── icon16.png
│   └── icon48.png
├── jquery-3.0.0.min.js
├── manifest.json <- 定义了 extension 的基本信息，权限等，可以概览整个应用
├── options.html <- 为客户提供可选项
├── options.js
├── popup.html <- 点击弹出页面，用于交互
├── popup.js
├── style.css
└── test.js
```

* [Chrome Extension Official](https://developer.chrome.com/extensions/getstarted) 官方文档好又多

## JS 的一些知识点

* .aspx 页面，是基于微软 .Net 开发的站点
* html 页面中可以直接在 onclick 里面写 logic，简直是随心所欲
* 通过 ajax 可以实现表单提交

```html
<!-- click 中设置 confirm 内容 -->
<!DOCTYPE html>
<html>
    <body>
        <input type="button" name="ctl00$ContentPlaceHolder$GridViewLive$ctl02$Deletion" value="Delete" onclick="if (!confirm(&#39;Are you sure you want to delete the company?&#39;)) return false; console.log('Click Confirmed')" />
    </body>
</html>
```

```js
$.ajax({
  type: "POST",
  url: url,
  data: data,
  success: success,
  dataType: dataType
});

// 简写形式
$.post( "ajax/test.html", function( data ) {
  $( ".result" ).html( data );
});

// form.serialize() 可以方便的实现数据提取
$.post( "test.php", $( "#testform" ).serialize() );

// 如果想要成功提示，还可以
$.post(url, $("#ctl00").serialize()).done(
    function( data ) {
    alert( "extends success" );
    }
);
```

## 调试脚本

由于这次只是查看代码，而且验证一些函数的功能，调试还是挺顺利的，直接通过 Chrome console 就完成了，各种变量自动装载完成，美滋滋儿。
