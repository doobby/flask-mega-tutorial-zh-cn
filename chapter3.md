# Web 表单

在[第二章](chapter.md)中，我们创建了模板来生成应用的主页，其中的用户对象和博客对象是我们伪造的数据。在本章中我们来处理应用中的另一个重要问题：如何来接受用户提交的表单数据。

Web 表单 (Web Form) 是 Web 应用的重要组成部分。我将会演示如何通过 Web 表单来支持用户提交新的博客以及登录到应用。

在开始这一章这前，确保你已经完成了我们在前面章节完成的应用代码并且可以正常运行

## TODO Introduction to Flask-WTF (0/0.4)

要处理 Web 表单数据，我们准备使用 [Flask-WTF](http://packages.python.org/Flask-WTF) 扩展，该扩展是对 [WTForms](https://wtforms.readthedocs.io/) 包的一个轻量级封装和对 flask 的集成。这是我要演示的第一个扩展，但不是最后一个。扩展 (Extensions) 是 Flask 生态系统中的非常重要的部分，它们补足了 Flask 没有涉及的一些解决方案。

Flask 扩展也是普通的 Python 包，可以通过 `pip` 来安装，如下演示了在虚拟环境中安装 Flask-WTF 的命令

```bash
(venv) $ pip install flask-wtf
```

## TODO Configuration (0/2.7)
## TODO User Login Form (0/1.1)
## TODO Form Templates (0/2.1)
## TODO Form Views (0/1.9)
## TODO Receiving Form Data (0/3.3)
## TODO Improving Field Validation (0/2.3)
## TODO Generating Links (0/1.9)
