---
title: 定制 Shell 脚本
date: 2021-05-17 12:24:54
categories:
- Shell
tags:
- shell
---

## 定制 git pull

PS: 写完之后顺便看看 .zsh_history 是怎么实现的

git pull 算是开发时经常用到的一个命令了，但是更新代码之后，时常会遇到最新版本的代码不 work 的情况。由此，打算对 pull 命令做一个优化，每次 pull 的时候，将当前 repo 的信息写到历史记录中，类似 `.zsh_history` 的功能。

```bash
# 等看过 Shell 的 condition 篇再完善
function git {
  if [[ "$1" == "pull" && "$@" != *"--help"* ]]; then
    version_before=$(git rev-parse --short HEAD)
    echo "bef: %s" $version_before
    command git pull
    version_after=$(git rev-parse --short HEAD)
    echo "aft: %s" $version_after
    if [ "$v1" == "$v2" ]; then
      echo "No history recorded..."
    else
      echo "Record history..."
      # write process date time
      printf "%s  " "$(date '+%Y-%m-%d %H:%M:%S')" >> ~/.git_pull.history
      # write current version for recover
      printf "%s  " "$version_before"  >> ~/.git_pull.history
      # write repo name
      printf "%s\n" "$(git config --get remote.origin.url | cut -d '/' -f 2)"  >> ~/.git_pull.history
    fi
    echo end loop1
  else
    echo loop2
    command git "$@"    # command 用于调用外部命令
  fi
}
```

* [Stackoverflow](https://stackoverflow.com/questions/3538774/is-it-possible-to-override-git-command-by-git-alias)

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
* read/write to file in shell