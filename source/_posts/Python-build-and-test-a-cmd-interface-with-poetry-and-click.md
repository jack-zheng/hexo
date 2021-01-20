---
title: 使用 poetry 和 click 自定义终端命令
date: 2021-01-20 11:16:09
categories:
- python
tags:
- poetry
- vscode
- click
---

最近打算新加一个命令到项目，突然发现项目启动不了了，查了一下是 `setup.py` 和 toml 文件的兼容性有问题，找到了解决方案，顺便结合找到的资料，将使用 poetry 和 click 自定义命令重新记录一下。

## 版本信息

Python: 3.7.3
pip: 19.3.1
OS:       posix

## 创建工程

新建项目 `poetry new --name greet --src clickgreet`，如过想名字保持一致，那直接 `poetry new greet` 即可。命令执行完后会在当前目录下生成项目，结构如下

```txt
clickgreet
├── README.rst
├── pyproject.toml
├── src
│   └── greet
│       └── __init__.py
└── tests
    ├── __init__.py
    └── test_greet.py
```

安装依赖包 `poetry add click`, 首次运行 add 命令时，poetry 会帮你创建一个虚拟环境，并将包安装进去。安装完后你可以在项目的 toml 文件中看到 `tool.poetry.dependencies` 下有了 click 的依赖

在 src/greet 文件夹下新建 `greet.py` 添加逻辑代码, 代码实现如下功能：接受两个参数 name, count 后在终端输出对应次数的名字

```python
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def greet(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)


if __name__ == '__main__':
    greet()
```

在 toml 中添加程序入口

```toml
[tool.poetry.scripts]
greet = "greet.greet:greet"
```

终端输入 `poetry install` 将代码安装到虚拟环境，之后输入 `poetry run greet` 试运行脚本，可以看到终端给出提示

```bash
mypc ~/tmp/clickgreet > poetry run greet
Your name: jack
Hello jack!
```

至此，主题部分结束，下面开始编写测试部分，在目录的 `test_greet.py` 中添加测试代码如下

```python
from click.testing import CliRunner

from greet.greet import greet

from greet import __version__


def test_version():
    assert __version__ == '0.1.0'

def test_greet_cli():
    runner = CliRunner()
    result = runner.invoke(greet, ['--name', 'jack'])
    assert result.exit_code == 0
    assert "Hello jack!" in result.output
```

终端输人 `poetry run pytest` 运行测试用例, 至此教程主体结束。

PS: 可以在 toml 中添加配置使用 douban 镜像加速下载

```toml
[[tool.poetry.source]]
name = "douban"
url = "https://pypi.doubanio.com/simple/"
```

## Debug Click Command

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
    inout(["input_path", "output_path"])
```

只需要将 argument 直接写在最后的函数入口中就行了，这里有一个设定不是很理解，在最后一行，按理说我设置的参数列表应该是 `"arg1", "arg2"` 才对，但是执行的时候会出问题，设置成 list type 的就没问题。。。

## 集成 setup.py

以上的命令行运行时有一个限制，它必须在对应的文件夹下才能工作，pip 是支持将脚本安装到本地的。如何操作？步骤如下：

poetry 是没有 setup.py 文件的，运行 `poetry add dephell` 安装 dephell 来自动生成 setup.py 文件

toml 文件中添加生成 `setup.py` 的配置

```toml
[tool.dephell.main]
from = {format = "poetry", path = "pyproject.toml"}
to = {format = "setuppy", path = "setup.py"}
```

同时你还要修改 `[build-system]` 配置，在 requires 中添加 setuptools 的依赖 `requires = ["setuptools", "poetry>=0.12"]`，这是个 pip 的 bug 但是到 2020-1 为止还没有修复

运行 `dephell deps convert` 生成 setup.py 然后运行 `pip install -E .` 安装到本地。`cd` 到其他目录直接在终端输入 `greet` 测试通过，脚本正常工作，不需要什么 hack 的代码，棒棒哒 ╮(￣▽￣"")╭

## 资料白嫖

* [setup.py 安装到本地报错 no module name 'setuptools'](https://github.com/python-poetry/poetry/discussions/1135)
* [poetry + click + UT guide](https://dev.to/bowmanjd/build-a-command-line-interface-with-python-poetry-and-click-1f5k)