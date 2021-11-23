---
title: Nodejs 学习笔记
date: 2021-11-23 19:42:13
categories:
- Nodejs
tags:
- 笔记
---

[视屏教程](https://www.bilibili.com/video/BV1Ns411N7HU)

## Day 1

### 安装

TODO

版本查询：`node --version`

### Hello Word

新建测试文件 hello.js，写入内容如下

```js
var foo = 'bar';
console.log(foo);
```

执行文件 `node hello.js`, 可以看到 log 输出

### 读文件

输出文件内容

```js
var fs = require('fs')

fs.readFile('./helloworld.js', function(error, data) {
    console.log(data.toString())
})
```

### 写文件和错误处理

```js
var fs = require('fs')

fs.writeFile('./hello.md', 'writing to file', function(error) {
    console.log('write success...')
})
```

### 简单 http 服务

```js
var http = require('http')

var server = http.createServer()

server.on('request', function(request, response){
    console.log('accept request, path: ' + request.url)

    // 中文需要自定义 header
    response.setHeader('Content-type', 'text/plain; charset=utf-8')
    response.write('reponse given...')
    response.write('你好')
    // 必须用 end 结尾
    response.end()
})

server.listen(3000, function() {
    console.log('server started...')
})
```

### 核心模块

Node 为 JS 提供了很多服务器级别的 API，包装到具名的核心模块中。例如 fs/http/path/os 等，通过 `require('xx')` 使用。

### 模块系统

分三类：

* 具名核心模块
* 自己编写的文件模块 `require('/path/to/b.js')`

js 文件为顺序执行，包括 require 中的内容。

Node 中可以通过 exports.foo = 'hello' 暴露文件中的变量，通过 `var bExports = require('./b')` 得到

### Content-type

fs.readFile() 之后可以通过指定 Content-type 来指定返回数据，专业名称叫做 mime 类型