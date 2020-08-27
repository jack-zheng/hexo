---
title: Python 脚本高频报错
date: 2020-06-12 10:43:08
categories:
- 编程
tags:
- python
---

## requests lib SSLError

在使用 requests 发送 API 请求的时候，如果网站是 https 的，如果你没有对应的证书就会抛 SSLError, 示例如下：

```python
headers  = {'Authorization' : 'token xxx'}
url = 'https://github.domain.com/api/v3/users/ixxx'
resp = requests.get(url, headers=headers)

''' Error show as:
SSLError: HTTPSConnectionPool(host='github.wdf.sap.corp', port=443): Max retries exceeded with url: /api/v3/users/i332399 (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1076)')))

In [7]: resp = requests.get(url, headers=headers, verify=False)
/Users/i306454/gitStore/mycommands/.venv/lib/python3.7/site-packages/urllib3/connectionpool.py:851: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)
'''
```

解决方案有两个

1. 跳过verify
1. 指定证书

> 方案一

requests.get(url, auth=(), verify=False)
但是，这种方式会在发完request之后抛warning，对于强迫症患者说简直不能忍。

> 方案二

在request中指定证书路径 `requests.get(url, auth=auth, verify='/Users/jack/Downloads/my.crt')`
