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


## TODO Database Models (0/1.3)
## TODO Creating The Migration Repository (0/0.6)
## TODO The First Database Migration (0/0.9)
## TODO Database Upgrade and Downgrade Workflow (0/0.6)
## TODO Database Relationships (0/1.9)
## TODO Play Time (0/2.2)
## TODO Shell Context (0/1.4)
