---
title: LSCASSB partiii advanced shell scripting
date: 2021-06-05 16:38:52
categories:
- Shell
tags:
- Linux命令行与shell脚本编程大全 3rd
- TODO
---

## Creating Functions

### Basic Script Functions

#### Creating a function

方式一：

```sh
function name {
    commands
}
```

方式二：

```sh
name() {
    commands
}
```

#### Using functions

function 不一定要写在开头，但是你使用之前，他的定义必须已经声明了

```sh
cat test1
#!/usr/local/bin/bash
# using a function in a script

function func1 {
    echo "This is an example of a function"
}

count=1
while [ $count -le 4 ]
do
    func1
    count=$[ $count + 1 ]
done

echo "This is the end of the loop"
func1
echo "Now this is the end of the script"

./test1
# This is an example of a function
# This is an example of a function
# This is an example of a function
# This is an example of a function
# This is the end of the loop
# This is an example of a function
# Now this is the end of the script
```

### Returning a Value

bash shell 会在函数结束后给他一个返回值

#### The default exit status

默认情况下，exit status 是函数中最后一个命令的返回值，可以使用 `$?` 查看

下面实验中，我们在 func1 中 ls 一个不存在的文件，并打印 exit code

```sh
cat test4
#!/usr/local/bin/bash
# testing the exit status of a function

function func1 {
    echo "Trying to display a non-existent file"
    ls -l badfile
}

echo "testing the function"
func1
echo "The exit status is: $?"

./test4 
# testing the function
# Trying to display a non-existent file
# ls: badfile: No such file or directory
# The exit status is: 1

cat test4b
#!/usr/local/bin/bash
# testing the exit status of a function

function func1 {
    ls -l badfile
    echo "Trying to display a non-existent file"
}

echo "testing the function"
func1
echo "The exit status is: $?"


./test4b
# testing the function
# ls: badfile: No such file or directory
# Trying to display a non-existent file
# The exit status is: 0
```

#### Using the return command

你也可以用 return 关键子在函数末尾给他返回一个整数最 exit code

```sh
cat test5
#!/usr/local/bin/bash
# using the return command in a function

function db1 {
    read -p "Enter a value: " value
    echo "doubling the value"
    return $[ $value * 2 ]
}

db1
echo "The new value is $?"

./test5
# Enter a value: 10
# doubling the value
# The new value is 20
```

注意点：

* 记得在 function 执行完成后立刻取值备用
* exit code 返回只能在 0-255 之间

#### Using function output

你也可以将返回值包装在输出里面，通过 `result=$(db1)` 的形式拿到。和之前调用系统函数一个套路

```sh
cat test5b
#!/usr/local/bin/bash
# using the return command in a function

function db1 {
    read -p "Enter a value: " value
    echo $[ $value * 2 ]
}

result=$(db1)
echo "The new value is $result"

./test5b
# Enter a value: 20
# The new value is 40
```

### Using Variables in Functions

#### Passing parameters to a function

```sh
cat test6
#!/usr/local/bin/bash
# passing parameters to a function

function addem {
    if [ $# -eq 0 ] || [ $# -gt 2 ]
    then
        echo -1
    elif [ $# -eq 1 ]
    then
        echo $[ $1 + $1 ]
    else
        echo $[ $1 + $2 ]
    fi
}

echo -n "Adding 10 and 15: "
value=$(addem 10 15)
echo $value
echo -n "Let's try adding just one number: "
value=$(addem 10)
echo $value
echo -n "Now trying adding no numbers: "
value=$(addem)
echo $value
echo -n "Finally, try adding three numbers: "
value=$(addem 10 15 20)
echo $value

./test6
# Adding 10 and 15: 25
# Let's try adding just one number: 20
# Now trying adding no numbers: -1
# Finally, try adding three numbers: -1
```

但是脚本文件中的参数并不会默认的应用到你定义的 function 中去，你需要显示的指定才能使他生效

下面实验中，当我们运行脚本是没有给参数，则直接运行了 else 的路径，如果给了参数，这个参数也不会传递给 badfunc1

```sh
cat badtest1
#!/usr/local/bin/bash
# Trying to access script parameters inside a function

function badfunc1 {
    echo $[ $1 * $2 ]
}

if [ $# -eq 2 ]
then
    value=$(badfunc1)
    echo "The result is $value"
else
    echo "Usage: badtest1 a b"
fi

./badtest1
# Usage: badtest1 a b
./badtest1 10 15
# ./badtest1: line 5: *  : syntax error: operand expected (error token is "*  ")
# The result is
```

如果要修复上面的脚本，你可以直接在实现中指定 `value=$(badfunc1 $1 $2)`

```sh
cat test7
#!/usr/local/bin/bash
# Trying to access script parameters inside a function

function func7 {
    echo $[ $1 * $2 ]
}

if [ $# -eq 2 ]
then
    value=$(func7 $1 $2)
    echo "The result is $value"
else
    echo "Usage: badtest1 a b"
fi

./test7
# Usage: badtest1 a b
./test7 10 15
# The result is 150
```

#### Handling variables in a function

Functions 使用两种类型的变量

* Global
* Local

Global 变量是在 script 中任何地方都可以访问的变量。在主程序中定义的 global 变量，function 中可以访问到。function 中定义的 global 变量，主程序也能访问到。一般来说你在脚本中定义的变量都是 global 的

```sh
cat test8
#!/usr/local/bin/bash
# Using a global variable to pass a value

function db1 {
    value=$[ $value * 2 ]
}

read -p "Enter a value: " value
db1
echo "The new value is: $value"

./test8
# Enter a value: 10
# The new value is: 20
```

如果你在 script 和 function 中都用到了同名的变量，就可能导致赋值出问题，是你的程序难以排错

```sh
cat badtest2
#!/usr/local/bin/bash
# Demonstrating a db use of variables

function func1 {
    temp=$[ $value + 5 ]
    result=$[ $temp * 2 ]
}

temp=4
value=6

func1
echo "The result is $result"
if [ $temp -gt $value ]
then
    echo "temp is larger"
else
    echo "temp is smaller"
fi

./badtest2
# The result is 22
# temp is larger
```

我们可以使用 local variables 避免这种问题，格式如下 `local temp`. 他让你的变量只在 function 中生效

```sh
cat test9
#!/usr/local/bin/bash
# Demonstrating the local keyword

function func1 {
    local temp=$[ $value + 5 ]
    result=$[ $temp * 2 ]
}

temp=4
value=6

func1
echo "The result is $result"
if [ $temp -gt $value ]
then
    echo "temp is larger"
else
    echo "temp is smaller"
fi


./test9
# The result is 22
# temp is smaller
```

### Array Variables and Functions