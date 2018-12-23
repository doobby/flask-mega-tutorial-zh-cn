# 模板 (Template)

根据第一章的内容，我们已经实现了一个可用的简单 web 应用，它的目录结构是这样的

```
microblog\
  venv\
  app\
    __init__.py
    routes.py
  microblog.py
```

要运行这个程序，需要先设置 `FLASK_APP=microblog.py` 环境变量，然后执行 `flask run` 启动 web 服务，可以在浏览器上访问 http://localhost:5000 地址。

在本章中，我们将继续改进这一应用，学习如何优雅的生成具体复杂结构和动态内容的 web 页面。如果对于 web 服务的开发流程还有疑问的同学，可以先回顾一下 [第一章](chapter1.md) 的内容。

## 什么是模板 (Template)?

我想让我们的微博客应用 (microblogging application) 有一个标题栏用来欢迎用户。当然现在我们还没有用户这个概念，我们先假定有这样一个用户，用 Python 字典类型来表示

```python
user = {'username': "Miguel'}
```

这种模拟对象 (mock object) 可以让我们将注意力集中在应用的某一部分，而不用去完整的实现其细节，是开发中的一种常用技巧。比如我们现在关注于实现用户主页，则不需要把精力花在实现用户系统上，先定义一个用户对象来继续我们的研究。

相应的 view function 返回一个简单字符串，表示一个完整的 html 页面，如下 `app/routes.py` 所示

```python
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    return '''
<html>
    <head>
        <title>Home Page - Microblog</title>
    </head>
    <body>
        <h1>Hello, ''' + user['username'] + '''!</h1>
    </body>
</html>'''
```

如果你对 HTML 完全不了解，我建议你先读读 wikipedia 上关于 [HTML 标记语言](https://en.wikipedia.org/wiki/HTML#Markup) 的简要介绍文章。

更新 view function 后，我们的 web 页面将会变成这样

![mock-user](./images/ch02-mock-user.png)

我估计你们也看出来了，这样生成 HTML 页面不是个好方法，里面硬编码了太多的内容，如果我想要后续加入用户的博客内容，那将会有太多要改动的地方。而且应用有会有太多的 view function 来对应于这些不同的 URL，若后来有一天我想要修改应用的布局就不得不去修改每个 view function 里的 HTML 内容。也就是说这种方式非常不利于我们程序的扩展性。

如果你可以将程序的逻辑与具体的布局显示分享，事情就会简单明了得多。你可以雇用专门的网页设计师来设计一个非常专业的页面，而你的 Python 代码逻辑并不需要修改。

模板 (Template) 可以帮助你来分享展示 (Presentation) 与业务逻辑 (Business Logic）。在 Flask 中，模板被写在单独的文件中，通常保存于一个单独的模板目录中。现在我们在 microblog 项目目录中创建一个 templates 目录来保存模板

```bash
(venv) $ mkdir app/templates
```

然后我们来生成第一个模板，和之前的 view function `index()` 中返回的 HTML 几乎完全相同。主页模板 `app/templates/index.html` 内容如下

```html
<html>
    <head>
        <title>{{ title }} - Microblog</title>
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```

这是一个标准的、最简单的 HTML 页面。里面唯一需要关注的是有几处我们使用占位符 (placeholder，用 `{{ ... }}` 包裹)。其内容将在运行时被展开成为上下文的变量内容。

这样我们就把页面的展示功能抽取出来，放在 HTML 模板中了。因此我们的 view function 实现就可以简化成渲染模板。修改后的 `app/routes.py` 文件如下

```python
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    return render_template('index.html', title='Home', user=user)
```

现在代码清晰多了不是？试试这个新版本，它和之前的效果是一样的。在浏览器中加载了页面后你可以通过查看源码来比较 HTML 内容，看看它的原始的模板有什么不同。

通过上下文和模板生成 HTML 页面的操作我们称之为渲染 (render) 。渲染模板的函数在 Flask 框架中，名为 `render_template()` 。这一函数接收模板文件路径和一组模板变量，返回替换了占位符的渲染结果。

`render_template()` 实际上调用了 Flask 集成的 [jinja2 模板引擎](http://jinja.pocoo.org/)。由 Jinja2 来实现用模板变量对 `{{ ...}}` 的替换工作。

## TODO Conditional Statements (0/0.7)

## TODO Loops (0/2.5)

## TODO Template Inheritance (0/2.3)