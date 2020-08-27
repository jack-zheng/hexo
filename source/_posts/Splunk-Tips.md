---
title: Splunk API 使用记录
date: 2020-07-14 17:34:02
categories:
- 小工具
tags:
- splunk
---

## 常用 Query

```sql
-- event 出现的次数
search=index | stats sum(linecount) as Total
-- or
search=index | stats count as Total
```

### 查询 event 的日均量

```bash
eventtype="searchAccountLocked" | timechart span=1d count | stats avg(count)

# 在此基础上，计算 7 天的平均值
eventtype="searchAccountLocked" | timechart span=1d count | stats avg(count) as avgc ｜ eval n=exact(1 * avgc)
```

## 通过 Regex 匹配得到目标百分比

* [社区类似问题](https://community.splunk.com/t5/Splunk-Search/Get-percentage-of-matchin-to-all-events/td-p/39113)

```bash
# 搜索全部 event, 通过 regex 匹配到目标，计算百分比
UserChangeEvent MessageBox | stats count(eval(match(field1, ".*updatedFields\":\[{\"fieldName\":\"jobCode\".*"))) as JCEvent count as total | eval JC_pct=JCEvent/total*100

# 升级版，计算多个百分比情况
# EMP_PCT 内容是空的 event 在所有 jobcode event 中的占比，和 jobcode 在所有 event 中的占比
UserChangeEvent MessageBox | stats count(eval(match(field1, ".*updatedFields\":\[{\"fieldName\":\"jobCode\".*"))) as totalJCEvent count(eval(match(field1, ".*updatedFields\":\[{\"fieldName\":\"jobCode\",\"fieldType\":\"java.lang.String\",\"afterValue\":\"\"}\].*"))) as emptyEvent count as total | eval EMP_PCT=emptyEvent/totalJCEvent*100, JC_PCT=totalJCEvent/total*100
```

## 取两位小数

```bash
| 7xAVG=round((7*total/1), 2)
```

## 通过正则创建新 field

```bash
# 选出结果集，从输出信息中匹配 'Company: ' 开头 ', total CommonField' 结尾的部分并命名为 cname 统计出现次数
# _raw 表示 record 内容
search condition | rex field=_raw "Company: (?<cname>.*), total CommonField" | stats count by cname
```

## stats 和 eval 的区别

stats 是对已经有的 field 的删选，而 eval 是通过已有的 field 计算出新的 field 加到结果集中进行删选，等价于新增 field

```bash
# 删选 event, 新建一个 field 名叫 is_prod, 当 host 匹配 pattern 时赋值 yes_prod
search event | eval is_prod=if(like(host, "pc%"), "yes_prod", "not_prod") | stats count by is_prod
```

其中 eval 还支持多种删选条件，可塑性好高

```bash
# 统计各环境的 event 数量并统计比例
search event | eval env=case(like(host, "pc%"), "prod", like(host, "sc%"), "prov", like(host, "*"), "others") | stats count by env
```

## Splunk SDK

尝试了 python 版本的 SDK，香！

参考 [官方文档](https://dev.splunk.com/enterprise/docs/python/sdk-python/examplespython/commandline) 下载依赖，在本地配置 `.splunkrc` 文件写入连接信息方便调用。第一次用的时候密码配错了，还以为内网不可用，需要用 vlab，再测试的时候发现了这个问题。总的来说很可以。

Steps:

1. clone git 开源项目 [Splunk SDK Python](https://github.com/splunk/splunk-sdk-python)
2. 用户目录下创建 `.splunkrc` 文件
3. cd 到 `splunk-sdk-python/examples` folder 下，运行命令 `python search.py "search * | head 10" --earliest_time="2011-08-10T17:15:00.000-07:00" --rf="desc" --output_mode=json` 可以看到对应时间戳下的前 10 条记录

.splunkrc 文件模板

```config
# Splunk host (default: localhost)
host=xxx.xxx.xxx
# Splunk admin port (default: 8089)
port=8089
# Splunk username
username=jack
# Splunk password
password=mypwd
# Access scheme (default: https)
scheme=https
# Your version of Splunk (default: 5.0)
version=7.1.2
```
   

## 三个小例子快速入门

### 搜索 event 并通过饼图展示

1. 输入时间节点和关键词：`MessageBox topic=com.successfactors.usermanagement.event.UserChangeEvent | stats count by servername`
2. 选择可视化 tab
3. 选择饼图

![饼图](pie.png)

### 显示每天的 event 量

1. 选择时间
2. 输入搜索条件: `MessageBox topic=com.successfactors.usermanagement.event.UserChangeEvent | timechart count span=1d`
3. 选择图形

![柱状图](bar.png)

### 通过正则删选 event 并计算百分比

1. 选择时间
2. 输入删选条件: `MessageBox topic=com.successfactors.usermanagement.event.UserChangeEvent | stats count as total count(eval(match(field1, "companyId"))) as containsCID | eval CID_PCT=round(containsCID/total*100, 2)`

![百分比表](regex.png)
