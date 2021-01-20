---
title: VSCode setup python 环境
date: 2020-06-12 15:43:34
categories:
- vscode
tags:
- python
---

VSCode setup python 独立运行环境

## 安装依赖

1. 安装 pipenv `pip install pipenv --user`
2. 创建独立环境 `pipenv shell`, 还可以通过 `pipenv --three/two` 指定 python 版本
3. 修改 pipfile, 使用国内源加速
4. 安装依赖 `pipenv install pdfminer.six`

```pipfile
[[source]]
name = "pypi"
url = "https://pypi.douban.com/simple"
verify_ssl = true
```

查看 VSCode 左下角的运行环境是不是你新建的那个，不是的话 `pipenv --venv` 查看新建 venv 路径， `Ctrl + Shift + p` 搜索 `python: select interpreter` 选择你新建的那个 env

## reload module after update

如果某些方法正在进行中，可能频繁修改，在 ipython 中调试的时候可以用 reload 来重新加载，也可以指定 ipython 到自动重加载模式 [autoreload mode](https://ipython.org/ipython-doc/stable/config/extensions/autoreload.html)

```python
import importlib
importlib.reload(PDFParser)
```

## Debug Click command

测试代码，接收 count, name 参数并在终端输出

```python
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()
```

配置 launch.json 运行文件

```json
{
    "name": "click",
    "type": "python",
    "request": "launch",
    "program": "${file}",
    "console": "integratedTerminal",
    "args": [
        "--count", "3", "--name", "jaaack"
        ]
}
```

点击菜单栏的 debug 按钮，选择配置的 'click' run config，点击这个配置**左边**的运行按钮，直接运行即可。需要注意的点：

1. 别点右上角那个，那个是直接运行当前文件的，不会接收配置的参数！！
2. 当断点生效时，VSCode 还提供了一个 DEBUG CONSOLE 来给你操作运行时的变量，真是太酷了
3. 如果你想要输入多行，使用 `Shift + Enter` 实现换行

如果要调试带有 argumnet 注解的代码，比如

```python
@click.command()
@click.argument('input', type=click.File('rb'))
@click.argument('output', type=click.File('wb'))
def inout(input, output):
    """Copy contents of INPUT to OUTPUT."""
    while True:
        chunk = input.read(1024)
        if not chunk:
            break
        output.write(chunk)

if __name__ == '__main__':
    inout("input_path", "output_path")
```

只需要将 argument 直接写在最后的函数入口中就行了