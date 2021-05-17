---
title: Shell 范例
date: 2021-05-17 12:24:54
categories:
- 小工具
tags:
- shell
---

## 过滤文件

任务描述：

有一个 git repo 存储了所有的测试文件(java 格式)，已知需要筛选的文件列表，筛选出所有文件中包含 'DB' (忽略大小写)的文件。并列出包含关键字的列，以便人工删选。

原始文件名列表格式如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<results>
  <testcase external_id='PLT#-123456950'>
    <result>p</result>
    <notes />
  </testcase>
  ...
</results>
```

使用编辑器的快捷功能将 `#-` 去掉

PS: 查询一下 sed, awk 和内置的字符串处理方式

repo 中文件名格式如下：PLT123457725UTF8ExportTemplateUserRecordWithUTF8UserImportAndShownCorrectEncoding

使用 find + grep 删选哪些文件包含 db 操作

```shell
find . -iname 'PLT123457726*.java' -exec grep -i "dbUtil\|DB" {} \;
# "Priorities.P1", "DBType.Oracle", "all", "TestType.Regression", "systemUltra", "systemv12",
# String ssn = dbUtil.getSingleUserFieldValueByUserIdFromSMUserInfoTable(PLT123457670, SSN);

find . -iname 'PLT123457726*.java' -exec grep -i "dbUtil\|db" {} \; | grep -v 'DBType'
# String ssn = dbUtil.getSingleUserFieldValueByUserIdFromSMUserInfoTable(PLT123457670, SSN);
```

将文件中的 case name 删选出来 `grep -E -o 'PLT.*[0-9]+' bulk.xml >> bulk2.xml`

遍历每一行，做 query

```shell
file="./bulk2.xml"
while IFS= read line

do
    echo "------------- start -------------"
    # write file name to file
    find . -iname "$line*.java" -exec basename {} \;
    echo '------------- match start -------------'
    # write matched line to file
    find . -iname "$line*.java" -exec grep -i "dbUtil\|db" {} \; | grep -v "DBType"
    echo '------------- match end -------------'
done <"$file"
```

PS: 由于技术手段的缺失，很多函数我都不清楚怎么调用，对 shell 还是很不熟悉，需要更多的实践练习

## 问题

* shell 文件里面写 `echo -n ','` 会把 `-n` 也写进去？
* 因为每次执行都会扫一遍 repo，所以想能不能建立一个 name-context map 到内存然后扫描
* 以上面的 idea 为基础，能不能实时监测 shell 运行时的内存消耗
* 很多类似技术的使用案例
* read/write to file in shell