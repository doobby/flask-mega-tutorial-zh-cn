# Web 表单

在[第二章](chapter.md)中，我们创建了模板来生成应用的主页，其中的用户对象和博客对象是我们伪造的数据。在本章中我们来处理应用中的另一个重要问题：如何来接受用户提交的表单数据。

Web 表单 (Web Form) 是 Web 应用的重要组成部分。我将会演示如何通过 Web 表单来支持用户提交新的博客以及登录到应用。

在开始这一章这前，确保你已经完成了我们在前面章节完成的应用代码并且可以正常运行

## Flask-WTF 简介

要处理 Web 表单数据，我们准备使用 [Flask-WTF](http://packages.python.org/Flask-WTF) 扩展，该扩展是对 [WTForms](https://wtforms.readthedocs.io/) 包的一个轻量级封装和对 flask 的集成。这是我要演示的第一个扩展，但不是最后一个。扩展 (Extensions) 是 Flask 生态系统中的非常重要的部分，它们补足了 Flask 没有涉及的一些解决方案。

Flask 扩展也是普通的 Python 包，可以通过 `pip` 来安装，如下演示了在虚拟环境中安装 Flask-WTF 的命令

```bash
(venv) $ pip install flask-wtf
```

## 配置

到目前我们的应用还是相对简单的，我们暂时还没有关注配置 (Configuration) 的问题。一旦涉及到更复杂的问题，就需要通过一系列的配置来使 Flask 及其扩展实现更灵活的功能。

为应用程序提供配置选项有多种方式。最基本的办法是将键值对存储在 `app.config` 字典对象中。例如

```python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'you-will-never-guess'
# ... add more variables here as needed
```

虽然这种语法可以提供配置，但我还是建议遵守“关注点分离”原则 (the principle of separation of concerns)，不要把所有的配置放在一起。我可以采用更复杂一些的方法，把我们的配置放在不同的文件中。

我建议使用类来存储配置，这种方式比较容易扩展，可以把配置按类组织成 Python 模块。下面是我改进后的配置类，保存在项目根目录下的 `config.py` 文件，包含有 `Secret Key` 配置

```python
import os

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
```

很简单不是？配置被定义成 `Config` 类成员变量。其它变量也可以依样添加到这个类中，如果后面我需要不止一组配置集时，我可以继承实现 `Config` 的子类。当然现在我们不用这么做。

`SECRET_KEY` 变量是 Flask 应用的一个重要配置。Flask 和它的一些扩展使用这个 SECRET KEY 用于加密、生成签名或令牌（tokens）。Flask-WTF 扩展使用 SECRET KEY 来保护 Web Form 来免受 [跨站请求伪造(Cross-Site Request Forgery, CSRF)](http://en.wikipedia.org/wiki/Cross-site_request_forgery) 的攻击。因为这个变量是私密的，生成的令牌或是签名的安全性都依赖于它，要确保不会有不可信的人或者应用获得这一密码。

我们设定 SECRET_KEY 时使用了 `or` 语句，表示首先我们尝试使用环境变量 `SECRET_KEY`，如果没有环境变量，则使用硬编码的字符串。这种优先使用环境变量，提供硬编码配置作为备用的方式在开发中很常见。在开发过程中，我们对安全性的要求并不那么高，这样我们可以使用默认的硬编码值。当真实部署到生产环境时，我们可以为每个环境设置不同的密码以确保安全性。

现在我们有了配置文件，我们要告诉 Flask 加载并使用它。可以通过 `app.config.from_object()` 方法来实现，如下 `app/__init__.py` 所示

```python
from flask import Flask
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

from app import routes
```

从 `config` 中导入 `Config` 需要留意一个，正如我们从 `flask` 包中导入 `Flask` 类。小写的 `config` 表示 Python 模块 _config.py_ ，而大写的 C 表示 Config 类。

`app.config` 的配置项可以像字典元素一样被访问。这里我们在 Python 解释器进行验证

```python
>>> from microblog import app
>>> app.config['SECRET_KEY']
'you-will-never-guess'
```

## 用户登录表单

Flask-WTF 扩展使用 Python 类来表示 Web 表单。一个 Form 类基本就是把定义了类成员变量的普通类。

记着我们的关注点分离原则，我现在要定义一个新的 `app/forms.py` 模块来存储我们的 Web 表单类。用户登录时需要提供用户名和密码，此外表单还会有一个“记住登录状态”的选项以及一个“提交”按键

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired

class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')
```

大多数的 Flask 扩展都以 `flask_<name>` 来命名，这里 Flask-WTF 的名称就是 `flask_wtf` ，我们从中导入了基类 `FlaskForm`

用来表示成员类型的四个类直接由 WTForms 包导入，Flask-WTF 扩展并不支持自定义类型。对于 `LoginForm` 类的每个成员变量被限制为某种类型，并配置了标签名（或者描述信息）。

有的变量中传入了第二个可选参数 `validators`，其中 `DataRequired` 用来验证该表单项不为空。验证函数有很多，后面我们还会在其它表单项中用到。

## 表单模板 (Form Templates)

接下来我们需要把表单嵌入到 HTML 模板中来生成页面。一个好消息是我们定义在 `LoginForm` 中的成员变量可以被自动生成 HTML 内容，这样我们要作的事情就很简单了。下面你将看到我们的登录模板代码，我们把它保存在 `app/templates/login.html` 文件

```html
{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```

我们继承了（`extends`）[第二章](chapter2.md)里定义的 `base.html` 基本模板，来保证页面顶部有一个通用样式的导航栏。

这个模板需要使用 `LoginForm` 生成 `form` 对象来提供参数，用户的输入将被发送到相应的 view function 中，目前我们还没有实现。

HTML 的 `<form>` 元素中包含了我们的 Web 表单。其中 `action` 属性指示了浏览器在进行提交时应该使用的 URL，当设置为空时默认使用当前 URL 地址（当前页面）。`method` 方法表示使用何种 HTTP 请求方法，默认方式为 `GET`，不过大多数时间我们选择 `POST` 方法，因为它把用户提交的数据包含在请求内容中，而非像 GET 一样添加在 URL 中。`novalidate` 告诉浏览器不要对表单的内容进行验证，而是交给 Flask 服务来验证。使用 `novalidate` 是可选的，我们先这样设置，这样我们可以测试服务端的验证功能。

`form.hidden_tag()` 生成了一个隐藏的元素，其中包含了用于防止 CSRF 攻击的令牌信息。我们要做是确保引入这一隐藏元素，并且在 Flask 配置中定义 `SECRET_KEY` 变量，其它的都由 Flask-WTF 扩展来完成。

如果你曾经写过 HTML Web 表单，你会奇怪在模板中我们没有为每个表单项加入 HTML 标签。这是因为我们的 form 对象知道如何生成 HTML 标签。我们要做的只是引入 `{{ form.<field_name>.label }}` 来插入标签名，引入 ``{{ form.<field_name>() }}` 来生成表单项。要为 HTML 标签添加属性，我们只需要把它们当成参数传入即可。例如上面的用户名和密码我们传入了 `size` 参数，用被添加到 HTML 的 `<input>` 标签属性中。同样，你的 CSS Class 或者 ID 设置也可以这样做。

## TODO Form Views (0/1.9)
## TODO Receiving Form Data (0/3.3)
## TODO Improving Field Validation (0/2.3)
## TODO Generating Links (0/1.9)
