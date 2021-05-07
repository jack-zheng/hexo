---
title: Requests 关闭 SSL 验证提示
date: 2021-05-07 12:11:08
categories:
- python
tags:
- requests
- SSL
---

使用 requests lib 访问 HTTPS 网站的时候，如果关闭了认证，虽然访问可以继续，但是会抛 Warning 信息。

```python
requests.get(request_url, headers=headers, verify=False)

# /Users/jack/.pyenv/versions/3.7.3/lib/python3.7/site-packages/urllib3/connectionpool.py:851: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
#   InsecureRequestWarning)
```

按照提示，去对应的网页查看关闭提示信息的办法

```python
import urllib3
urllib3.disable_warnings()
```

可行！ (●°u°●)​ 」