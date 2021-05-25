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

> Nesting ifs

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
# if test
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

创括号是正对算数运算的

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

双方括号是针对字符运算的，格式为 `[[ expression ]]`. 糊了 test 相同的计算外，他还额外提供了**正则**的支持

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