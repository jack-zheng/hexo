---
title: LSCASSB partii shell scripting basics
date: 2021-05-24 11:04:47
categories:
- Shell
tags:
- Linux命令行与shell脚本编程大全 3rd
- TODO
---

Linux命令行与shell脚本编程大全 3rd 第二部分命令实验记录

## Chapter 11: Basic Script building

### Using Multiple Commands

使用分号分隔命令

```bash
date ; who
# Mon May 24 12:36:11 CST 2021
# i306454  console  May 24 11:18 
# i306454  ttys000  May 24 11:18 
# i306454  ttys003  May 24 11:33
```

### Creating a Script File

1. 新建文件 mysh.sh
2. 填入内容
3. 设置环境
4. 运行

> However, the fi rst line of a shell script fi le is a special case, and the pound sign followed by the exclamation point tells the shell what shell to run the script under
> Shell script 的第一行表示你想要用哪个 Shell 运行你的脚本

```sh
#!/bin/bash
# This scri displays the date and who's logged on
date
who
```

尝试运行 `mysh.sh`，运行失败 `bash: mysh.sh: command not found`

这是你有两种选择

1. 将包含脚本的目录添加到 PATH 中，eg: `export PATH=$PATH:path_to_folder`
2. 使用相对或绝对路径调用脚本, eg: `./mysh.sh`

PS: 发现直接用 `sh mysh.sh` 即可，还省去了赋权的操作

这里直接使用相对路径调用 `./mysh.sh`, 运行失败 `bash: ./mysh.sh: Permission denied`。通过 `ls -l mysh.sh` 查看权限, 发现并没有权限，然后赋权,再次尝试，运行成功。

```sh
ls -l mysh.sh 
# -rw-r--r--  1 i306454  staff  70 May 24 12:40 mysh.sh

chmod u+x mysh.sh 
./mysh.sh 
# Mon May 24 13:11:37 CST 2021
# i306454  console  May 24 11:18 
# i306454  ttys000  May 24 11:18
```

### Displaying Messages

使用 `echo` 打印信息

* 当输出内容中没有什么特殊符号时，可以直接在 echo 后面接你要的内容
* 当输出内容中包含单/双引号时，需要将 echo 的内容包含在 双/单引号中
* 默认 echo 是会换行的，使用 `echo -n xxx` 取消换行

### Using Variables

`set`: 打印当前环境的所有环境变量, 输出内容包括 PATH，HOME 等

```sh
set
# ANT_HOME=/Users/i306454/SAPDevelop/tools/apache-ant-1.8.4
# BASH=/usr/local/bin/bash
# BASHOPTS=checkwinsize:cmdhist:complete_fullquote:expand_aliases:extquote:force_fignore:globasciiranges:hostcomplete:interactive_comments:progcomp:promptvars:sourcepath
# BASH_ALIASES=()
# ...
```

在你的 sh 脚本中，可以调用这些变量

```sh
#!/bin/bash
echo "User info for userid $USER"
echo UID: $UID
echo HOME: $HOME

sh test_var.sh 
# User info for userid i306454
# UID: 501
# HOME: /Users/i306454
```

当你在输出语句中想要打印 `$` 这个特殊字符时，只需要在前面加斜杠

```sh
echo "This book cost $15"
# This book cost 5
echo "This book cost \$15"
# This book cost $15
```

PS: `${variable}` 也是合法的，这种声明起到强调变量名的作用

变量名限制：最多 20 个字符，可以是英文，数组或者下划线，区分大小写。使用等号链接，中间**不允许**有空格

> Command substitution

你可以通过一下方式将 cmd 输出的值赋给你的变量

1. `\`` 反引号符(backtick character)
2. `$()` 表达式

```sh
tdate=`date`
echo $tdate
# Mon May 24 15:46:51 CST 2021
tdate2=$(date)
echo $tdate2
# Mon May 24 15:47:39 CST 2021
```

这只是初级用法，更骚的操作是将输出定制之后用作后续命令的输出，比如下面这个例子，将文件夹下的所有目录写进 log 文件，并用时间戳做后缀 `today=$(date +%y%m%d); ls -al /usr/bin > log.$today`

**Caution:** Command substitution creates what’s called a subshell to run the enclosed command. A subshell is a separate child shell generated from the shell that’s running the script. Because of that, any variables you create in the script aren’t available to the subshell command.

Command substitution 这种使用方法会创建一个子 shell 计算变量值，子 shell 运行的时候是看不到你外面的 shell 中定义的变量的

## Redirecting Input and Output

> Output redirection

可以使用大于号(greater-than symbol)将输出导入文件

```sh
date > test6
ls -l test6
# -rw-r--r--  1 i306454  staff  29 May 24 18:31 test6
cat test6
# Mon May 24 18:31:40 CST 2021
```

如果文件已经存在，则覆盖原有内容

```sh
who > test6
cat test6
# i306454  console  May 24 11:18 
# i306454  ttys000  May 24 11:18 
# i306454  ttys003  May 24 18:00
```

使用两个大于号(double greater-than symbol)来做 append 操作

```sh
date >> test6
cat test6
# i306454  console  May 24 11:18 
# i306454  ttys000  May 24 11:18 
# i306454  ttys003  May 24 18:00 
# Mon May 24 18:35:12 CST 2021
```

> Input redirection

使用小于号(less-htan symbol)将文件中的内容导入输入流 `command < inputfile`

```sh
wc < test6
#    4      21     125
# wc: show lines, words and bytes of input
```

除了从文件导入，从命令行直接导入多行也是可行的，术语叫做 `inline input redirection`。这个之前在之前阿里云 setup 环境的时候操作过的。使用两个小于号(<<) + 起止描述符实现

```sh
wc << EOF
> test String 1
> test String 2
> test String 3
> EOF
# 3       9      42
```

### Pipes

使用方式 `command1 | command2`

> Don’t think of piping as running two commands back to back. The Linux system actually runs both commands at the same time, linking them together internally in the system. As the fi rst command produces output, it’s sent immediately to the second command. No inter-mediate fi les or buffer areas are used to transfer the data.
> pipe 中的命令是同时执行的，变量传递不涉及到中间变量

```sh
ls 
# input.txt       mysh.sh         output.txt      test_echo.sh
# log.210524      out.txt         test6           test_var.sh
ls | sort
# input.txt
# log.210524
# mysh.sh
# ...
```

pipe 可以无限级联 `cmd1 | cmd2 | cmd3 | ...`

### Performing Math

Bash 中提供了两种方式进行计算

> The expr command

```sh
expr 1 + 5
# 6
```

expr 支持常规算数运行 + 与或非 + 正则 + 字符串操作等

这个 expr 表达式有点尴尬，他的一些常规运算是要加反斜杠的，简直无情

```sh
expr 1 * 6
# expr: syntax error
expr 1 \* 6
# 6
```

这个还算好的，如果在 sh 文件中调用，表达式就更操蛋了

```sh
#!/usr/local/bin/bash
var1=10
var2=20
var3=$(expr $var2 / $var1)
echo The result is $var3

sh test_expr.sh 
# The result is 2
```

> Using brackets

Bash 中保留 `expr` 是为了兼容 Bourne shell,同时它提供了一种更简便的计算方式，`$[ operation ]`

```sh
var1=$[1 + 5]
echo $var1
# 6
var2=$[ $var1 * 2 ]
echo $var2
# 12
```

方括号表达式可以自动完成计算符号的识别，不需要用反斜杠做转义符，唯一的缺陷是，他只能做整数运行

```sh
var1=100
var2=45
var3=$[$var1/$var2]
echo $var3
# 2
```

> A floating-point solution

bc: bash calculation, 他可以识别一下内容

* Number(integer + floating point)
* Variables(simple variables + arrays)
* Comments(# or /* ... */)
* Expressions
* Programming statements(such as if-then statements)
* Functions

```sh
bc
bc 1.06
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
12 * 5.4
64.8
3.156 * (3 + 5)
25.248
quit
```

可以通过 scale 关键字指定精确位, 默认 scale 为 0, `-q` 用于跳过命令文字说明

```sh
bc -q 
3.44 / 5
0
scale=4
3.44 / 5
.6880
quit
```

如前述，bc 可以识别变量

```sh
bc -q
var1=10
var1 * 4
40
var2 = var1 / 5
print var2
2
quit
```

你可以通过 Command substitution，在 sh 脚本中调用 bc, 形式为 `variable=$(echo "options; expression" | bc)`

```sh
cat test9
# #!/usr/local/bin/bash
# var1=$(echo "scale=4; 3.44/5" | bc)
# echo This answer is $var1

sh test9
# This answer is .6880
```

脚本中定义的变量也可以使用

```sh
cat test10
# #!/usr/local/bin/bash
# var1=100
# var2=45
# var3=$(echo "scale=4; $var1 / $var2" | bc)
# echo The answer for this is $var3bash-5.1$ 
sh test10
# The answer for this is 2.2222
```

当遇到很长的计算表达式时，可以用 << 将他们串起来

```sh
cat test12
# #!/usr/local/bin/bash
# var1=10.46
# var2=43.67
# var3=33.2
# var4=71

# var5=$(bc << EOF
# scale = 4
# a1 = ($var1 * $var2)
# b1 = ($var3 * $var4)
# a1 + b1
# EOF
# )

# echo The final answer for this mess is $var5
sh test12
# The final answer for this mess is 2813.9882
```

PS: 在上面的脚本中我们用了 Command substitution 所以中间的变量前面都是要加 `$` 的，当终端调用 `bc` 时就不需要了

### Exiting the Script

There's a more elegant way of completing things available to us. 每个命令结束时，系统都会分配一个 0-255 之间的整数给他

> Checking the exit status

Linux 使用 `$?` 表示上一条命令的执行状态

```sh
date
# Mon May 24 19:47:29 CST 2021
echo $?
# 0
```

运行正常，返回 0。如果有问题，则返回一个非零

```sh
asd
# bash: asd: command not found
echo $?
# 127
```

常见错误码表

| Code  | Desc                          |
| :---- | :---------------------------- |
| 0     | Success                       |
| 1     | General unknown error         |
| 2     | Misuse of shell command       |
| 126   | The cmd can't execute         |
| 127   | Cmd not found                 |
| 128   | Invalid exit argument         |
| 128+x | Fatal err with Linux singal x |
| 130   | Cmd terminated with Ctrl+C    |
| 255   | Exit status out of range      |

> The exit command

exit 关键字让你可以定制脚本的返回值

```sh
cat test13
# #!/usr/local/bin/bash
# # testing the exit status
# var1=10
# var2=20
# var3=$[$var1 + $var2]
# echo The answer is $var3
# exit 5

chmod u+x test13
./test13
# The answer is 30
echo $?
# 5
```

PS: 这里看出来差别了，如果我直接用 sh test13 执行的话，结果是 0

这里需要指出的是，exit code 最大为 255 如果超出了，系统会自己做修正

```sh
cat test14
# #!/usr/local/bin/bash
# # exit code more than 255
# var=300
# exit $var

chmod u+x test14
./test14
echo $?
# 44
```

## Chapter 12: Using Structured Commands