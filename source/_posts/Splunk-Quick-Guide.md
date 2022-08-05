---
title: Splunk 快速入门
date: 2020-09-10 15:43:42
categories:
- 小工具
tags:
- splunk
---

## 测试环境 setup

官方给出了 Splunk 的 docker image 我们可以通过它来创建本地测试环境 [docker hub link](https://hub.docker.com/r/splunk/splunk/)。就两条命令，官方也给了很详细的命令解释，赞。

```bash
docker pull splunk/splunk:latest

docker run -d -p 8000:8000 -e "SPLUNK_START_ARGS=--accept-license" -e "SPLUNK_PASSWORD=<password>" --name splunk splunk/splunk:latest
```

PS: 如果需要测试 Splunk REST API, 还需要开放其端口 -p 8089:8089。登陆系统后 Settings -> Server settings -> General settings 查看端口映射

本地测试环境运行测试用例体验比公司的快很多，体验很好

## 创建测试数据

发现一个很有意思的函数，可以用它创建简单的测试数据，使用之前最好对照官方文档看看例子，新版有支持 format 为 json, csv 的数据，旧版不行

* [makeresults](https://docs.splunk.com/Documentation/Splunk/8.1.4/SearchReference/Makeresults)

```bash
# 创建 100 条数据只显示  5% < index < 95%
| makeresults count=100
| eval random_num=random() 
| streamstats count as index
| eventstats max(index) as m_idx
| where index > m_idx*0.05 AND index < m_idx*0.95

# 创建多行
# makemv: 通过分隔符创建多个数据
# mvexpand: 多个数据展开成 event
# mv 是 multi value 的缩写
| makeresults
| eval test="buttercup rarity tenderhoof dash mcintosh fleetfoot mistmane"
| makemv delim=" " test
| mvexpand test

# random() 会产生 0-(2^31-1) 之间的随机数
# 可以使用取模的方式产生 n 以内的随机数
| makeresults
| eval n=(random() % 100)
```

## 最常用的 case - search with keyword

直接输入 keyword + enter

## 常用统计操作 stats

对整个数据集做统计，可以结合 average，count 和 sum 使用，支持 by 做分组

```bash
# 创建 name-score 测试集
| makeresults count=5
| eval name="Jerry Tom"
| makemv delim=" " name
| mvexpand name
| eval score=(random() % 100)
# 统计每个人的 score 总和，平均值
| stats sum(score) avg(score) by name
```

## stats vs eventstats vs streamstats

* [Command Type](https://docs.splunk.com/Documentation/Splunk/8.1.4/SearchReference/Commandsbytype#Streaming_commands)

* stats: transforming command, 数据集会发生改变
* eventstats: Dataset processing command, 需要所有 event 都找出来后才能工作
* streamstats: streaming command, 针对每一个 event 做操作

```bash
# 创建测试集
| makeresults count=4 
| streamstats count 
| eval age = case(count=1, 25, count=2, 39, count=3, 31, count=4, null())
| eval city = case(count=1 OR count=3, "San Francisco", count=2 OR count=4, "Seattle") 
# stats 类似 sql 中的聚合函数，计算之后，只会产出一行数据
| stats sum(age)
# eventstats 工作时并不会改变原始数据，而是在原有的 raw 数据外新增一列数据
| eventstats avg(age) by city
# streamstats 和 eventstats 很像，也是不改变原数据去新增一列的操作。
# 区别就在 stream 这个关键字，它是用流的方式处理。只在每条数据出现的节点做计算
# 本例中第一个 San Francisco 均值为自身，第二个则为两个之和做平均了，很神奇。
| streamstats  avg(age) by city
```

此外 streamstats count 还可以用来显示**行号**，很方便

```bash
| makeresults count=10
| eval x=random()
| streamstats count as index
# eventstats count 可以在每一行上增加一个总计 field
| eventstats count
```

## eval

可以对每一行做计算，支持数学计算，字符串操作和布尔表达式

```bash
# 生产 4 条数据并对 count 列做平方操纵
| makeresults count=4 
| streamstats count
| eval pow=pow(count, 2)
```

**和 eventstats 的区别**

总觉得 eval 和 eventstats 很相似，特意查了一下定义，还是有很明显的区别的。

eval 是针对每一行做计算，比如新增 field = field1 - field2

eventstats 突出以总体概念，可以做 by group 的操作，比如 sum(field1) by field2

## Comparison and Conditional functions

* [Comparison and Conditional functions Doc](https://docs.splunk.com/Documentation/SCS/current/SearchReference/ConditionalFunctions)

* case
* match
* nullif
* searchmatch
* validate
* ...

eval 配合条件函数使用可以衍生出很多效果

```bash
# 根据 status 匹配 description 信息
| makeresults
| eval status="400 300 500 200 200 201 404"
| makemv delim=" " status
| mvexpand status 
| eval description=case(status == 200, "OK", status ==404, "Not found", status == 500, "Internal Server Error")
# 检测 status 是否匹配 4xx 格式
| eval matches = if(match(status,"^4\d{2}"), 1, 0)

# 根据是否包含关键字对 query type 的 log 做细分
search cmd | eval ReqType=if(like(_raw, "%lastModified%") AND ReqType="query", "query - lastModified", ReqType)
```

## rex 抽取特定的 field

splunk 支持字符串匹配，常用案例如下，需要注意的是，他只会从 _raw 中抽取信息

```bash
| makeresults
| eval _raw="[4211fb51-6ae6-41eb-a0bf-e6dd693bbdb2] [EngineX Perf] Request takes 1116 ms"
| rex "Request takes (?<time_cost>.*) ms"
# 我们还可以通过添加 (?i) 达到忽略大小写的效果
| rex "(?i)request takes (?<time_cost>.*) ms"
```

## 收集分散在多个 event 中的数据

有两种解决方案，一种是子查询，一种是 eventstats，两种方案对向能消耗都很大。

* [join form multi events](https://community.splunk.com/t5/Splunk-Search/Joining-data-from-multiple-events-with-stats/m-p/526162)

看到一个从多条分散的 event 中收集数据的例子，刚好是我现在需要的。核心思路是通过 eventstats 为每个 event 计算一个新的 field 做跳板

这里还有一个很神奇的语法，通过 values 统计出来的结果，在 if 条件中，我们可以直接通过使用 field=values 的语法达到类似 in 的效果。找了半天文档，没发现有这种个语法说明 （；￣ェ￣）

相似的还有一个 command 叫做 [subsearch](https://docs.splunk.com/Documentation/Splunk/9.0.0/Search/Aboutsubsearches)，通过自查询缩小范围，然后进一步查询

就我处理的案例来说，subsearch performance 要小很多，通过子查询可以很快的缩小处理范围

## 行转列 transpose

如果统计结果为

| A    | B    | C    |
| :--- | :--- | :--- |
| 1    | 2    | 3    |

转化为饼图的时候，只会显示 A 类型的数据，因为 Splunk 默认使用 X 轴作为分类标的。这时可以使用 `search cmd | transpose` 达到行专列的效果

```bash
| makeresults
| eval _raw="A=2,B=5,C=8,D=1"
| extract
| table A B C D
| transpose
```

## 去重

```bash
| makeresults count=4 
| streamstats count 
| eval age = case(count=1, 25, count=2, 39, count=3, 31, count=4, null())
| eval city = case(count=1 OR count=3, "San Francisco", count=2 OR count=4, "Seattle")
| dedup city
# 或者使用 values
| stats values(city)
# 当然也可以使用 stats 计算 count 达到曲线救国的效果
| stats count by city
```

## 使用 where 达到 filter 的效果

```bash
| makeresults count=4 
| streamstats count 
| eval age = case(count=1, 25, count=2, 39, count=3, 31, count=4, null())
| eval city = case(count=1 OR count=3, "San Francisco", count=2 OR count=4, "Seattle")
| where age > 31
```

## 查询 event 的日均量

```bash
eventtype="searchAccountLocked" | timechart span=1d count | stats avg(count)

# 在此基础上，计算 7 天的平均值
eventtype="searchAccountLocked" | timechart span=1d count | stats avg(count) as avgc ｜ eval n=exact(1 * avgc)
```

## 取整数

使用 ceil 和 round 分别达到向上和向下取整

```bash
| makeresults 
| eval n=10.3
| eval n_round=round(n)
| eval n_ceil=ceil(n)
```

更多计算函数，参考 [math](https://docs.splunk.com/Documentation/Splunk/8.1.4/SearchReference/MathematicalFunctions)

## fields

指定显示结果 `search cmd | fields host, src`, 从结果集中 remove 某个 field `search cmd | fields - host, src`

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

## Splunk REST API

本地测试过了，但是产品上失败，可能公司用的 Splunk 有特殊限制

```bash
# 创建搜索 job
curl -u admin:Splunkpwd0001$ -k https://localhost:8089/services/search/jobs -d search="search ScimPerformanceInterceptor"
# 查看 job 状态
curl -u admin:Splunkpwd0001$ -k https://localhost:8089/services/search/jobs/1658910321.148
# 查看 job 结果
curl -u admin:Splunkpwd0001$ -k https://localhost:8089/services/search/jobs/1658910321.148/results --get -d output_mode=json
```

## Splunk dashboard sample

官方制作了一个 dashboard 插件，里面有大量的精美 bashboard 案例 [dashboard examples](https://splunkbase.splunk.com/app/1603/)。需要注册账号，不过是免费的，下载完成后还会给出安装步骤。

1. Log into Splunk Enterprise.
2. On the Apps menu, click Manage Apps.
3. Click Install app from file.
4. In the Upload app window, click Choose File.
5. Locate the .tar.gz file you just downloaded, and then click Open or Choose.
6. Click Upload.
7. Click Restart Splunk, and then confirm that you want to restart.

To install apps and add-ons directly into Splunk Enterprise
Put the downloaded file in the $SPLUNK_HOME/etc/apps directory.
Untar and ungzip your app or add-on, using a tool like tar -xvf (on *nix) or WinZip (on Windows).
Restart Splunk.

After you install a Splunk app, you will find it on Splunk Home. If you have questions or need more information, see Manage app and add-on objects.

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
