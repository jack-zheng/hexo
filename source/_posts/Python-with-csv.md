---
title: Python 操作 csv 文件
date: 2020-06-18 17:43:48
categories:
- python
tags:
- csv
---

## 基本操作

[qutochar, delimiter 使用详解](https://segmentfault.com/a/1190000013031439)

```python
# 写 csv 文件
# newline='' 可以在读写时移除空白行
import csv
with open('eggs.csv', 'w', newline='') as csvfile:
    spamwriter = csv.writer(csvfile)
    spamwriter.writerow(['Spam'] * 5 + ['Baked Beans'])
    spamwriter.writerow(['Spam', 'Lovely Spam', 'Wonderful Spam'])

# cat eggs.csv            
# Spam,Spam,Spam,Spam,Spam,Baked Beans
# Spam,Lovely Spam,Wonderful Spam

# 一次性写多行
header = ['name', 'area', 'country_code2', 'country_code3']
data = [
    ['Albania', 28748, 'AL', 'ALB'],
    ['Angola', 1246700, 'AO', 'AGO']
]

with open('countries.csv', 'w', encoding='UTF8', newline='') as f:
    writer = csv.writer(f)
    # write the header
    writer.writerow(header)
    # write multiple rows
    writer.writerows(data)

# 如果数据以 dict 的格式出现，可以使用 DictWriter 简化操作
fieldnames = ['name', 'area', 'country_code2', 'country_code3']
# csv data
rows = [
    {'name': 'Algeria',
    'area': 2381741,
    'country_code2': 'DZ',
    'country_code3': 'DZA'},
    {'name': 'American Samoa',
    'area': 199,
    'country_code2': 'AS',
    'country_code3': 'ASM'}
]

with open('countries.csv', 'w', encoding='UTF8', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(rows)

# 读 csv 文件
import csv
with open('some.csv', newline='') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
# ['Spam', 'Spam', 'Spam', 'Spam', 'Spam', 'Baked Beans']
# ['Spam', 'Lovely Spam', 'Wonderful Spam']
```

## 实操

有一个 csv 文件，其中有个 column 名为 '_raw' 包含我们需要的信息，写一段脚本解析之

```txt
_raw 中文本为

08:42:50,222 INFO  [RESTCallbackSubscriber] [customerId,customerId,null,null,SFAPI,null,null] [IrisSubscriber Container[queue_seb.subscriber.pillar.deactivateuser]1]Postback for event com.company.hermes.core.SFEvent={meta:Meta={priority:0,proxyId:"null",serverName:"null",topic:"com.company.platform.mobile.deactivateuser",ptpName:null,companyId:"customerId",eventId:"a3b43584-3ceb-4760-9c01-699d635f4461",type:"null",sourceArea:"null",effectiveStartDate:"null",publishedAt:"2020-05-31 08:42:39",publishBy:"SFAPI",publishServer:"serverip",externalAllowed:false,filterParameters:{{companyId=customerId, userId=SFAPI, type=null, sourceArea=null, effectiveStartDate=null, publishedAt=1590914553205, publishedBy=SFAPI, externalAllowed=false, publishServer=serverip, priority=0, proxyId=null, serverName=null, topic=com.company.platform.mobile.deactivateuser, ptpName=null}}},body:{"companyId": "customerId", "inactiveUserId": ["E_UUU_21934","E_UUU_21935"]}} sent to https://domain/api/deactivate, (HTTP/1.1 200 OK)

提取目标：publishedAt, publishedAt of filterParameters, inactiveUserId
```

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

## Issues

**2021-06-09** python+csv & shell 出问题了

Scenraio: 自动化脚本实现批量创建 Jira ticket

Issue desc: python + csv lib 组织一个数据源文件，之后使用 shell 读取，但是数据读取后，format 出问题了，会在末尾包含一个换行

Reproduce:

```python
import csv
with open('eggs.csv', 'w', newline='') as csvfile:
    spamwriter = csv.writer(csvfile)
    spamwriter.writerow(['Spam', 'Baked Beans'])
```

使用 cat 或者 sed 查看，可以看到末尾包含一个 `\r` 换行符号

```sh
cat -v eggs.csv 
# Spam,Baked Beans^M
sed -n 'l' eggs.csv                                                      
# Spam,Baked Beans\r$
```

在 sh 脚本中我会解析这个 csv 文件并使用解析得到的内容作为后续操作的输入.

重现的脚本中，我们拿到解析的 csv 内容并打印出来。打印内容分别加了前后坠便于观察。可以看到第二个 echo 的后缀打印会出问题

```sh
cat reproduce.sh 
#!/usr/local/bin/bash
while IFS=',' read -r col1 col2
do
  echo --$col1---
  echo --$col2---
done < eggs.csv
echo "Finish reproduce script..."

./reproduce.sh
# --Spam---
# ---aked Beans
# Finish reproduce script...
```

将 `echo --$col2---` 换成 `echo "--$col2---" | sed -n 'l'` 之后再次运行 reproduce.sh 输出如下

```sh
./reproduce.sh
# --Spam---
# --Baked Beans\r---$
# Finish reproduce script...
```

回头细想了一下 `read -r` 是不会读取结尾没有换行的行的，在这个例子中 `\r---` 已经是下一行了，而且没有换行，自动跳过了（；￣ェ￣）

Solution: 我的解决方案是，不用 python 的 csv write 方法，直接把解析的过程用基础的 `file.write()` 完成，一行写完自己写 `\n` 做换行即可
