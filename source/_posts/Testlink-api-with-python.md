---
title: Python 调用 Testlink API
date: 2021-03-17 16:43:05
categories:
- Third Part
tags:
- Testlink
- API
---

备忘一下 Testlink API 调用实现。Testlink 是有实现自己的 API 接口的，你可以通过它来操作 project， test plan 等对象。

## API 示例

安装 pip lib: `pip install TestLink-API-Python-client`， project repo: [Github, Testlink clint - Python](https://github.com/lczub/TestLink-API-Python-client)

登陆 Testlink -> My Settings -> My personal access key 拿到 API 授权的 key, 尝试链接

```python
import testlink
url = "https://<hostname>/lib/api/xmlrpc/v1/xmlrpc.php"
#  for this key, you can get forom Testlink -> click My Settings → my personal access key field
key = "api_auth_key" # test link personal key
tlk = testlink.TestLinkHelper(url, key).connect(testlink.TestlinkAPIClient)
print(tlk.tlk.countProjects())
# 39

# get project info
projects = tlk.getTestProjects()
for sub in projects:
    print("id: %s, prefix: %s, name: %s" % (sub['id'], sub['prefix'], sub['name']))
# id: 5182, prefix: PLT#, name: Platform Foundations and Integrations
```

其他的 API 都和差不多类似的，通过使用 Ipython + tab 基本都可以找到

## 注意点

python 提供的接口中有一个很有意思：`tlk.whatArgs('getTestProjects')` 他可以给出对应 API 的调用方式，参数列表的信息，很有用

client 项目的 [example](https://github.com/lczub/TestLink-API-Python-client/blob/master/example/) 路径下，有几个示例，写的很清楚，基本上把所有支持的命令都写了，值得参考

PS：记得查看 Testlink 版本，就我自己的情况，公司内部用的还是 13 年的版本(1.9.7) 而最新的都已经是 1.9.20 了，好多 API 都不支持，可以 blame 以下 example，查看对应的例子是什么时候加进去的，看你想要的功能是否支持