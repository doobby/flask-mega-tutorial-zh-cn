# 数据库 （Database）

本章非常重要。对于大多数应用都需要数据的持久化和高效访问，这正是引入数据库的意义。

## 数据库与 Flask 

你大概已经知道 Flask 本身并没有提供数据库支持。事实上 Flask 在很多方面都做了刻意的裁剪，这样也很好，我们可以自由的使用最适合于我们应用场景的数据库，而不必被迫写一堆代码来适配。

Python 访问数据库有很多种选择，其中有很多都提供了 Flask 的扩展以更方便地与应用集成。数据库本身被分为两大类：关系型的（relational model）和非关系型的。后者也常常被称为 _NoSQL_，表示这类数据库并不支持关系数据库查询语言（SQL）。两类数据库都有很多种产品，在我看来关系型数据库更加适合于我们的应用，它可以有效的组织用户、博客等数据结构。而 NoSQL 数据库更加适应于结构性不那么强的数据。当然我们的应用也可以用 NoSQL 来实现，不过我还是准备使用关系型的。

在[第三章](chapter3.md)中，我为你展示了第一个 Flask 扩展。在本章中我还将使用两个扩展，首先是[Flask-SQLAlchemy](http://packages.python.org/Flask-SQLAlchemy)，这是对 [SQLAlchemy](http://packages.python.org/Flask-SQLAlchemy) 的封装，使得更适合 Flask 的规范和使用。SQLAlchemy 提供了 ORM (Object Relational Mapper）实现，使得应用可以以更高层次的类(class)，对象(object)，方法(methods) 来操作数据库，避免直接操纵表(table) 和 SQL 语句。ORM 的作用是将高阶操作映射成为数据库的底层操作。

SQLAlchemy 同时支持很多种关系型数据库的 ORM 映射，常见的如 [MySQL](https://www.mysql.com/)， [PostgreSQL](https://www.postgresql.org/) 以及 [SQLite](https://www.postgresql.org/)。这种特性很有用，我们可以在开发时使用本地的 SQLite 数据库，而在部署时换用更健壮的 MySQL 或者 PostgreSQL 数据库，不需要对代码作出修改。

要安装 Flask-SQLAlchemy 包，确保已经激活 (activate) 虚拟环境，执行 pip 命令

```bash
(venv) $ pip install flask-sqlalchemy
```

## 数据库迁移（Migrations）

大部分的数据库教程都会详细介绍如何创建和使用库，但对于当应用更新而需要对数据库进行更新的问题解释很不详细。当然，关系型数据库的更新本身就很困难，因为围绕着结构化数据，任何结构的改变都有很大的迁移(Migrated) 工作量。

我要使用的第二个扩展是 [Flask-Migrate](https://github.com/miguelgrinberg/flask-migrate)，这正是我们想要的。这一扩展封装了 [Alembic](https://github.com/miguelgrinberg/flask-migrate) 库，该库是针对 SQLAlchemy 的数据库迁移框架。要加入数据库迁移支持，我们需要在开始时做一些额外的工作 ，但这是值得的，这让你后来的修改变得更加容易和稳定。

同样使用 pip 来安装扩展

```bash
(venv) $ pip install flask-migrate
```

## 配置 Flask-SQLAlchemy

在开发过程中，我们选择使用 SQLite 数据库。SQLite 数据库是开发小型甚至更大规模应用最方便好用的数据库。它的数据被存储于硬盘上的一个文件中，不需要像 MySQL 或者 PostgreSQL 那样启动服务。

我们为配置 `config.py` 中添加相关配置，如下

```python
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    # ...
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

Flask-SQLAlchemy 扩展从 `SQLALCHEMY_DATABASE_URI` 配置变量中获取数据库的连接信息。回忆一下我们在[第三章](chapter3.md)所讲，使用环境变量来配置 Flask，并为之提供一个默认值以防环境变量没有设置。在这里，我通过 `DATABASE_URL` 环境变量来设置数据库地址，如果未指定，则使用应用主目录( `basedir` )下的 `app.db` 文件做为默认位置 。

`SQLALCHEMY_TRACK_MODIFICATIONS` 配置项设为 `False` 禁用了 Flask-SQLAlchemy 中我们暂时用不到的功能：在每次修改数据库时触发某个操作。

应用中我们把数据库表示成 Python 对象，同样数据库迁移也是一个 Python 对象。这些对象需要在初始化时被创建，如下 `app/__init__.py` 所示

```python
from flask import Flask
from config import Config
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)
migrate = Migrate(app, db)

from app import routes, models
```

我们加入了三部分内容：
1. `db` 对象表示数据库实例
2. `migrate` 表示数据库迁移实例
3. `models` 模块中将定义数据库的表结构

大部分的 Flask 扩展都和上面 `db`、`migrate` 有一样初始化语法

## 数据库模型

数据库的数据将被表示成为类(class)，这种表示方法称为数据库模型(database models)。SQLAlchemy 的 ORM 层将负责把类数据和操作转换为相应的数据库的表和行。

让我们先来定义用户的数据库模型。使用[WWW SQL Designer](http://ondras.zarovi.cz/sql/demo)工具，我生成了下面的用户表示意图：

![users.png](images/ch04-users.png)

其中 `id` 几乎在表示模型中都会出现，通常被用作主键(primary key)。库中的每个用户都被分配以唯一的 ID。大多数情况主键会由数据库自动分配，所以我们这里只需要将之标记为主键即可。

`username`, `email`, `password_hash` 字段为字符串类型（数据库中称为 `VARCHAR` 类型），我们设定了它们的最大长度以优化数据库存储性能。`username` 和 `email` 不用多解释，`password_hash` 有点技巧。为了安全计，我们不应该直接把密码保存在数据库里，否则很可能被攻击而导致密码泄露。所以我们存放的是密码的哈希值，这样安全得多。后面我将详细解释，现在我们暂时跳过其中细节。

设计好了用户表，我们现在可以实现代码来表示它，如下我们在 `app/models.py` 中定义了用户表

```python
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)    
```

`User` 类继承了 `db.Model` 基类，该基类是 Flask-SQLAlchemy 所有模型都应该继承的。在类中，我们定义了四个类成员变量，其内容是 `db.Column` 的实例，指定了列的类型，加上一些可选的配置项，表示哪些列是不允许重复的(unique)，哪些列需要加索引（indexed）以提高访问效率。

里面的 `__repr__` 方法告诉 Python 如何来显示这类对象，清晰的表示有利于调试。我们可以在交互式命令行中试验一下

```python
>>> from app.models import User
>>> u = User(username='susan', email='susan@example.com')
>>> u
<User susan>
```

## 创建迁移仓库 (Migration Repository)

上一节中创建了数据库表结构(schema)的初始版本。不过随着应用的演进，我们很可能需要修改这一结构，比如添加一列，或者修改、删除一些内容。Alembic (Flask-Migrate 扩展迁移框架）可以帮助我们完成这些修改，而不需要完全重新创建整个库。

要完成这个复杂的工作，Alembic 需要维护一个迁移仓库 (Migration Repository)，这是一个存储有迁移脚本的目录。每次我们对数据库结构进行修改时，就有一个用于完成迁移功能的脚本在这个目录下生成。我们需要顺序执行这些脚本来完成迁移工作。

Flask-Migrate 扩展了 `flask` 子命令（记得之前的 `flask run` 命令吗？）。`flask db` 子命令用于调用 Flask-Migrate 进行数据库迁移。初次创建库时，我们需要同时用 `flask db init` 创建这个迁移库

```bash
(venv) $ flask db init
  Creating directory /home/miguel/microblog/migrations ... done
  Creating directory /home/miguel/microblog/migrations/versions ... done
  Generating /home/miguel/microblog/migrations/alembic.ini ... done
  Generating /home/miguel/microblog/migrations/env.py ... done
  Generating /home/miguel/microblog/migrations/README ... done
  Generating /home/miguel/microblog/migrations/script.py.mako ... done
  Please edit configuration/connection/logging settings in
  '/home/miguel/microblog/migrations/alembic.ini' before proceeding.
```

记住 `flask` 命令依赖于 `FLASK_APP` 环境变量，所以在此之前我们需要设定 `FLASK_APP=microblog.py` （参见[第一章](chapter1.md)）

正确执行后将生成一个 `migrations` 目录，其中包含有一些文件和有版本号标识的子目录。这些文件从现在起也是我们工程的一部分，需要将它们加入到我们的版本管理系统中。

## 第一次数据库迁移

当迁移仓库就位后，我们可以演示下如何对 `User` 数据库模型进行迁移。迁移有两种方式：手动或者自动。要完成自动迁移，Alembic 通过比较 Python 的数据库模型与实际数据库的表结构，生成迁移脚本来对数据库进行一些修改以符合模型定义。此时，我们还没有生成数据库，因此自动迁移将生成整个用户表。下面演示了 `flask db migrate` 子命令如何完成自动迁移操作的

```bash
(venv) $ flask db migrate -m "users table"
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_username' on '['username']'
  Generating /home/miguel/microblog/migrations/versions/e517276bb1c2_users_table.py ... done
```

命令打印出了本次迁移的过程。前两行可以直接忽略，下来它找到一个 `user` 表和两个索引，然后生成了迁移脚本，`e517276bb1c2` 是本次迁移对应的标识号（每个人执行结果不同）。我们传给 `-m` 的内容是可选的注释，用来描述本次迁移。

生成的迁移脚本现在是项目的一部分，我们把它加入版本控制。如果好奇你可以研究研究生成的迁移代码。其中有两个函数分别是 `upgrade()` 和 `downgrade()`，前者用于更新，更者用于回退。这样 Alembic 可以在不同的历史版本间切换。

`flask db migrate` 命令并没有对数据库进行修改，只是生成了迁移脚本。要做用于数据库，需要调用 `flask db upgrade` 命令

```bash
(venv) $ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> e517276bb1c2, users table
```

由于我们使用 SQLite 做为数据库，`upgrade` 命令检测到数据库不存在（命令执行后将生成 `app.db` 文件）而创建它。对于 MySQL 或者 PostgreSQL 数据库，你将会看到有新的 database 被创建。

注意 Flask-SQLAlchemy 使用了 snake case （蛇形命令法）来命名表名。因此 `User` 类对应的表名为 `user`。如果数据库模型名为 `AddressAndPone`，则生成的表名应该为 `address_and_phone`。你可以自由选择生成的表名，可以在模型类中添加 `__tablename__` 属性，将之设置为你想要的表名。

## 数据库的升级与回退流程

尽管我们的应用现在还只是个雏形，现在讨论数据库迁移升级策略并不显早。假设你的应用在开发环境上运行，同时在线上还部署了一份生产环境在使用。

如果在下一版本中你对你的数据库模型做了一些修改，比如添加了一个新的表结构。若是没有迁移策略来帮助我们修改数据库表结构，那么你就得手动对本地开发环境和线上生产环境进行维护，那将会是很大的工作量。

但是如果有了数据库迁移支持，在你修改了数据库模型后，你只需要生成一个新的迁移脚本(`flask db migrate`)，可以研究下代码看看这个自动生成的脚本确实做了我们想要做的事情，然后对我们开发环境使用这个脚本（`flask db upgrade`）就可以了。此外我们会把这个迁移脚本加入到版本控制系统中和源码一起保存。

当你准备好为线上生产环境发布一个盯死时，你需要做的是确定你的应用的迁移更新版本，然后运行 `flask db upgrade`。Alembic 会自动监测生产环境数据库状态，运行所有的迁移升级脚本来确保它升级到是新版本的数据库表结构。

我之前已经提到过，使用 `flask db downgrade` 命令可以撤消升级迁移。若有回退的需要，这一功能对生产环境至关重要。很可能你设计了新的表结构，升级后发现是有问题的，这样你可以回退它，然后删除升级迁移脚本，重新设计你的表结构模型并升级之。

## 数据库关系 (Relationships)

关系型数据库非常适合表示数据之间的关系。比如我们的用户信息(`users`)和博客(`posts`)之间就有关联关系。最高效的方式是设计一个关联关系表，把两者连接在一起。

一旦在用户和博客之间建立了关联关系，数据库就能自动处理这条连接。最基本的功能是给你一篇博客，你就能得知它的作者是谁。较复杂一点的应用场景是给你一个用户，我们可以查找同所有他写的博客。Flask-SQLAlchemy 同时支持这两种查询方式。

这我们扩展数据库，来存储博客信息以及与用户的关联关系。下面是我们新设计的 `blogs` 表

![users-posts](images/ch04-users-posts.png)

`posts` 表中包含了主键 `id`，`body` 存储博客内容，`timestamp` 时间戳。此外还有一个额外的 `user_id` 字段，与作者相关联。因为所有的用户都有一个唯一的主键 `id`，我们让 `user_id` 指向这个用户 `id`，这种指向主键的关联字段称为 _foreign key_ （外键）。数据库的设计结构中显示了外键链接到了另一个表的主键。这种关系称为一对多关系 (one-to-many)，其中一个用户可能有多个博客。

我们修改 `app/models.py` 添加了新的 `Post` 数据库模型

```python
from datetime import datetime
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))
    posts = db.relationship('Post', backref='author', lazy='dynamic')

    def __repr__(self):
        return '<User {}>'.format(self.username)

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.String(140))
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

    def __repr__(self):
        return '<Post {}>'.format(self.body)
```

新的 `Post` 类表示用户书写的博客。`timestamp` 需要被索引，因为我们需要按时间来排序博客，同时我们为它指定了一个默认值生成函数 `datetime.utcnow` （注意这里是函数，没有 `()`），在没有提供值时，由 SQLAlchemy 来调用这个函数生成一个默认值。通常来说，我们可以使用 UTC 时间来避免不同时区的处理出现问题。UTC 时间在展示时可以被转换为当地时间。

`user_id` 初始化成 `user.id` 的外键。在这种引用关系下，`user` 表示数据库的表名而不是模型的类名称（这在设计上有些不一致，因为有的 SQLAlchemy 函数如 `db.relationship()` 中使用的模型类名称，模型类名称一般以大写开头，而表名则是小写的蛇形命名）。

`User` 类中新加了一个 `post` 字段，被使用 `db.relationship` 来初始化。这并不是个真正的数据库字段，而是更高阶的逻辑视图，用来表示用户和博客表之间的关联。在一对多的关系中，`db.relationship` 需要被定义在 "一" 的那一边，这样通过 `User` 可以容易地访问与之关联和多个博客。例如，如果我的用户存储在 `u` 中，那么 `u.posts` 会自动运行一个 SQL 查询来返回与之关联的所有博客。`db.relationship` 的第一个参数是与之关联的那个目标模型类名称（可以是类本身，或是类名，用类名可以避免类未定义的错误）。`backref` 参数定义了字段名，返回的关联结果中将添加一个新的字段表示主表的原始记录，即 `post.author` 记录了当前博客的作者记录。`lazy` 参数定义了如何进行关联关系查询，这个我们在后面具体说明，现在我们暂时跳过它。

现在我更新的数据库模型，需要生成一个新的迁移脚本

```bash
(venv) $ flask db migrate -m "posts table"
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'post'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_post_timestamp' on '['timestamp']'
  Generating /home/miguel/microblog/migrations/versions/780739b227a7_posts_table.py ... done
```

并把脚本应用到需要升级的数据库上

```bash
(venv) $ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade e517276bb1c2 -> 780739b227a7, posts table
```

记着把升级迁移脚本加到你的版本控制中去。

## TODO Database Relationships (0/1.9)
## TODO Play Time (0/2.2)
## TODO Shell Context (0/1.4)
