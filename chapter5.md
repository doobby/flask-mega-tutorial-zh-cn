# 用户登录

在[第三章](chapter3.md)中我们实现了登录页面的表单和处理方法，在[第四章](chapter4.md)学习如何操作数据库。在本章中，我将来告诉你们如何把这两章的内容结合起来，实现一个简单的用户登录系统。

## 密码 Hash 

在[第四章](chapter4.md)的用户模型中，我们定义了 `password_hash` 字段，但还没有使用。这个字段用来保存用户密码的哈希值，用户登录时将用户使用相同的哈希函数进行检查。密码 Hash 是一个涉及到安全性的复杂话题，好在我们可以使用一些现成的库来进行相关操作。

我们选择使用[Werkzeug](http://werkzeug.pocoo.org/)库来进行密码哈希，这个包是 Flask 的依赖包之一，因此在安装 Flask 时已经被安装好了。下面我们演示了如何在 Python shell 对密码进行哈希操作

```python
>>> from werkzeug.security import generate_password_hash
>>> hash = generate_password_hash('foobar')
>>> hash
'pbkdf2:sha256:50000$vT9fkZM8$04dfa35c6476acf7e788a1b5b3c35e217c78dc04539d295f011f01f18cd2175f'
```

在本例中，密码 `foobar` 被编码成了一长串加密过的字符串，这一过程是不可逆的（即无法从 hash 值中反推出原始密码）。此外，如果我们再次计算密码 hash 值，会发现与之前的并不相同。所以不可能通过比较哈希结果是否相同来判定原始密码是否相同。

检验密码使用使用另一个函数，如下所示

```python
>>> from werkzeug.security import check_password_hash
>>> check_password_hash(hash, 'foobar')
True
>>> check_password_hash(hash, 'barfoo')
False
```

检验函数传入两个参数：生成的哈希值和要检验的密码（登录表单提交）。若哈希值与密码相符，返回 `True`；否则返回 `False`

整个密码哈希的代码逻辑可以实现为 `User` 数据库模型的两个方法，如下是加入了密码哈希功能的 `app/models.py` 

```python
from werkzeug.security import generate_password_hash, check_password_hash

# ...

class User(db.Model):
    # ...

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
```

有了这两个方法，user 对象可以安全地进行密码校验，不需要存储原始密码。如下所示

```python
>>> u = User(username='susan', email='susan@example.com')
>>> u.set_password('mypassword')
>>> u.check_password('anotherpassword')
False
>>> u.check_password('mypassword')
True
```

## Flask-Login 简介

在本章中我们将使用到一个非常流行的 Flask 扩展 [Flask-Login](https://flask-login.readthedocs.io/)。该扩展用于管理用户登录状态，以便记录用户登录信息并在不同的页面之间进行跳转。此外，还提供了 “记住登录状态” 的功能，即使关闭浏览器，服务也能保证用户的登录状态。要继续后面的章节，我们先要在虚拟环境中安装 `Flask-Login` 扩展

```bash
(venv) $ pip install flask-login
```

和其它扩展一样，Flask-Login 也需要在应用 `app` 实例化后进行初始化，如下 `app/__init__.py` 局部修改所示

```python
# ...
from flask_login import LoginManager

app = Flask(__name__)
# ...
login = LoginManager(app)

# ...
```

## TODO Preparing The User Model for Flask-Login (0/0.8)
## TODO User Loader Function (0/0.7)
## TODO Logging Users In (0/1.9)
## TODO Logging Users Out (0/1.2)
## TODO Requiring Users to Login (0/2.6)
## TODO Showing The Logged In User in Templates (0/1.4)
## TODO User Registration (0/4.9)
