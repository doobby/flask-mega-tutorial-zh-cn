# 个人信息页 (Profile Page) 和 Avator 图标

在本章我们将专注于实现用户个人信息面(Profile)。个人信息面只包含了用户自己输入的一些信息。我将演示如何动态的为每个用户生成信息页，并添加一个编辑器让用户可以自行修改其信息内容。

## 个人信息页

要实现一个用户信息页，先来实现一个新的 view function 映射到 _/user/<username>_ URL，如下 `app/routes.py` 所示

```python
@app.route('/user/<username>')
@login_required
def user(username):
    user = User.query.filter_by(username=username).first_or_404()
    posts = [
        {'author': user, 'body': 'Test post #1'},
        {'author': user, 'body': 'Test post #2'}
    ]
    return render_template('user.html', user=user, posts=posts)
```

`@app.route` 装饰器与之前的不太一样，我们在里面使用了动态元素 `<username>` （动态元素被包裹在 `<` 和 `>`）中。Flask 允许在动态元素位置填写任何字符串，调用 view function 时，将传入实际元素作为参数。比如，客户端请求 `/user/susan` 地址，view function 中的参数 `username` 被设置为 `'susan'`。该 view function 只允许登录用户访问，因此我们还加上了 `@login_required` 装饰器（来自 Flask-Login）。

函数本身实现相当简单，我们首先从数据中查询用户信息。之前我们用的是 `all()` 函数来获取所有用户，`first()` 只获取第一个（如果没有结果则返回 `None`）。在这里我们使用了 `first()` 的变种 `first_or_404()`，它的做用相当于 `first()` 返回用户，如果没有结构则自动发送[404 错误](https://en.wikipedia.org/wiki/HTTP_404)到客户端。也就是说，我们把用户信息存储在 `user` 中，如果用户不存在，则抛出 404 异常。

如果数据库查询正常，则表示用户被查询到。接下来，我们来伪造一组 posts，将 user 和 posts 渲染到 `user.html` 模板中。

如下所示是我们的 `user.html` 模板（定义了用户信息页）

```html
{% extends "base.html" %}

{% block content %}
    <h1>User: {{ user.username }}</h1>
    <hr>
    {% for post in posts %}
    <p>
    {{ post.author.username }} says: <b>{{ post.body }}</b>
    </p>
    {% endfor %}
{% endblock %}
```

信息页现在就完成了，不过还缺少一个链接让用户跳转到这个页面。要让用户比较容易来检查他们的信息页，我将在导航栏里加一个链接。如下是修改后的 `app/templates/base.html`

```html
    <div>
      Microblog:
      <a href="{{ url_for('index') }}">Home</a>
      {% if current_user.is_anonymous %}
      <a href="{{ url_for('login') }}">Login</a>
      {% else %}
      <a href="{{ url_for('user', username=current_user.username) }}">Profile</a>
      <a href="{{ url_for('logout') }}">Logout</a>
      {% endif %}
    </div>
```

用 `url_for()` 来生成到具体用户信息页的链接地址。因为用户信息页的 view function 中包含了动态参数，`url_for()` 函数接收 `username` 做为参数，我们传入了 Flask-login 的 `current_user` 参数来生成正确的 URL。

![user-profile](ch06-user-profile.png)

试着运行一下我们的程序。点击导航栏中的 `Profile` 链接，跳转到用户信息页。现在没有到其它用户信息页的链接，如果想要访问其他用户的信息页，我们可以直接在浏览器地址栏中手动输入 URL 地址。比如要访问 "john" 用户的信息页（用户已存在），可以输入地址 http://localhost:5000/user/john 来查看。

## TODO Avatars (0/2.6)
## TODO Using Jinja2 Sub-Templates (0/0.8)
## TODO More Interesting Profiles (0/1.2)
## TODO Recording The Last Visit Time For a User (0/1.4)
## TODO Profile Editor (0/2.6)
