---
title: Python 解析 csv 文件
date: 2020-06-18 17:43:48
categories:
- 编程
tags:
- python
- csv
---

有一个 csv 文件，其中有个 column 名为 '_raw' 包含我们需要的信息，写一段脚本解析之

```txt
_raw 中文本为

08:42:50,222 INFO  [RESTCallbackSubscriber] [customerId,customerId,null,null,SFAPI,null,null] [IrisSubscriber Container[queue_seb.subscriber.pillar.deactivateuser]1]Postback for event com.company.hermes.core.SFEvent={meta:Meta={priority:0,proxyId:"null",serverName:"null",topic:"com.company.platform.mobile.deactivateuser",ptpName:null,companyId:"customerId",eventId:"a3b43584-3ceb-4760-9c01-699d635f4461",type:"null",sourceArea:"null",effectiveStartDate:"null",publishedAt:"2020-05-31 08:42:39",publishBy:"SFAPI",publishServer:"serverip",externalAllowed:false,filterParameters:{{companyId=customerId, userId=SFAPI, type=null, sourceArea=null, effectiveStartDate=null, publishedAt=1590914553205, publishedBy=SFAPI, externalAllowed=false, publishServer=serverip, priority=0, proxyId=null, serverName=null, topic=com.company.platform.mobile.deactivateuser, ptpName=null}}},body:{"companyId": "customerId", "inactiveUserId": ["E_UUU_21934","E_UUU_21935"]}} sent to https://domain/api/deactivate, (HTTP/1.1 200 OK)

提取目标：publishedAt, publishedAt of filterParameters, inactiveUserId
```

## Impl

```python
import csv
# 拿到 csv 的 _raw 列数据
context = []
rows = []
with open('dump_csv.csv', newline='') as csvfile:
   contexts = csv.reader(csvfile)
   # 使用 reader = csv.DictReader(csvfile) 的话可以使用 column name 取值
   # 例如: reader['companyId'], 不过缺点是要在 with loop 中处理完数据
   rows = [row[16] for row in contexts]
   rows = [1:]

# 分析 _raw 数据特性，决定使用正则匹配数据
# publishedAt:"(.*?)" 加 ? 表示 非贪婪
# publishedAt=(\d+)
# inactiveUserId": (\[.*?\])
# 以上表达式取 group 1 数据

import re
from datetime import datetime
# re.findAll
# re.match() 从开头开始匹配
# re.search(reg, src) 匹配任意位置

reg1 = 'publishedAt:"(.*?)"'
reg2 = 'publishedAt=(\d+)'
reg3 = 'inactiveUserId": (\[.*?\])'

rowlist = []
for row in rows:
    infolist = []
    timestr01 = re.search(reg1, row).group(1)
    d1 = datetime.strptime(timestr01, '%Y-%m-%d %H:%M:%S')
    infolist.append(d1)

    timestr2 = int(re.search(reg2, row).group(1))
    d2 = datetime.fromtimestamp(timestr2/1000.0)
    infolist.append(d2)

    users = re.search(reg3, row).group(1)
    ulist = eval(users) # string 转化为 list
    infolist.append(ulist)
    rowlist.append(infolist)

# 把数据根据时间先后排序
sortedList = sorted(rowlist, key=lambda sub: sub[1])

def printList(line):
    formatStr01 = '%y-%m-%d %H:%M:%S'
    print(line[0].strftime(formatStr01), end=' | ')
    print("%15f" % (line[1].timestamp()), end=' | ')
    arrStr = str(line[2][:5]) + "..." + str(len(line[2])) if len(line[2]) > 5 else str(line[2])
    print(arrStr)

for sub in sortedList:
    printList(sub)
```
