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

### 无分号代码风格

代码以 (, [, ` 开头的，补分号确保语法解析正确。 ES6 中使用 反引号 凭借自负，支持换行。反引号中可以使用 ${} 做替换操作。

### 模版引擎

安装：`npm install art-template`

浏览器中使用 art-tempalte, 新建 html 文本，写入内容. 打开 browser 可以看到 console 中有对应的 'hello Jack' log.

{{}} 被称为 mustach 语法

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title> Test art template </title>
</head>

<body>
    <script src="node_modules/art-template/lib/template-web.js"></script>


    <script type="text/template" id="tpl">
        我叫 {{ name }}
        今年 {{ age }}
        喜欢 {{ each hobbies }} {{ $value }} {{ /each }}
    </script>

    <script>
        var ret = template('tpl', {
            name: 'Jack',
            age: 11,
            hobbies: [
                '唱',
                '跳',
                'rap'
            ]
        })

        console.log(ret)
    </script>
</body>
</html>
```

Node 中使用 模版引擎

```js
var template = require('art-template')

var ret = template.render('hello {{ name }}', { name: "Jack"})
console.log(ret)
// hello Jack
```

查看源代码能找到的都是服务器端渲染的，商品列表一般用服务器端渲染，方便 SEO 搜索引擎查找。

终端直接输入 node 可以得到自带的交互界面（REPL）

### Node 中的模块系统

程序主要在：

* EcmaScript 语言
  * 和浏览器不一样，没有 BOM，DOM
* 核心模块
  * 文件操作 fs
  * http
  * url
  * ...
* 第三方模块
  * art-template
* 自己写的模块

#### CommonJS 模块规范

* 模块作用于
* 使用 require 加载模块
* 使用 exports 带出模块中成员

require 语法：`var name = require('module')`, 作用：

* 执行被夹在模块中的代码
* 得到被夹在模块中 exports 到处接口对象

exports 作用：

* Node 中是模块作用域，默认文件中所有的成员只在当前文件模块有效
* 对于希望可以被其他模块访问的成员，我们需要把这个公开的成员挂在到 exports 接口对象上

导出多个成员：

```js
exports.a = 123
exports.b = 'hello'
```

导出单个成员:

```js
module.exports = add

module.exports = {
    add: function() {
        return x + u
    },
    str: 'hello'
}
```

声明多个，后者覆盖前者

如果模块想要直接到处成员，而非挂在的方式，可以使用 `module.exports=add`

加载规则：

* 优先从缓存加载
* 判断模块标识符
  * 核心模块，核心模块文件已被编译成二进制文件，直接使用名字即可
  * 第三方模块，通过 npm 下载，通过 require('包名') 引用
  * 自己写的模块

### package.json

建议每个项目都要有 package.json, 管理包依赖。 --save 选项可以将依赖加进去 e.g. `npm install jquery --save`

这个文件可以通过 `npm init` 向导生成

### npm 常用命令

```bash
npm --version

npm install --gloabl npm # 自己升级

npm init
    npm init -y 跳过向导，快速生成

npm install packege_name
npm install packege_name --save
npm i -S package_name # 简写

npm uninstall package_name
# 新版的都不需要 --save 了
```

### npm 加速

```bash
npm install --gloabl cnpm
cnpm install xxx
```

### Express

原生的 http 在某些方面不足以满足开发需求，使用框架加快开发，代码高度统一。Express 是 node 的一个 web 框架。

hello world 代码如下

```js
var express = require('express')

var app = express()

app.get('/', function(req, res){
    res.send('hello express...')
})

// 中文自动处理了
app.get('/about', function(req, res){
    res.send('你好 express...')
})

app.listen(3000, function() {
    console.log('app is running at 3000')
})

// 方便的公开指定目录
app.use('/public', express.static('./public/'))
```

### 热部署

nodemon, 代码修改完立刻生效 `npm install --global nodemon`, 使用时使用 `nodemon app.js` 即可

### 基本路由

当

```js
app.get('/', function(req, res){
    res.send('..')
})

app.post('/', function(req, res){
    res.send('..')
})
```

### 静态文件

通过 `app.use('/public', express.static('./public/'))` 的方式公开资源访问。第一个参数为别名，可以为任何表达式，同文件夹名更容易辨识

### Express 中配置使用 art-template 模版引擎

```js
npm install --save art-template
npm install --save express-art-template
```

配置

```js
app.engine('art', require('express-art-template'))
```

使用

```js
app.get('/', function(req, res){
    // express 默认回去项目中的 views 目录找 index.html
    res.render('index.html', {
        title: 'hello world'
    })
})
```

如果要改默认 views 试图渲染存储目录，可以

```js
app.set('views', 目录路径)
```

### express 获取表单 post 请求

安装：`npm install --save body-parser`, 貌似新版的已经自动集成了，不需要自己下载

使用案例

```js
var express = require('express')
var bodyParser = require('body-parser')

var app = express()

// parse application/x-www-form-urlencoded
app.use(bodyParser.urlencoded({ extended: false }))

// parse application/json
app.use(bodyParser.json())

app.use(function (req, res) {
  res.setHeader('Content-Type', 'text/plain')
  res.write('you posted:\n')
  res.end(JSON.stringify(req.body, null, 2))
})
```

PS: 对于 get 请求，内置了 req.query 对象作为 body 的容器

### crud demo

```bash
npm init -y
npm i -S express
npm i -S bootstrap@3
```

### MongoDB

```bash
mongo # 启动终端

exit # 退出

show dbs # 显示数据库

db # 当前数据库

use xxx # 切换指定数据库

db.students.insertOne({ "name": "Jack" }) # 插入数据

db.students.find() # 查询

show collections # 显示表
```

基本概念

* 数据库
* 集合 - 表
* 文档 - 一条记录
* 文档结构没有任何限制
* 灵活，不需要建表，直接使用

### Mongoose

基于官方包的再封装

```bash
npm init -y
npm i mongoose
```

跟着官方文档跑了一个 demo

