---
title: Python 日期和时间
date: 2020-06-18 16:34:33
categories:
- 编程
tags:
- python
---

对 python 中涉及到时间的类库做一个大致的了解并收集一些常用的 sample。类库包括：time, date, datetime, timezone 等

## datetime 日期时间

时间类型分为感知型和简单型，感知型包含 timezone 信息，简单型则没有这种意义。

* date 都是简单型的
* time 和 datetime 可以是简单型也可以是感知型，通过 d.tzinfo 不等于 None 或者 d.tzinfo.utcoffset(d) 部位 None 来确定

```python
from datetime import datetime

# 获取当前时间
datetime.now()
# Out[18]: datetime.datetime(2020, 6, 18, 17, 2, 48, 14847)

# 感知型 now
from datetime import timezone
dt =datetime.now(timezone.utc)

# datetime 得到 s
dt.timestamp()
# Out[41]: 1592472504.59345

# s 转 datetime, ms 的话把时间除1000.0即可 1592472504.59345/1000.0
d = datetime.fromtimestamp(1592472504.59345)
# Out[43]: datetime.datetime(2020, 6, 18, 17, 28, 24, 593450)
```

date, time, datetime 都支持 strftime(), 只有 datetime 支持 strptime()。

```python
# strftime: string from time, 即格式化输出时间, 对象方法
now = datetime.now()
now.strftime('[%y%m%d]-[%H:%M:%S]')
# Out[22]: '[200618]-[17:12:46]'

# strptime: string parse to time, 即将字符串转化为时间, 类方法
dt = datetime.strptime('[200618]-[17:12:46]', '[%y%m%d]-[%H:%M:%S]')
# Out[24]: datetime.datetime(2020, 6, 18, 17, 12, 46)
```

## deltatime 时间间隔

```python
from datetime import timedelta
delta = timedelta(days=50, seconds=27, microseconds=10, milliseconds=29000, minutes=5, hours=8, weeks=2)
# Out[16]: datetime.timedelta(days=64, seconds=29156, microseconds=10)

# 可以通过 datetime 做计算得到
now - dt
# Out[26]: datetime.timedelta(seconds=329, microseconds=894908)
```
