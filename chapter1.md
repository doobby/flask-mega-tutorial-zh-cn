# Hello World

## TODO 前言 (1)

## 安装 Python

要做的第一件事情是安装 Python 环境。如果你的操作系统没有提供安装包，可
以到[官网](http://python.org/download/)获取。如果你用的是 Windows 的
WSL 或者 Cygwin 环境，需要注意你要安装的 Python 不是 Windows 原生版本，
而应该选择 Ubuntu (WSL) 或者 Cygwin 的版本。

要验证 Python 是否正常工作，可以打开一个 terminal 窗口，输入 `python3`
或者 `python`。正常的话应该会看到

```bash
$ python3
Python 3.5.2 (default, Nov 17 2016, 17:05:23)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> _
```

此时 Python 解释器在等待输入。在后面的章节里，你将学习到交互式命令行的
各种功能。不过现在只是确认 Python 被安装好了。输入 `exit()` 回车退出
Python 命令行。或者在 Linux 或者 Mac OS X 上使用快捷键 `Ctrl-D`，
Windows 上使用 `Ctrl-Z` 加回车退出。

## 安装 Flask

接下来安装 `Flask`，但在此之前我说明一下 Python 包安装的最佳实践。

Python 的包（比如 Flask）保存在一个公共仓库，任何人都可以下载并且安装它们。Python 官方的包仓库被称为 [PyPI](https://pypi.python.org/pypi) (Python Package Index，Python 包索引，也有人把这个仓库称作奶酪商店 (cheese shop))。

要从 PyPI 上获取并安装软件包非常容易，Python 提供了一个 `pip` 工具来完成这件事（在 Python 27 中 `pip` 尚未被打包在 Python 中需要单独的安装）。

下面的命令演示使用 pip 安装一个包

```bash
$ pip install <package-name>
```

不过大多数情况上面的命令不会成功。因为当 Python 解释器被全局安装（可以被计算机上的所有用户访问），作为普通用户你是没有权限来对 Python 进行一些修改操作的，只有管理员用户才能正常安装。`pip` 工具从 PyPI 上下载包，把它添加到 Python 的安装目录中，这样所有的 Python 脚本都可以使用这个包提供的脚本。

假设你已经使用 Flask 0.11 版本来实现了自己的 web 应用。而 Flask 有了 0.12 版本，你要开发的第二个应用决定使用 0.12 版，但是升级 Flask 版本可能会影响之前的 web 应用。所以我们最好设法让第一个应用使用 Flask 0.11，而新的应用使用 Flask 0.12 版。

要解决这一问题——为不同的应用配置不同版本的包，Python 使用了 Virutal Environment （虚拟环境）的概念。Virtual Environment 是一份完整的 Python 解释器副本。当你在 Virtual Environment 中安装包时，系统级的 Python 并不受影响，只有当前副本会被修改。你可以自由为每个应用配置不同的 Python 虚拟环境。Virtual Environment 由创建者所有，因此不需要他们具有管理员权限。

现在让我们创建一个目录 `microblog` 来保存我们的工程

```bash
$ mkdir microblog
$ cd microblog
```

如果你使用的是 Python3，virtual environment 支持已经包含在其中，你可以直接创建 Virtual Environment

```bash
python3 -m venv venv
```

命令指示 Python 运行 venv 包，告诉它来创建一个名为 `venv` 的 Virtual Environment。第一个 `venv` 是 Python Virutal Environment 包的名称，第二个是要创建的 Virtual Environment 包的名称。如果你对此仍有困惑，那么你可以他第二个 `venv` 改成任何名称来表示 Virtual Environment。一般习惯会在工程目录下创建一个 `venv` 目录来作为  Virtual Environment 环境，这样切到工程目录时就能找到对应的 Virtual Environment 环境。

注意，有些操作系统中，你需要用 `python` 而不是 `python3`。不过也有一些系统的 `python` 指的是 Python 2.X 版本，`python3`表示 3.X 版本。

当命令执行完成后，将会生成一个 `venv` 目录，里面保存有 Virtual Environment 需要的文件。

如果你使用的 Python 版本低于 3.4（包括 2.7），Virtual Environment 并没有被打包在 Python 中。对这些低版本，你需要去单独下载和安装 [virtualenv](https://virtualenv.pypa.io/) 工具。安装后你可以直接用 `virtualenv` 创建 virtual environment

```bash
$ virtualenv venv
```

无论用何种方式创建好 venv，接下来你需要告知系统你要使用（激活）Virtual Environment

```bash
$ source venv/bin/activate
(venv) $ _
```

如果你使用的是 Window CMD 窗口，则需要用以下命令
```cmd
$ venv\Scripts\activate
(venv) $ _
```

激活 virtual environment 后，你的当前 terminal 会话讲使用虚拟环境中的 Python 解释器。并且命令提示符也发生了变化提醒你在虚拟环境中。Virtual Environment 仅临时并且私有的在当前会话内生效，关闭窗口自动退出。甚至在你同时打开了多个 terminal 时，可以为每个 terminal 配置使用不同的虚拟环境。

现在在虚拟环境中，我们来安装 Flask

```bash
(venv) $ pip install flask
```

如果你想确认 Flask 被安装成功，可以启动一个 Python 解释器尝试导入 Flask 包

```bash
>>> import flask
>>> _
```

上面的指令没有报错，表示 Flask 被正常安装，庆祝一下吧。

## TODO A "Hello, World" Flask Application (3.5)

