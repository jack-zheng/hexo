---
title: Splunk 快速入门
date: 2020-09-10 15:43:42
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

## makeresults

发现一个很有意思的函数，可以用它创建简单的测试数据，使用之前最好对照官方文档看看例子，新版有支持 format 为 json, csv 的数据，旧版不行

* [makeresults](https://docs.splunk.com/Documentation/Splunk/8.1.4/SearchReference/Makeresults)

## 行转列 transpose

如果统计结果为

| A    | B    | C    |
| :--- | :--- | :--- |
| 1    | 2    | 3    |

转化为饼图的时候，只会显示 A 类型的数据，因为 Splunk 默认使用 X 轴作为分类标的。这时可以使用 `search cmd | transpose` 达到行专列的效果

## stats vs eventstats vs streamstats

stats 类似 sql 中的聚合函数，计算之后，只会产出一行数据

```search
| makeresults count=4 
| streamstats count 
| eval age = case(count=1, 25, count=2, 39, count=3, 31, count=4, null())
| eval city = case(count=1 OR count=3, "San Francisco", count=2 OR count=4, "Seattle") 
| stats sum(age)
```

eventstats 工作时并不会改变原始数据，而是在原有的 raw 数据外新增一列数据

```search
| makeresults count=4 
| streamstats count 
| eval age = case(count=1, 25, count=2, 39, count=3, 31, count=4, null())
| eval city = case(count=1 OR count=3, "San Francisco", count=2 OR count=4, "Seattle")
| eventstats avg(age) by city
```

streamstats 和 eventstats 很像，也是不改变原数据去新增一列的操作，区别就在 stream 这个关键字，它是用流的方式处理。只在每条数据出现的节点做计算

```search
| makeresults count=4 
| streamstats count 
| eval age = case(count=1, 25, count=2, 39, count=3, 31, count=4, null())
| eval city = case(count=1 OR count=3, "San Francisco", count=2 OR count=4, "Seattle")
| streamstats  avg(age) by city
```

还是相同的案例，用 streamstats 计算平均值时，第一个 San Francisco 均值为自身，第二个则为两个之和做平均了，很神奇。

此外 streamstats count 还可以用来显示**序号**

## 去重

可以使用 dedup

```search
| makeresults count=4 
| streamstats count 
| eval age = case(count=1, 25, count=2, 39, count=3, 31, count=4, null())
| eval city = case(count=1 OR count=3, "San Francisco", count=2 OR count=4, "Seattle")
| dedup city
```

当然也可以使用 stats 计算 count 达到曲线救国的效果

```search
search result 
| stats count by city

或者
| stats values(city)
```

## eval

通过 eval 可以在原有数据的基础上，通过计算新增一列数据，是 stream 类型的 command。比如我们想要添加 flag 列，条件为 responseTime > 5000 的值为 1 否则为 0. search 语句可以这样写 `search command | eval Status=if(responseTime > 10000, 1, 0)`

## 查询 event 的日均量

```bash
eventtype="searchAccountLocked" | timechart span=1d count | stats avg(count)

# 在此基础上，计算 7 天的平均值
eventtype="searchAccountLocked" | timechart span=1d count | stats avg(count) as avgc ｜ eval n=exact(1 * avgc)
```

## table + where 达到 filter 的效果

pipeline 后面接 where 语句可以起到 filter 的效果，和 SQL 一样

## 收集分散在多个 event 中的数据

* [join form multi events](https://community.splunk.com/t5/Splunk-Search/Joining-data-from-multiple-events-with-stats/m-p/526162)

看到一个从多条分散的 event 中收集数据的例子，刚好是我现在需要的。核心思路是通过 eventstats 为每个 event 计算一个新的 field 做跳板

这里还有一个很神奇的语法，通过 values 统计出来的结果，在 if 条件中，我们可以直接通过使用 field=values 的语法达到类似 in 的效果。找了半天文档，没发现有这种个语法说明 （；￣ェ￣）

想过相似的还有一个 command 叫做 [subsearch](https://docs.splunk.com/Documentation/Splunk/9.0.0/Search/Aboutsubsearches)，通过自查询缩小范围，然后进一步查询

就我处理的案例来说，subsearch performance 要小很多，通过子查询可以很快的缩小处理范围

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

Steps:

1. clone git 开源项目 [Splunk SDK Python](https://github.com/splunk/splunk-sdk-python)
2. 用户目录下创建 `.splunkrc` 文件
3. cd 到 `splunk-sdk-python/examples` folder 下，运行命令 `python search.py "search * | head 10" --earliest_time="2011-08-10T17:15:00.000-07:00" --rf="desc" --output_mode=json` 可以看到对应时间戳下的前 10 条记录

`.splunkrc` 文件模板

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
