---
title: Linux命令行与shell脚本编程大全 第二章 Shell 脚本基础知识
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

1. `` ` `` 反引号符(backtick character)
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

### Redirecting Input and Output

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

本章内容主要包括 loigc flow control 部分

> Working with the if-then Statement

if-then 是最基本的控制方式，format 如下, 判断依据是 command 的 exit code, 如果是 0 则表示 success，其他的则为 fail.

```sh
if command
then 
    command
fi
```

positive sample 如下

```sh
cat test1.sh 
# #!/usr/local/bin/bash
# # testing the if statement
# if pwd
# then
#     echo "It worked"
# fi
chmod u+x test1.sh 
./test1.sh
# /Users/i306454/gitStore/hexo
# It worked
```

negative sample 如下

```sh
cat test2.sh 
# # #!/usr/local/bin/bash
# # # testing a bad command
# if IamNotACommand
# then
#     echo "It worked"
# fi
# echo "We are outside the if statement"
chmod u+x test2.sh
./test2.sh 
# ./test2.sh: line 3: IamNotACommand: command not found
# We are outside the if statement
```

PS: `if-then` 可以改一下 format，将 if-then 写在一行，看上去更贴近其他语言的表现形式

```sh
if command; then
    commands
fi
```

then 中可以写代码段，如下

```sh
cat test3.sh
# #!/usr/local/bin/bash
# # testing multiple commands in the then section
# #
# testuser=MySQL
# #
# if grep $testuser /etc/passwd
# then
#     echo "This is my first command"
#     echo "This is my second command"
#     echo "I can even put in other commands beside echo:"
#     ls -a /home/$testuser/.b*
# fi
./test3.sh 
# _mysql:*:74:74:MySQL Server:/var/empty:/usr/bin/false
# This is my first command
# This is my second command
# I can even put in other commands beside echo:
# ls: /home/MySQL/.b*: No such file or directory
```

PS：由于是 mac 系统，有点出入，但是目的还是达到了。顺便测了一下缩进，将 echo 的缩进去了一样 work

> Exploring the if-then-else Statement

格式如下, 

```sh
if command
then
    commands
else
    commands
fi
```

改进 test3.sh 如下

```sh
# #!/usr/local/bin/bash
# # testing multiple commands in the then section
# #
# testuser=NoSuchUser
# #
# if grep $testuser /etc/passwd
# then
#     echo "This is my first command"
#     echo "This is my second command"
#     echo "I can even put in other commands beside echo:"
#     ls -a /home/$testuser/.b*
# else
#     echo "The user $testuser does not exist on this system."
#     echo
# fi
./test4.sh 
# The user NoSuchUser does not exist on this system.
```

> Nesting if

```sh
if command1
then
    commands
elif command2
then
    more commands
fi
```

写脚本检测帐户是否存在，然后检测用户文件夹是否存在

```sh
cat test5.sh
# #!/usr/local/bin/bash
# # testing nested ifs - use elif
# #
# testuser=NoSuchUser
# #
# if grep $testuser /etc/passwd
# then
#     echo "The user $testuser exists on this system"
# elif ls -d /home/$testuser
# then
#     echo "The user $testuser does not exist on this system"
#     echo "However, $testuset has directory."
# fi
chmod u+x test5.sh 
./test5.sh 
# ls: /home/NoSuchUser: No such file or directory
```

**Tips**: Keep in mind that, with an elif statement, any else statements immediately following it are for that elif code block. They are not part of a preceding if-then statement code block.(elif 之后紧跟的 else 是一对的，它不属于前面的 if-then)

多个 elif 串连的形式

```sh
if command1
then
    command set 1
elif command2
then
    command set 2
elif command3
then    
    command set 3
elif command4
then
    command set 4
fi
```

> Trying the test Command

if-then 条件判断只支持 exit code，为了使它更通用，Linux 提供了 test 工具集，如果 test 判定结果为 TRUE 则返回 0 否则非 0，格式为 `test condition`, 将它和 if-then 结合，格式如下

```sh
if test condition
then
    commands
fi
```

如果没有写 condition，则默认为非 0 返回值

```sh
cat test6.sh 
# #!/usr/local/bin/bash
# # Testing the test command
# #
# if test
# then
#     echo "No expression return a True"
# else
#     echo "No expression return a False"
# fi
./test6.sh 
# No expression return a False
```

将测试条件替换为变量，输出 True 的语句

```sh
cat test6.sh 
# #!/usr/local/bin/bash
# # Testing the test command
# my_variable="Full"
# #
# if my_variable
# then
#     echo "The $my_variable expression return a True"
# else
#     echo "The $my_variable expression return a False"
# fi
./test6.sh 
# The Full expression return a False

# replace my_variable=""
./test6.sh 
# The  expression return a False
```

测试条件还可以简写成如下形式

Be careful; you **must have a space** after the first bracket and a space before the last bracket, or you’ll get an error message.

```sh
if [ condition ]
then
    commands
fi
```

test condition 可以测试如下三种情景

* Numeric comparisons
* String comparisons
* File comparisions

> Using numeric comparisons

| Comparison | Description                             |
| :--------- | :-------------------------------------- |
| n1 -eq n2  | Check if n1 is equal to n2              |
| n1 -ge n2  | Check if n1 greater than or equal to n2 |
| n1 -gt n2  | Check if n1 is greater than n2          |
| n1 -le n2  | Check if n1 is less than or equal to n2 |
| n1 -lt n2  | Check if n1 less than n2                |
| n1 -ne n2  | Check if n1 is not equal to n2          |

test condition 对变量也是有效的，示例如下

```sh
cat numeric_test.sh 
# #!/usr/local/bin/bash
# # Using numeric test eveluations
# value1=10
# value2=11
# #
# if [ $value1 -gt 5 ]
# then
#     echo "The test value $value1 is greater than 5"
# fi
# #
# if [ $value1 -eq $value2 ]
# then echo "The values are equal"
# else 
#     echo "The values are different"
# fi
./numeric_test.sh 
# The test value 10 is greater than 5
# The values are different
```

但是 test condition 有个缺陷，它不能测试浮点型数据

```sh
value1=5.55
[ $value1 -gt 5 ]; echo $?
# bash: [: 5.55: integer expression expected
# 2
```

**Caution:** Remember that the only numbers the bash shell can handle are integers.

> Using string comparisons

| Comparison   | Description                                  |
| :----------- | :------------------------------------------- |
| str1 = str2  | Check if str1 is the same as string str2     |
| str1 != str2 | Check if str1 is not the same as string str2 |
| str1 < str2  | Check if str1 is less than str2              |
| str1 > str2  | Check if str1 is greater than string str2    |
| -n str1      | Check if str1 has a length greater than zero |
| -z str1      | Check if str1 has a length of zero           |

```sh
cat test8.sh 
# #!/usr/local/bin/bash
# # Testing string equality
# testuser=baduser
# #
# if [ $USER != $testuser ]
# then
#     echo "This is $testuser"
# else
#     echo "Welcome $testuser"
# fi
./test8.sh 
# This is baduser
```

在处理 `<` 和 `>` 时，shell 会有一些很奇怪的注意点

* `<` 和 `>` 必须使用转义符号，不然系统会把他们当作流操作
* `<` 和 `>` 的用法和 sort 中的用法时不一致的

针对第一点的测试，测试中，比较符号被当成了流操作符，新生产了一个 hockey 文件。这个操作 exit code 是 0 所以执行了 true 的 loop. 你需要将条件语句改为 `if [ $val1 \> $val2 ]` 才能生效

```sh
cat badtest.sh 
# #!/usr/local/bin/bash
# # mis-using string comparisons
# val1=baseball
# val2=hockey
# #
# if [ $val1 > $val2 ]
# then
#     echo "$val1 is greater than $val2"
# else
#     echo "$val1 is less than $val2"
# fi
/badtest.sh 
# baseball is greater than hockey
ls
# badtest.sh      hockey
```

针对第二点，sort 和 test 对 string 的比较是相反的

```sh
cat test9b.sh 
# #!/usr/local/bin/bash
# # testing string sort order
# val1=Testing
# val2=testing
# #
# if [ $val1 \> $val2 ]
# then
#     echo "$val1 is greater than $val2"
# else
#     echo "$val1 is less than $val2"
# fi
./test9b.sh 
Testing is less than testing

sort << EOF
> Testing
> testing
> EOF
Testing
testing
```

PS: 这里和书本上有出入，我在 MacOS 里测试两者是一致的，大写要小于小些，可能 Ubantu 上不一样把，有机会可以测一测

**Note:** The test command and test expressions use the standard mathematical comparison symbols for string compari-sons and text codes for numerical comparisons. This is a subtle feature that many programmers manage to get reversed. If you use the mathematical comparison symbols for numeric values, the shell interprets them as string values and may not produce the correct results.

test condition 的处理模式是 数字 + test codes(-eq); string + 运算符(</>/=)，刚好是交叉的，便于记忆

对 `-n` 和 `-z` 进行测试，undefined 的变量默认长度为 0

```sh
cat test10.sh 
# #!/usr/local/bin/bash
# # testing string length
# val1=testing
# val2=''
# #
# if [ -n $val1 ]
# then
#     echo "The string '$val1' is not empty"
# else
#     echo "The string '$val1' is empty"
# fi
# #
# if [ -z $val2 ]
# then
#     echo "The string '$val2' is empty"
# else
#     echo "The string '$val2' is not empty"
# fi
# #
# if [ -z $val3 ]
# then
#     echo "The string '$val3' is empty"
# else
#     echo "The string '$val3' is not empty"
# fi
./test10.sh 
# The string 'testing' is not empty
# The string '' is empty
# The string '' is empty
```

> Using file comparisons

| Comparison      | Description                                                                |
| :-------------- | :------------------------------------------------------------------------- |
| -d file         | Check if file exists and is a directory                                    |
| -e file         | Check if file or directory exists                                          |
| -f file         | Check if file exists and is a file                                         |
| -r file         | Check if file exists and is readable                                       |
| -s file         | Check if file exists and is not empty                                      |
| -w file         | Check if file exists and is writable                                       |
| -x file         | Check if file exists and is executable                                     |
| -O file         | Check if file exists and is owned by the current user                      |
| -G file         | Check if file exists and the default group is the same as the current user |
| file1 -nt file2 | Check if file1 is newer than file2                                         |
| file1 -ot file2 | Check if file1 is older than file2                                         |

测试范例如下

```sh
mkdir test_folder
[ -d test_folder ] && echo true || echo false
# true
[ -e test_folder ] && echo true || echo false
# true
[ -e xxx ] && echo true || echo false
# false

chmod u+xrw test_file.sh 
ls -l test_file.sh 
# -rwxr--r--  1 i306454  staff  190 May 25 17:51 test_file.sh
[ -r test_file.sh ] && echo true || echo false
# true
[ -w test_file.sh ] && echo true || echo false
# true
[ -x test_file.sh ] && echo true || echo false
# true

chmod u-rxw test_file.sh 
ls -l test_file.sh 
# ----r--r--  1 i306454  staff  190 May 25 17:51 test_file.sh
[ -r test_file.sh ] && echo true || echo false
# false
[ -w test_file.sh ] && echo true || echo false
# false
[ -x test_file.sh ] && echo true || echo false
# false

touch tmp_file
[ -s tmp_file ] && echo true || echo false
# false
echo new line >> tmp_file 
[ -s tmp_file ] && echo true || echo false
# true

ls -l
# total 8
# -rw-r--r--  1 i306454  staff  9 May 25 17:59 tmp_file
# -rw-r--r--  1 i306454  staff  0 May 25 18:01 tmp_file2
[ tmp_file2 -nt tmp_file ] && echo true || echo false
# true
[ tmp_file -nt tmp_file2 ] && echo true || echo false
# false
[ tmp_file -ot tmp_file2 ] && echo true || echo false
# true
[ tmp_file2 -ot tmp_file ] && echo true || echo false
# false
```

### Considering Compound Testing

组合条件

* [ condition1 ] && [  condition2 ]
* [ condition1 ] || [  condition2 ]

```sh
[ -f tmp_file ] && [ -d $HOME ] && echo true || echo false
# true
```

### Working with Advanced if-then Features

if-then 的增强模式

* Double parentheses for mathematical expressions(双括号)
* Double square brackets for advanced string handling functions(双方括号)

> Using double parentheses

创括号是针对算数运算的

test command 只提供了简单的算术运算，双括号提供的算力更强，效果和其他语言类似，格式 `(( expression ))`. 除了 test 支持的运算，它还支持如下运算

| Comparison | Description         |
| :--------- | :------------------ |
| val++      | Post-incremnet      |
| val--      | Post-decrement      |
| ++val      | Pre-increment       |
| --val      | Pre-decrement       |
| !          | Logical negation    |
| ~          | Bitwise negation    |
| **         | Exponentiation      |
| <<         | Left bitwise shift  |
| >>         | Right bitwise shift |
| &          | Bitwise Boolean AND |
| \|         | Bitwise Boolean OR  |
| &&         | Logical AND         |
| \|\|       | Logical OR          |

测试范例如下

```sh
# ** 次方操作
(( $val1**2 > 90 )) && echo true || echo false
true
```

> Using double bracket

双方括号是针对字符运算的，格式为 `[[ expression ]]`. 除了 test 相同的计算外，他还额外提供了**正则**的支持

**Note:** bash 是支持双方括号的，但是其他 shell 就不一定了

```sh
[[ $USER == i* ]] && echo true || echo false
# true
```

PS: 双等(==)表示 string 符合 pattern，直接用等号也是可以的

### Considering the case Command

对应 Java 中的 switch-case 语法, 格式如下. 当内容和 pattern 匹配时，就会执行对应的语句

```sh
case variable in
pattern1 | pattern2) command1;;
pattern3) command2;;
*) default commands;;
esac
```

```sh
cat test26.sh 
# #!/usr/local/bin/bash
# # using the case command
# case $USER in 
# rich | barbara)
#     echo "Welcome $USER"
#     echo "Please enjoy your visit";;
# i306454)
#     echo "Special testing account";;
# jessica)
#     echo "Do not forget to log off when you're done";;
# *)
#     echo "Sorry, you are not allowed here";;
# esac
./test26.sh 
# Special testing account
```

### Issues

**Issue1:** 写脚本的时候，发现一个很奇怪的问题

```sh
# 未赋值的变脸 -n 会返回 true ?!
[ -n $ret23 ] && echo true || echo false
# true
# 经多方查证，需要加引号
[ -n "$ret23" ] && echo true || echo false
# false

# 这里就体现出增强型的好处了
[[ -n $ret23 ]] && echo true || echo false
# false
```

以后可以的话都用增强型把，容错率更高


## More Structured Commands

这章介绍了其他一些流程控制的关键词

### The for Command

```sh
for var in list
do
    commands
done
```

PS: for 和 do 也可以写一起(for var in list; do)，和 if-then 那样

> Reading values in a list

```sh
cat test1.sh 
# #!/usr/local/bin/bash
# # basic for command
# for test in a b c d e
# do
#     echo The character is $test
# done
./test1.sh 
# The character is a
# The character is b
# The character is c
# The character is d
# The character is e
```

当 for 结束后变量还会存在

```sh
cat test1b.sh 
# #!/usr/local/bin/bash
# # Testing the for variable after the looping
# for test in a b c d e
# do
#     echo The character is $test
# done

# echo The last character is $test

# test=Connecticut
# echo "Wait, now we're visiting $test"
./test1b.sh 
# The character is a
# The character is b
# The character is c
# The character is d
# The character is e
# The last character is e
# Wait, now we're visiting Connecticut
```

> Reading complex values in a list

当 list 中包含一些标点时，结果可能就不是预期的那样了

```sh
cat badtest1.sh 
# #!/usr/local/bin/bash
# # another example of how not to use the for command
# for test in I don't know if this'll work, append some thing more?
# do
#     echo The character is $test
# done
./badtest1.sh 
# The character is I
# The character is dont know if thisll
# The character is work,
# The character is append
# The character is some
# The character is thing
# The character is more?
```

解决方案：

1. 给引号加转义符(for test in I don\'t know if this\'ll work, append some thing more?)
2. 将 string 用双引号包裹(for test in I don"'"t know if this"'"ll work, append some thing more?)

`for` 默认使用空格做分割，如果想要连词，你需要将对一个的词用双引号包裹起来

```sh
cat badtest2.sh
# #!/usr/local/bin/bash
# # another example of how not to use the for command

# for test in Nevada New Hampshire New Mexico New York North Carolina
# do
#     echo "Now going to $test"
# done
./badtest2.sh 
# Now going to Nevada
# Now going to New
# Now going to Hampshire
# Now going to New
# Now going to Mexico
# Now going to New
# Now going to York
# Now going to North
# Now going to Carolina

# update to: for test in Nevada "New Hampshire" "New Mexico" "New York"
./badtest2.sh 
# Now going to Nevada
# Now going to New Hampshire
# Now going to New Mexico
# Now going to New York
```

> Reading a list from a variable

```sh
cat test4.sh
# #!/usr/local/bin/bash
# # using a variable to hold the list
# list="Alabama Alaska Arizona Arkansas Colorado"
# list=$list" Connecticut"
# for state in $list
# do
#     echo "Have you ever visited $state?"
# done
./test4.sh 
# Have you ever visited Alabama?
# Have you ever visited Alaska?
# Have you ever visited Arizona?
# Have you ever visited Arkansas?
# Have you ever visited Colorado?
# Have you ever visited Connecticut?
```

PS: `list=$list" Connecticut"` 是 shell 中 append 字符串的常见操作

> Reading values from a command

结合其他命令，计算出 list 的值

```sh
echo a b c > states
cat test5.sh 
# #!/usr/local/bin/bash
# # reading values from a file
# file=states
# for state in $(cat $file)
# do
#     echo "Visit beautiful $state"
# done
./test5.sh 
# Visit beautiful a
# Visit beautiful b
# Visit beautiful c
```

> Changing the field separator

有一个特殊的环境变量叫做 IFS(internal field separator). 他可以作为分割 field 的依据。默认的分割符有

* A space
* A tab
* A newline

如果你想要将换行作为分割符，你可以使用 `IFS=$'\n'`

**Caution:** 定制 IFS 之后一定要还原

测试环节：如何打印当前 IFS 的值？

```sh
echo -n "$IFS" | hexdump
# 0000000 20 09 0a
# 0000003
printf %q "$IFS"
# ' \t\n'
```

**Caution:** 变量和引号之间的关系：

* 单引号，所见即所得。写什么即是什么
* 双引号，中间的变量会做计算
* 没符号，用于连续的内容，如果内容中带空格，需要加双引号

脚本中 `IFS.OLD=$IFS` 的赋值语句经常会抛异常 `./test_csv.sh: line 3: IFS.OLD=: command not found` 但是写成 `IFS_OLD` 的话就可以运行，可能是点号的形式会变成其他一些什么调用也说不定。以后为了稳定，还是用下划线的形式把 


```sh
IFS_OLD=$IFS
IFS=$'\n'
<use the new IFS value in code>
IFS=$IFS_OLD
```

其他定制分割符 `IFS=:`, 或者多分割符 `IFS=$'\n':;"`

> Reading a directory using wildcards

```sh
cat test6.sh
# #!/usr/local/bin/bash
# # iterate through all the files in a directory
# for file in /Users/i306454/tmp/*
# do
#     if [ -d "$file" ]
#     then 
#         echo "$file is a directory"
#     elif [ -f "$file" ]
#     then
#         echo "$file is a file"
#     fi
# done
./test6.sh 
# /Users/i306454/tmp/backup is a directory
# /Users/i306454/tmp/bash_test is a directory
# /Users/i306454/tmp/csv is a directory
# /Users/i306454/tmp/dfile is a directory
# /Users/i306454/tmp/ifenduser.sh is a file
# /Users/i306454/tmp/plantuml is a directory
# /Users/i306454/tmp/sh is a directory
```

PS: 在这个例子中有一个很有意思的点，在 test 中，将变量 file 使用双引号包裹起来了。这是因为 Linux 中带空格的文件或文件夹是合法的，如果没有引号，解析就会出错

**Caution:** It's always a good idea to test each file or directory before trying to process it.

### The C-Style for command

C 语言中 for 循环如下

> The C language for command

```c
for (i=0; i<10; i++)
{
    printf("The next number is %d\n", i);
}
```

bash 中也提供了类似的功能, 语法为 `for (( variable assignment; condition; iteration process ))` 例子：`for(( a=1; a<10; a++ ))`

限制：

* The assignment of the variable value can contain space
* The variable in the condition isn't preceded with a dollar sign
* The equation for the iteration process doesn't use the expr command format

这种用法，对我倒是很亲切，但是和之前用过的那些变量赋值之类的语句确实有一些语法差异的。这个语句中各种缩进，空格都不作限制

```sh
cat test8.sh 
# #!/usr/local/bin/bash
# # Testing the C-style for loop
# for ((i=1; i<= 3; i++))
# do
#     echo "The next number is $i"
# done
./test8.sh 
# The next number is 1
# The next number is 2
# The next number is 3
```

> Using multiple variables

for 中包含多个参数

```sh
cat test9.sh 
# #!/usr/local/bin/bash
# # Testing the C-style for loop
# for ((a=1, b=10; a<= 3; a++, b--))
# do
#     echo $a - $b
# done
./test9.sh 
# 1 - 10
# 2 - 9
# 3 - 8
```

### The while Command

```sh
while test command
do
    other commands
done
```

示例

```sh
cat test10.sh 
# #!/usr/local/bin/bash
# # while command test
# var1=3
# while [ $var1 -gt 0 ]
# do
#     echo $var1
#     var1=$[ $var1 - 1 ]
# done
./test10.sh 
# 3
# 2
# 1
```

> Using multiple test commands

while 判断的时候可以接多个条件，但是只有最后一个条件的 exit code 起决定作用。就算第一个条件我直接改为 `cmd` 每次都抛错，循环照常进行

还有，每个条件要新起一行, 当然用分号隔开也是可以的

```sh
cat test11.sh 
# #!/usr/local/bin/bash
# # Testing a multicommand while loop
# var1=3
# while echo $var1
#     [ $var1 -gt 0 ]
# do
#     echo "This is inside the loop"
#     var1=$[ $var1 - 1 ]
# done
./test11.sh 
# 3
# This is inside the loop
# 2
# This is inside the loop
# 1
# This is inside the loop
# 0
```

### The until Command

语意上和 while 相反，但是用法一致

```sh
until test commands
do
    other commands
done
```

```sh
cat test12 
# #!/usr/local/bin/bash
# # using the until command
# var1=100
# until [ $var1 -eq 0 ]
# do
#     echo $var1
#     var1=$[ $var1 - 25 ]
# done
./test12 
# 100
# 75
# 50
# 25
```

### Nesting Loops

循环嵌套，很常见

```sh
cat test14             
# #!/usr/local/bin/bash
# # nesting for loops
# for (( a=1; a<=3; a++))
# do
#     echo "Starting loop $a:"
#     for (( b=1; b<=3; b++ ))
#     do
#         echo "    Inside loop: $b"
#     done
# done
./test14
# Starting loop 1:
#     Inside loop: 1
#     Inside loop: 2
#     Inside loop: 3
# Starting loop 2:
#     Inside loop: 1
#     Inside loop: 2
#     Inside loop: 3
# Starting loop 3:
#     Inside loop: 1
#     Inside loop: 2
#     Inside loop: 3
```

while + for 的例子。树上的例子 for 中 边界条件是 `$var2<3;` 和之前表现的语法不一样，试了一下，有无 `$` 都是可以的

```sh
cat test14
# #!/usr/local/bin/bash
# # nesting for loops
# var1=3
# while [ $var1 -ge 0 ]
# do
#     echo "Outer loop: $var1"
#     for (( var2=1; var2<3; var2++))
#     do
#         var3=$[ $var1*$var2 ]
#         echo "    Inner loop: $var1 * $var2 = $var3"
#     done
#     var1=$[ $var1 - 1 ]
# done
./test14
# Outer loop: 3
#     Inner loop: 3 * 1 = 3
#     Inner loop: 3 * 2 = 6
# Outer loop: 2
#     Inner loop: 2 * 1 = 2
#     Inner loop: 2 * 2 = 4
# Outer loop: 1
#     Inner loop: 1 * 1 = 1
#     Inner loop: 1 * 2 = 2
# Outer loop: 0
#     Inner loop: 0 * 1 = 0
#     Inner loop: 0 * 2 = 0
```

### Looping on File Data

[Stackoverflow](https://stackoverflow.com/questions/4128235/what-is-the-exact-meaning-of-ifs-n) 上看到一篇解释 IFS=$'\n' 的帖子，挺好。一句话就是 `$'...'` 的语法可以表示转义符

通常来说，你会需要遍历文件中的内容，这需要两个知识点

1. Using nested loops
2. Changing the IFS environment variable

通过设置 IFS 你可以在包含空格的情况下处理一行内容. 下面是处理 `/etc/passwd` 文件是案例

```sh
cat test1
# #!/usr/local/bin/bash
# # changing the IFS value
# IFS.OLD=$IFS
# IFS=$'\n'
# for entry in $(cat /etc/passwd)
# do 
#     echo "Values in $entry -"
#     IFS=:
#     for value in $entry
#     do
#         echo "    $value"
#     done
# done
./test1
# Values in _oahd:*:441:441:OAH Daemon:/var/empty:/usr/bin/false
#     _oahd
#     badtest1.sh
#     badtest2.sh
#     test1
#     test12
#     test14
#     test15
#     441
#     441
#     OAH Daemon
#     /var/empty
#     /usr/bin/false
# ...
```

### Controlling the loop

通过 break, continue 控制流程

> The break command

打断单层循环, 这个语法适用于任何循环语句，比如 for, while, until 等

```sh
cat test17
# #!/usr/local/bin/bash
# # Breaking out of a for loop
# for var1 in 1 2 3 4 5
# do
#     if [ $var1 -eq 3 ]
#     then
#         break
#     fi
#     echo "Iteration number: $var1"
# done
# echo "The for loop is completed"
./test17
# Iteration number: 1
# Iteration number: 2
# The for loop is completed
```

打断内层循环

```sh
cat test19
# #!/usr/local/bin/bash
# # Breaking out of an inner loop
# for (( a=1; a<4; a++ ))
# do
#     echo "Outer loop: $a"
#     for (( b=1; b<4; b++ ))
#     do
#         if [ $b -eq 2 ]
#         then
#             break
#         fi
#         echo "    Inner loop: $b"
#     done
# done
./test19
# Outer loop: 1
#     Inner loop: 1
# Outer loop: 2
#     Inner loop: 1
# Outer loop: 3
#     Inner loop: 1
```

在内部循环执行过程中，打断外层循环，这个特性倒是很新颖，Java 中没见过 Haha

`break n` 默认是 1，打断当前的循环，设置成 2 就是打断外面一层直接退出。下面例子中，我们通过在 inner for 中 break 2 直接退出了外层 for 循环

```sh
cat test20 
# #!/usr/local/bin/bash
# # Breaking out of an outer loop
# for (( a=1; a<4; a++ ))
# do
#     echo "Outer loop: $a"
#     for (( b=1; b<4; b++ ))
#     do
#         if [ $b -gt 2 ]
#         then
#             break 2
#         fi
#         echo "    Inner loop: $b"
#     done
# done
./test20
# Outer loop: 1
#     Inner loop: 1
#     Inner loop: 2
```

> The continue command

提前结束循环，继续下一次循环. 下面例子中，当当前变量 3 < x < 8 时跳过打印. 前面介绍的循环体都适用，如 for, while 和 until

```sh
cat test21 
# #!/usr/local/bin/bash
# # Using the continue command
# for (( var1=1; var1<10; var1++ ))
# do 
#     if [ $var1 -gt 3 ] && [ $var1 -lt 8 ]
#     then
#         continue
#     fi
#     echo "Iteration number: $var1"
# done
./test21 
# Iteration number: 1
# Iteration number: 2
# Iteration number: 3
# Iteration number: 8
# Iteration number: 9
```

和 break 一样，continue 也支持 `continue n` 来跳过循环。测试用例中，当外层变量值 2 < x < 4 时，跳过打印

```sh
cat test22
# #!/usr/local/bin/bash
# # Continuing an outer loop
# for (( a=1; a<=5; a++ ))
# do 
#     echo "Iteration $a:"
#     for (( b=1; b<3; b++))
#     do
#         if [ $a -gt 2 ] && [ $a -lt 4 ]
#         then
#             continue 2
#         fi
#         var3=$[ $a * $b ]
#         echo "    The result of $a * $b is $var3"
#     done
# done
./test22
# Iteration 1:
#     The result of 1 * 1 is 1
#     The result of 1 * 2 is 2
# Iteration 2:
#     The result of 2 * 1 is 2
#     The result of 2 * 2 is 4
# Iteration 3:
# Iteration 4:
#     The result of 4 * 1 is 4
#     The result of 4 * 2 is 8
# Iteration 5:
#     The result of 5 * 1 is 5
#     The result of 5 * 2 is 10
```

### Processing the Output of a Loop

`for` 中打印的语句可以在 `done` 后面接文件操作符一起导入，还有这种功能。。。那我之前写脚本用的 printf 不是显得有点呆

```sh
cat test23
# #!/usr/local/bin/bash
# # redirecting the for output to a file
# for (( a=1; a<=5; a++ ))
# do 
#     echo "The number is $a"
# done > test23.txt
cat test23.txt 
# The number is 1
# The number is 2
# The number is 3
# The number is 4
# The number is 5
```

PS: 试了一下，`echo -n` 也是 OK 的

同理，`done` 后面还可以接其他的命令, 这个扩展很赞

```sh
cat test24
# #!/usr/local/bin/bash
# # piping a loop to another command
# for state in "North Dakota" Connecticut Illinois Alabama Tennessee
# do
#     echo "$state is the next place to go"
# done | sort
# echo "This completes our travels"
./test24
# Alabama is the next place to go
# Connecticut is the next place to go
# Illinois is the next place to go
# North Dakota is the next place to go
# Tennessee is the next place to go
# This completes our travels
```

### Practical Examples

一些实用的脚本范例

> Finding executable files

通过遍历 PATH 中的路径，统计处你可以运行的 commands 列表

```sh
cat test25
# #!/usr/local/bin/bash
# # Finding file in the PATH
# IFS.OLD=$IFS

# IFS=:
# for folder in $PATH
# do
#     echo "$folder:"
#     for file in $folder/*
#     do
#         if [ -x $file ]
#         then
#             echo "    $file"
#         fi
#     done
# done

# IFS=$IFS.OLD
./test25 | more
# /Users/i306454/.jenv/bin:
# /usr/local/sbin:
#     /usr/local/sbin/unbound
#     ....
```

PS: 这个 `more` 就用的很灵性！！

> Creating multiple user accounts

将需要创建的新用户写到文件中，并通过脚本解析文件，批量创建

```sh
#!/bin/bash
# Process new user accounts

input="users.csv"
while IFS=',' read -r userid name
do
    echo "adding $userid"
    useradd -c "$name" -m $userid
done < "$input"
```

## Chapter 14: Handling User Input

这章主要讲如何在脚本中做交互

### Passing Parameters

> Reading parameters

bash 会将传入的所有变量都赋给 positional parameters. 这些位置变量以 `$` 开头，`$0` 为脚本名称，`$1` 为第一个参数，以此类推

根据传入参数计算斐波那契额终值

```sh
cat test1
# #!/usr/local/bin/bash
# # Using one command line parameter
# factorial=1
# for (( number=1; number<=$1; number++ ))
# do
#     factorial=$[ $factorial * $number ]
# done
# echo The factorial of $1 is $factorial
./test1 5
# The factorial of 5 is 120
```

多参数调用案例

```sh
cat test2
# #!/usr/local/bin/bash
# # Testing two command line parameters
# total=$[ $1 * $2 ]
# echo The first parameter is $1
# echo The first parameter is $2
# echo The total value is $total

./test2 2 5
# The first parameter is 2
# The first parameter is 5
# The total value is 10
```

字符串作为参数

```sh
cat test3
# #!/usr/local/bin/bash
# # Testing string parameters
# echo Hello $1, glad to meet you

bash-5.1$ ./test3 jack
# Hello jack, glad to meet you
```

如果字符串之间有空格，需要用引号包裹起来

当参数数量超过 **9** 个的时候，你需要用花括号来调用

```sh
at ./test4
# #!/usr/local/bin/bash
# # handling lots of parameters
# total=$[ ${10} * ${11} ]
# echo The tenth parameter is ${10}
# echo The eleventh parameter is ${11}
# echo The total is $total
./test4 1 2 3 4 5 6 7 8 9 10 11 12
# The tenth parameter is 10
# The eleventh parameter is 11
# The total is 110
```

> Reading the script name

`$0 代表了脚本文件的名字

```sh
cat test5
# #!/usr/local/bin/bash
# # Testing the $0 parameter
# echo the zero parameter is set to: $0

bash test5
# the zero parameter is set to: test5
./test5
# the zero parameter is set to: ./test5
bash /Users/i306454/tmp/bash_test/test5
# the zero parameter is set to: /Users/i306454/tmp/bash_test/test5
```

不一样的调用方式，得到的第 0 参数值会不一样，如果像统一得到文件名，可以使用 basename 命令

```sh
cat test5b
# #!/usr/local/bin/bash
# # Using basename with the $0 parameter
# name=$(basename $0)
# echo the zero parameter is set to: $name

bash-5.1$ ./test5b
# the zero parameter is set to: test5b
sh test5b
# the zero parameter is set to: test5b
```

> Testing parameters

当脚本中需要用到参数，但是参数没有给，则脚本会抛异常，但是我们可以更优雅的处理这种情况

```sh
cat test7 
# #!/usr/local/bin/bash
# # Testing parameters before use
# if [ -n "$1" ]
# then
#     echo Hello $1, glad to meet you.
# else
#     echo "Sorry, you did not identity yourself."
# fi

./test7 jack
# Hello jack, glad to meet you.
./test7
# Sorry, you did not identity yourself.
```

### Using Special Parameter Variables

> Counting parameters

`$#` 用于统计参数数量

```sh
cat test8
# #!/usr/local/bin/bash
# # Getting the number of parameters
# echo There were $# parameters supplied.
./test8
# There were 0 parameters supplied.
./test8 123
# There were 1 parameters supplied.
./test8 1 2 3
# There were 3 parameters supplied.
./test8 "jack zheng"
# There were 1 parameters supplied.
```

根据上面的特性我们可以试着发散一下思路，尝试拿到最后一个参数

```sh
cat badtest1
# #!/usr/local/bin/bash
# # Testing grabbing last parameter
# echo The last parameter was ${$#}
./badtest1 1 2 3
# The last parameter was 13965
```

尝试失败，语法上来说，花括号中间是不允许有 `$` 符号的，你可以用叹号表达上面的意思

```sh
# #!/usr/local/bin/bash
# # Testing grabbing last parameter
# echo The last parameter was ${!#}

./badtest1 1 2 3
# The last parameter was 3
```

> Grabbing all the data

你可以使用 `$*` 或者 `$@` 拿到所有的参数，区别如下

* `$*` 会将所有的参数当作一个变量对待
* `$@` 会将所有的参数当作类似数组那种概念，分开对待。也就是说，你可以在 for 中循环处理

```sh
cat test11
# #!/usr/local/bin/bash
# # Testing $* and $@
# echo
# echo "Using the \$* method: $*"
# echo "Using the \$@ method: $@"

./test11 a b c d

# Using the $* method: a b c d
# Using the $@ method: a b c d
```

看上去没区别。。。这里需要结合 for 来观察

```sh
cat test12
# #!/usr/local/bin/bash
# # Testing $* and $@
# echo
# count=1
# #
# for param in "$*"
# do
#     echo "\$* Parameter #$count = $param"
#     count=$[ $count + 1 ]
# done
# #
# echo
# count=1
# #
# for param in "$@"
# do
#     echo "\$@ Parameter #$count = $param"
#     count=$[ $count + 1 ]
# done
./test12 a b c d

# $* Parameter #1 = a b c d

# $@ Parameter #1 = a
# $@ Parameter #2 = b
# $@ Parameter #3 = c
# $@ Parameter #4 = d
```

### Being Shify

我们可以通过 `shift` 关键字将参数左移，默认左移一位.

PS: note that the value for variable `$0`, the program name, remains unchanged

PPS: Be careful when working with the shift command. When a parameter is shifted out, its value is lost and can’t be recovered.

```sh
cat test13
#!/usr/local/bin/bash
# Demostrating the shift command
echo
count=1
while [ -n "$1" ]
do
    echo "Parameter #$count = $1"
    count=$[ $count + 1 ]
    shift
done

./test13 a b c d

# Parameter #1 = a
# Parameter #2 = b
# Parameter #3 = c
# Parameter #4 = d
```

移动多个位置测试

```sh
cat test14
#!/usr/local/bin/bash
# Demostrating a multi-position shift
echo
echo "The original parameters: $*"
shift 2
echo "The changed parameters: $*"
echo "Here is the new first parameter: $1"

./test14 1 2 3 4

# The original parameters: 1 2 3 4
# The changed parameters: 3 4
# Here is the new first parameter: 3
```

### Working with Options

介绍三种添加 Options 的方法，Options 顾名思义，就是命令中的可选参数。

> Finding your options

**单个依次处理法：** 可以使用 case + shift 的语法识别 options。将预先设置的 Options 添加在 case 的过滤列表中，然后遍历 `$1` 识别它

```sh
cat test15
#!/usr/local/bin/bash
# Extracting command line options as parameters
#
echo 
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option";;
        -b) echo "Found the -b option";;
        -c) echo "Found the -c option";;
         *) echo "$1 is not an option";;
    esac
    shift
done

./test15 -a -b -c -d asd

# Found the -a option
# Found the -b option
# Found the -c option
# -d is not an option
# asd is not an option
```

**Options parameters 分开处理法：** 我们可以认为的在两种参数中间添加一个分割符，比如 `--` 作为 options 的结束和 parameter 的开始. 在脚本中现实的识别并处理它。

PS: 突然意识到 `$0` 是不算在 `$*` 和 `$@` 中的

下面的例子中，如果没有 `--`，则所有参数都在第一个 do-while 中处理了。加了之后会在两个 loop 中处理

```sh
cat test16
#!/usr/local/bin/bash
# Extracting options and parameters
#
echo
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option";;
        -b) echo "Found the -b option";;
        -c) echo "Found the -c option";;
        --) shift
            break;;
         *) echo "$1 is not an option";;
    esac
    shift
done
#
count=1
for param in $@
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done

./test16 -c -a -b test1 test2 test3

# Found the -c option
# Found the -a option
# Found the -b option
# test1 is not an option
# test2 is not an option
# test3 is not an option

./test16 -c -a -b -- test1 test2 test3
# Found the -c option
# Found the -a option
# Found the -b option
# Parameter #1: test1
# Parameter #2: test2
# Parameter #3: test3
```

**带值的 options 处理：** 有些命令中 options 是带值的，比如 `./testing.sh -a test1 -b -c -d test2`。这是我们就需要在脚本中识别可选参对应的值

下面的例子中 `-b test1` 是一个带值的可选参数，我们在 识别到 `-b` 后立即拿到 `$2` 即为对应的值

PS: 但是怎么看，bash 中添加可选参数都很麻烦啊，如果是可选参数带多个值呢，那不是还得加逻辑。。。

```sh
cat test17
#!/usr/local/bin/bash
# Extracting command line options and values
#
echo
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option";;
        -b) param="$2"
            echo "Found the -b option, with parameter value $param"
            shift ;;
        -c) echo "Found the -c option";;
        --) shift
            break;;
         *) echo "$1 is not an option";;
    esac
    shift
done
#
count=1
for param in $@
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done

./test17 -a -b test1 -d 

# Found the -a option
# Found the -b option, with parameter value test1
# -d is not an option
```

> Using the getopt command

介绍 `getopt` 工具函数，方便处理传入的参数

**Looking at the command format** `getopt` 可以接受一系列的 options 和 parameters 并以正确的格式返回, 语法如下 `getopt optstring parameters`

**Tips** getopt 还有一个增强版 getopts, 后面章节会介绍

测试 getopt, b 后面添加了冒号表示它是带值的可选参数. 如果输入的命令带有未定义的参数，则会给出错误信息。如果想要忽略错误信息，则需要 getopt 带 -q 参数

```sh
getopt ab:cd -a -b test1 -cd test2 test3
#  -a -b test1 -c -d -- test2 test3

getopt ab:cd -a -b test1 -cde test2 test3
# getopt: illegal option -- e
#  -a -b test1 -c -d -- test2 test3

getopt -q  ab:cd -a -b test1 -cde test2 test3
#  -a -b 'test1' -c -d -- 'test2' 'test3'
```

PS: MacOS 的 bash 是不支持 -q 参数的！使用 docker 绕过了这个限制 诶嘿 ╮(￣▽￣"")╭

**Using getopt in your scripts** 这里有一个小技巧，我们需要将 getopt 和 set 配和使用 `set -- $(getopt -q ab:cd "$@")`

```sh
#!/usr/local/bin/bash
# Extracting command line options and values with getopt
#
set -- $(getopt ab:cd "$@")
# 为了兼容 Mac version bash, 将参数去掉了
# set -- $(getopt -q ab:cd "$@")
#
echo
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option";;
        -b) param="$2"
            echo "Found the -b option, with parameter value $param"
            shift ;;
        -c) echo "Found the -c option";;
        --) shift
            break;;
         *) echo "$1 is not an option";;
    esac
    shift
done
#
count=1
for param in $@
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done

./test18 -ac

# Found the -a option
# Found the -c option

./test18 -a -b test1 -cd test2 test3 test4

# Found the -a option
# Found the -b option, with parameter value test1
# Found the -c option
# -d is not an option
# Parameter #1: test2
# Parameter #2: test3
# Parameter #3: test4

./test18 -a -b test1 -cd "test2 test3" test4

# Found the -a option
# Found the -b option, with parameter value test1
# Found the -c option
# -d is not an option
# Parameter #1: test2
# Parameter #2: test3
# Parameter #3: test4
```

PS: 最后一个例子中可以看到 getopt 并不能很好的处理字符串， "test2 test3" 被分开解析了。幸运的是，我们有办法解决这个问题

> Advancing to getopts

`getopts` 是 `getopt` 的增强版本，格式如下 `getopts optstring variable`, optstring 以冒号开始

如下所示，`getopts` 

* 自动为我们将每个参数封装到 opt 变量中
* 删选的时候省区了 `-`
* 提供了内置的 `$OPTARG` 代表可选参数的值
* 将为定义的参数类型用问好替换

```sh
cat test19
#!/usr/local/bin/bash
# Simple demostration of the getopts command
#
echo
while getopts :ab:c opt
do
    case "$opt" in
        a) echo "Found the -a option" ;;
        b) echo "Found the -b option, with value $OPTARG" ;;
        c) echo "Found the -c option" ;;
        *) echo "Unknown option: $opt" ;;
    esac
done

./test19 -ab test1 -c 

# Found the -a option
# Found the -b option, with value test1
# Found the -c option

./test19 -b "test1 test2" -a

# Found the -b option, with value test1 test2
# Found the -a option

./test19 -d

# Unknown option: ?
```

`getopts` 还内置了一个 OPTIND 变量，可以在处理每个参数的时候自动 +1. OPTIND 变量初始值为 1，如果要取 params 部分，则 shift $[ $OPTIND - 1 ]

下面例子中, 

```sh
cat test20
#!/usr/local/bin/bash
# Processing options $ paramters with getopts
#
echo
echo start index: "$OPTIND"
while getopts :ab:cd opt
do
    case "$opt" in
        a) echo "Found the -a option" ;;
        b) echo "Found the -b option, with value $OPTARG" ;;
        c) echo "Found the -c option" ;;
        d) echo "Found the -d option" ;;
        *) echo "Unknown option: $opt" ;;
    esac
    echo changing index: "$OPTIND"
done
#
echo
echo opt index: "$OPTIND"
shift $[ $OPTIND -1 ]
#
echo
count=1
for param in "$@"
do
    echo "Parameter $count: $param"
    count=$[ $count + 1 ]
done

./test20 -a -b test1 -d test2 test3 test4

# Found the -a option
# Found the -b option, with value test1
# Found the -d option

# opt index: 5

# Parameter 1: test2
# Parameter 2: test3
# Parameter 3: test4
```

### Standardizing Options

介绍 shell 中参数表示的 comman sense

| Option | Description                                     |
| :----: | :---------------------------------------------- |
|   -a   | Shows all objects                               |
|   -c   | Produces a count                                |
|   -d   | Specifies a directory                           |
|   -e   | Expands an object                               |
|   -f   | Specifies a file to read data from              |
|   -h   | Displays a help message for the command         |
|   -i   | Ignores text case                               |
|   -l   | Produces a long format version of the output    |
|   -n   | Uses a non-interactive(batch) mode              |
|   -o   | Specifies an output file to redirect all output |
|   -q   | Run in quiet mode                               |
|   -r   | Processes directories and files recursively     |
|   -s   | Runs in silent mode                             |
|   -v   | Produces verbose output                         |
|   -x   | Excludes an object                              |
|   -y   | Answers yes to all questions                    |

### Getting User Input

bash 提供了 read 方法来作为用户输入

> Reading basics

`read` 可以从键盘或文件中获取输入

```sh
cat test21
#!/usr/local/bin/bash
# Testing the read command
#
echo -n "Enter your name: "
read name
echo "Hello $name, welcome to my program."

./test21 
# Enter your name: jack
# Hello jack, welcome to my program.

./test21
# Enter your name: jack zheng
# Hello jack zheng, welcome to my program.
```

上面的例子中，cmd 会将所有的输入看作一个变量处理

带用户提示的输入

```sh
cat test22
#!/usr/local/bin/bash
# Testing the read -p option
#
read -p "Please enter your age: " age
days=$[ $age * 365 ]
echo "That makes you over $days days old! "

./test22
# Please enter your age: 5
# That makes you over 1825 days old!
```

和第一个实验对照，cmd 也可以将所有输入当作 list 处理

```sh
cat test23
#!/usr/local/bin/bash
# Testing the read command
#
read -p "Enter your name: " first last
echo "Checking data for $last, $first"

./test23
# Enter your name: jack zheng
# Checking data for zheng, jack
```

如果你没有为 read 指定变量，bash 会自动将这个值赋给环境变量 $REPLY

```sh
cat test24
#!/usr/local/bin/bash
# Testing the REPLY Environment variable
#
read -p "Enter your name: "
echo
echo "Hello $REPLY, welcome to my program."

./test24 
# Enter your name: jack zheng

# Hello jack zheng, welcome to my program.
```

> Timing out

默认情况下 read 会一直阻塞在那里，等待用户输入，但是我们也可以设置一个等待时间

```sh
cat test25
#!/usr/local/bin/bash
# Timing the data entry
#
if read -t 5 -p "Please enter your name: " name
then
    echo "Hello $name, welcome to my script"
else
    echo
    echo "Sorry, too slow! "
fi

./test25
# Please enter your name: 
# Sorry, too slow! 
./test25
# Please enter your name: jack
# Hello jack, welcome to my script
```

read 还可以指定输入的长度, 设定好长度后，你输入对应长度的内容，他立马就执行下去了

```sh
cat test26
#!/usr/local/bin/bash
# Getting just one character of input
#
read -n1 -p "Do you want to continue [Y/N]? " answer
case $answer in
Y | y)  echo
        echo "fine, continue on..." ;;
N | n) echo
        echo OK, goodbye
        exit ;;
esac
echo "This is the end of the script"

./test26
# Do you want to continue [Y/N]? y
# fine, continue on...
# This is the end of the script

./test26
# Do you want to continue [Y/N]? n
# OK, goodbye
```

> Reading with no display

在输入一些敏感信息时，你不希望他显示在屏幕上，可以用 -s 参数

```sh
cat test27
#!/usr/local/bin/bash
# hiding input data from the monitor
#
read -s -p "Enter your password: " pass
echo
echo "Is you password really $pass? "

./test27
# Enter your password: 
# Is you password really jack?
```

> Reading from a file

Linux 系统中，可以通过 read 命令从文件中按行读取。当读完后，返回非 0

```sh
cat test28
#!/usr/local/bin/bash
# Reading data from a file
#
count=1
cat test | while read line
do
    echo "Line $count: $line"
    count=$[ $count + 1 ]
done
echo "Finished process the file"

cat test
# line 1
# line 2
./test28
Line 1: line 1
Line 2: line 2
Finished process the file
```

PS: 如果文件的末行没有换行，则最后一行并不会被处理