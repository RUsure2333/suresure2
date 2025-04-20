# Flask 开发个人博客项目实战教程

**项目简介：** 我们将使用 Flask 构建一个前后端分离的个人博客网站，实现用户注册登录（基于 Token 认证）、文章的发布、编辑、删除，以及文章列表浏览等基本功能。后端提供 JSON API 接口，前端使用简单的 HTML+JavaScript 调用这些接口进行交互。通过本教程，具有一定 Python 基础但缺乏 Web 开发经验的读者也能从零开始完成一个完整可运行的个人博客项目。

## 一、环境准备与项目结构

在开始编码之前，先准备好开发环境并了解项目的文件结构：

- **Python 版本：** 建议使用 Python 3.7+。
- **必需库：** Flask、Flask-SQLAlchemy、Flask-CORS 等。可以使用 `pip` 安装：  
  ```bash
  pip install Flask Flask-SQLAlchemy Flask-CORS PyJWT
  ```  
  （其中 PyJWT 用于生成和验证 JWT Token）
- **开发工具：** 可以使用任意文本编辑器或 IDE 编写代码。建议在虚拟环境下开发，以避免依赖混杂。

**项目文件结构设计：** Flask 官方并不强制特定的项目结构，但为了便于维护，我们按照“按模块分层”的方式组织代码，将配置、数据库模型、路由等分别存放 ([使用Flask搭建个人博客 - Frost's Blog](https://frostming.com/2019/08-11/flask-blog/#:~:text=%E4%BD%BF%E7%94%A8%20Flask%20%E6%9D%A5%E5%86%99%E5%8D%9A%E5%AE%A2%EF%BC%8C%E9%A6%96%E5%85%88%E8%A6%81%E8%80%83%E8%99%91%E7%9A%84%E6%98%AF%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E5%AE%83%E4%B8%8D%E5%83%8F%20Django%20%E4%B8%80%E6%A0%B7%EF%BC%8C%E6%9C%89%E5%9B%BA%E5%AE%9A%E7%9A%84%E6%8E%A8%E8%8D%90%E7%BB%93%E6%9E%84%EF%BC%8C%E8%80%8C%E6%98%AF%E7%BB%99%E4%BA%86%E7%94%A8%E6%88%B7%E5%BE%88%E5%A4%A7%E7%9A%84%E8%87%AA%E7%94%B1%E7%A9%BA%E9%97%B4%E6%9D%A5%E7%BB%84%E7%BB%87%E9%A1%B9%E7%9B%AE%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E6%80%BB%E7%9A%84%E6%9D%A5%E8%AF%B4%EF%BC%8C%E6%9C%89%E4%B8%A4%E5%A4%A7%E6%B5%81%E6%B4%BE%EF%BC%9A))。项目初期的文件结构如下：

```
flask_blog/               <-- 项目根目录
│
├── run.py               <-- 应用启动脚本
├── config.py            <-- 配置文件
├── requirements.txt     <-- （可选）依赖列表
└── blog/                <-- Flask 应用包
    ├── __init__.py      <-- 创建 Flask 应用对象、初始化扩展
    ├── models.py        <-- 定义数据库模型（User、Post）
    ├── routes.py        <-- 定义路由和视图函数（API接口）
    └── static/          <-- 前端静态文件
        └── index.html   <-- 前端主页（HTML+JS）
```

这样的结构清晰划分了功能模块。例如，`models.py` 专注于数据模型，`routes.py` 专注于路由和业务逻辑，`static/index.html` 则是简单的前端页面。 ([使用Flask搭建个人博客 - Frost's Blog](https://frostming.com/2019/08-11/flask-blog/#:~:text=%E4%BD%BF%E7%94%A8%20Flask%20%E6%9D%A5%E5%86%99%E5%8D%9A%E5%AE%A2%EF%BC%8C%E9%A6%96%E5%85%88%E8%A6%81%E8%80%83%E8%99%91%E7%9A%84%E6%98%AF%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E5%AE%83%E4%B8%8D%E5%83%8F%20Django%20%E4%B8%80%E6%A0%B7%EF%BC%8C%E6%9C%89%E5%9B%BA%E5%AE%9A%E7%9A%84%E6%8E%A8%E8%8D%90%E7%BB%93%E6%9E%84%EF%BC%8C%E8%80%8C%E6%98%AF%E7%BB%99%E4%BA%86%E7%94%A8%E6%88%B7%E5%BE%88%E5%A4%A7%E7%9A%84%E8%87%AA%E7%94%B1%E7%A9%BA%E9%97%B4%E6%9D%A5%E7%BB%84%E7%BB%87%E9%A1%B9%E7%9B%AE%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E6%80%BB%E7%9A%84%E6%9D%A5%E8%AF%B4%EF%BC%8C%E6%9C%89%E4%B8%A4%E5%A4%A7%E6%B5%81%E6%B4%BE%EF%BC%9A))

接下来，我们将按步骤创建这些文件，并逐步填充代码。

## 二、初始化 Flask 应用

首先，配置 Flask 应用并确保基本的运行通畅。

1. **创建配置文件 `config.py`：** 在项目根目录下创建 `config.py`，编写配置类，用于集中管理配置参数，例如秘钥、数据库连接字符串等。例如：

```python
# config.py
class Config:
    SECRET_KEY = 'dev_secret_key'  # 签发 JWT 所需密钥，正式环境应使用随机值
    SQLALCHEMY_DATABASE_URI = 'sqlite:///blog.db'  # 使用 SQLite 数据库文件 blog.db
    SQLALCHEMY_TRACK_MODIFICATIONS = False         # 禁用修改追踪（节省开销）
```

以上设置中，`SECRET_KEY` 用于对会话和Token进行加密签名（请在生产环境中使用更复杂的值），`SQLALCHEMY_DATABASE_URI` 指定数据库为 SQLite，路径为当前项目目录下的 `blog.db` 文件。

2. **创建 Flask 应用包 `blog/__init__.py`：** 在 `blog` 目录的 `__init__.py` 中初始化 Flask 应用、数据库和跨域扩展。例如：

```python
# blog/__init__.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_cors import CORS

db = SQLAlchemy()  # 初始化数据库对象（延后初始化应用）

def create_app():
    app = Flask(__name__, static_folder='static', static_url_path='')
    # 加载配置
    app.config.from_object('config.Config')
    # 初始化扩展
    CORS(app)            # 允许跨域请求
    db.init_app(app)     # 绑定数据库实例到应用
    
    # 注册蓝图（路由模块），注意避免循环导入
    from blog import routes  
    app.register_blueprint(routes.bp)
    
    # 配置根路由返回前端页面
    @app.route('/')
    def index_page():
        return app.send_static_file('index.html')
    
    return app
```

上述代码要点：
- 创建 Flask 应用时指定了 `static_folder` 和 `static_url_path`，将 `blog/static` 目录作为静态文件夹，并将静态文件URL前缀设为空字符串，这样前端的 `index.html` 可以通过根路径直接访问。
- `CORS(app)` 开启跨域资源共享，以便前端在不同源域名（如本地静态页面或其他端口）请求我们的 API。 ([跨域资源共享 CORS 详解 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/04/cors.html#:~:text=CORS%E6%98%AF%E4%B8%80%E4%B8%AAW3C%E6%A0%87%E5%87%86%EF%BC%8C%E5%85%A8%E7%A7%B0%E6%98%AF%22%E8%B7%A8%E5%9F%9F%E8%B5%84%E6%BA%90%E5%85%B1%E4%BA%AB%22%EF%BC%88Cross))
- 定义全局的 `db`（SQLAlchemy实例）并通过 `db.init_app(app)` 完成与应用的绑定。
- 使用 Flask **蓝图（Blueprint）** 注册路由。我们会在稍后创建 `routes.py` 定义蓝图 `bp`。这里提前引用并注册它。
- 定义了一个根路由 `/` 返回静态文件 `index.html`。这样当用户在浏览器访问 `http://127.0.0.1:5000/` 时，会直接看到我们稍后编写的前端页面。

3. **创建启动脚本 `run.py`：** 在项目根目录下创建 `run.py` 用于运行应用：

```python
# run.py
from blog import create_app, db

app = create_app()

# 数据库表创建（如不存在则创建）
with app.app_context():
    db.create_all()

if __name__ == '__main__':
    app.run(debug=True)
```

`create_app()` 工厂函数返回 Flask 应用对象，接着通过 `app.app_context()` 创建应用上下文并调用 `db.create_all()` 在 SQLite 中创建表。然后运行 `app.run(debug=True)` 启动开发服务器。（开启 `debug=True` 以便在开发时自动重载代码并提供调试信息。）

现在，可以运行 `python run.py` 启动 Flask 应用。第一次运行会在项目目录下生成一个 `blog.db` SQLite 数据库文件（由于 `db.create_all()` 创建了空的表结构）。此时应用虽然启动了，但我们还没有定义任何接口逻辑，暂时只有根路径会返回一个空的 `index.html` 文件（我们尚未编写前端页面内容）。接下来，我们开始实现后端的核心功能。

## 三、数据库模型设计 (Model)

个人博客主要涉及两类数据：**用户 (User)** 和 **文章 (Post)**。我们将在 `blog/models.py` 中使用 Flask-SQLAlchemy 定义这两个模型，并设置它们之间的关系。

打开或创建 `blog/models.py`，编写如下代码：

```python
# blog/models.py
from blog import db
from werkzeug.security import generate_password_hash, check_password_hash
from datetime import datetime

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)
    posts = db.relationship('Post', backref='author', lazy=True)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        return check_password_hash(self.password_hash, password)


class Post(db.Model):
    __tablename__ = 'posts'
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
```

**模型说明：**

- `User` 模型：  
  - `username`：用户名，最大长度80字符，要求唯一 (`unique=True`) 且不能为空。  
  - `password_hash`：密码哈希后的值，长度128。我们将存储经过哈希处理的密码，而不直接保存明文密码，提高安全性。Flask 框架自带 `werkzeug.security` 提供了方便的密码哈希函数。`generate_password_hash` 会对密码进行加盐哈希 ([Flask加盐密码生成和验证函数
 - Flask中文学习网](https://flask123.sinaapp.com/article/40/#:~:text=))，生成类似`方法$盐$哈希值`的字符串；`check_password_hash` 则用于验证输入密码是否与存储的哈希匹配。通过定义 `set_password()` 和 `check_password()` 方法，方便在用户注册和登录时设置/验证密码。  
  - `posts`：定义与 `Post` 模型的一对多关系。`backref='author'` 意味着每个 Post 实例可以通过 `post.author` 访问对应的 User 实例 ([使用Flask搭建个人博客 - Frost's Blog](https://frostming.com/2019/08-11/flask-blog/#:~:text=1,%E6%8C%89%E6%A8%A1%E5%9D%97%E5%88%92%E5%88%86%EF%BC%8C%E5%88%86%E6%88%90%E6%93%8D%E4%BD%9C%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%20models%20%E9%83%A8%E5%88%86%EF%BC%8C%E6%B8%B2%E6%9F%93%E8%A7%86%E5%9B%BE%E7%9A%84%20views%20%E9%83%A8%E5%88%86%EF%BC%8C%E5%A4%84%E7%90%86%E6%A8%A1%E6%9D%BF%E7%9A%84%E9%83%A8%E5%88%86%E7%AD%89%E7%AD%89%E3%80%82))。`lazy=True` 表示 SQLAlchemy 以惰性加载的方式获取数据。
  
  *安全提示：* 永远不要以明文形式保存用户密码。应采用加盐哈希方式存储 ([Flask加盐密码生成和验证函数
 - Flask中文学习网](https://flask123.sinaapp.com/article/40/#:~:text=)) ([Flask加盐密码生成和验证函数
 - Flask中文学习网](https://flask123.sinaapp.com/article/40/#:~:text=))。上面使用的 `generate_password_hash` 默认使用PBKDF2算法和随机盐，能够提供较好的安全性。

- `Post` 模型：  
  - `title`：文章标题。  
  - `content`：文章内容，使用 Text 类型存储长文本。  
  - `created_at`：发布时间，默认值为当前时间 `datetime.utcnow`（UTC 时间）。  
  - `user_id`：关联到作者用户的外键，`ForeignKey('users.id')` 表示引用 `users` 表（即 User模型）中的 id。这建立了用户和文章的关联，每篇文章属于某个用户。

定义好模型后，我们的数据库结构就明确了：**一个用户可以有多篇文章，每篇文章有一个作者**。

> **小结：**本节我们使用 Flask-SQLAlchemy 定义了 User 和 Post 两个模型，并通过外键和关系建立了二者之间的联系。密码将以哈希形式存储，以确保安全。

## 四、实现用户注册与登录 (JSON API)

有了模型，接下来实现用户相关的 API 接口，包括注册新用户和登录获取 Token。这些路由会写在 `blog/routes.py` 中，我们将使用 Flask 提供的路由装饰器和视图函数来处理请求和返回响应。

先在 `blog/routes.py` 中创建 Flask **蓝图**（Blueprint）。蓝图可以将一组相关路由组织在一起，便于模块化管理。本项目所有后端接口将挂载在 `/api` 路径下，我们创建一个名为 `api` 的蓝图：

```python
# blog/routes.py
from flask import Blueprint, request, jsonify, current_app
from blog import db
from blog.models import User, Post
import jwt
from datetime import datetime, timedelta

bp = Blueprint('api', __name__, url_prefix='/api')
```

这里我们导入了 `jwt` 库用于生成和验证 JSON Web Token。**JWT**（JSON Web Token）是一种紧凑的、自包含的令牌格式，用于在各方之间以 JSON 形式安全地传输信息 ([JSON Web Token 介绍 什么是jwt  - jwt 中文网 官网](https://jwt.p2hp.com/introduction#:~:text=JSON%20Web%20Token%20%EF%BC%88JWT%EF%BC%89%20%E6%98%AF%E4%B8%80%E7%A7%8D%E5%BC%80%E6%94%BE%E6%A0%87%E5%87%86,%E7%AE%97%E6%B3%95%EF%BC%89%E6%88%96%E4%BD%BF%E7%94%A8%20RSA%20%E6%88%96%20ECDSA%20%E7%9A%84%E5%85%AC%E9%92%A5%2F%E7%A7%81%E9%92%A5%E5%AF%B9%E8%BF%9B%E8%A1%8C%E7%AD%BE%E5%90%8D%E3%80%82))。本项目中，登录成功后服务器将签发 JWT，客户端在后续请求中携带该 Token 实现认证。

下面实现 **注册接口** `/api/register` 和 **登录接口** `/api/login`：

```python
@bp.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'message': '用户名和密码是必需的'}), 400
    username = data['username']
    password = data['password']
    # 检查用户名是否已存在
    if User.query.filter_by(username=username).first():
        return jsonify({'message': '用户名已存在'}), 400
    # 创建新用户
    user = User(username=username)
    user.set_password(password)  # 哈希密码后保存
    db.session.add(user)
    db.session.commit()
    current_app.logger.info(f"New user registered: {username}")
    return jsonify({'message': '用户注册成功，请登录'}), 201
```

- 该视图函数期望收到 JSON 格式的请求体，包括 `username` 和 `password`。通过 `request.get_json()` 获取请求数据并进行基本验证，如果缺少用户名或密码，返回 400 错误。  
- 调用 `User.query.filter_by(username=...)` 查询数据库，如果已存在同名用户，则返回错误提示。  
- 若用户名可用，则创建 `User` 实例，调用前面定义的 `set_password()` 方法对密码进行哈希处理并保存。随后 `db.session.add(user)` 将用户添加到会话，`db.session.commit()` 提交事务，将新用户写入数据库。  
- 使用 `current_app.logger.info()` 记录日志，注明有新用户注册（日志会打印在服务器控制台上，可帮助调试）。 ([Logging — Flask Documentation (3.1.x)](https://flask.palletsprojects.com/en/stable/logging/#:~:text=Flask%20uses%20standard%20Python%20logging,to%20log%20your%20own%20messages))  
- 返回 JSON 消息 `{'message': '用户注册成功，请登录'}`，HTTP 状态码 201（Created）。

接下来是登录接口：

```python
@bp.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    if not data or not data.get('username') or not data.get('password'):
        return jsonify({'message': '用户名和密码是必需的'}), 400
    username = data['username']
    password = data['password']
    user = User.query.filter_by(username=username).first()
    if not user or not user.check_password(password):
        current_app.logger.info(f"Failed login attempt for username: {username}")
        return jsonify({'message': '用户名或密码错误'}), 401
    # 生成 JWT Token
    token = jwt.encode({
            'user_id': user.id,
            'exp': datetime.utcnow() + timedelta(hours=1)  # Token 1小时后过期
        },
        current_app.config['SECRET_KEY'],
        algorithm='HS256'
    )
    current_app.logger.info(f"User logged in: {username}")
    return jsonify({
        'message': '登录成功',
        'token': token,
        'username': user.username
    }), 200
```

- 登录接口同样接受 JSON（用户名和密码），验证输入是否合法后，查询用户是否存在，并使用 `check_password()` 校验密码。若用户名不存在或密码不正确，记录一次登录失败日志并返回401 Unauthorized。  
- 如果验证通过，则使用 PyJWT (`jwt.encode`) 生成一个 JWT**令牌 (Token)**。我们将用户ID和过期时间(`exp`)放入 Token payload，并用应用配置中的 `SECRET_KEY` 签名生成 Token，算法采用 HS256。Token 会是一串经过加密签名的字符串，类似于`<header>.<payload>.<signature>`。客户端拿到 Token 后，应在后续请求的 HTTP头部加入该 Token 以证明身份。我们这里设置 Token 有效期1小时，可以根据需要调整。 ([JSON Web Token 介绍 什么是jwt  - jwt 中文网 官网](https://jwt.p2hp.com/introduction#:~:text=%E4%BB%A5%E4%B8%8B%E6%98%AF%20JSON%20Web%20Tokens%20%E6%9C%89%E7%94%A8%E7%9A%84%E4%B8%80%E4%BA%9B%E5%9C%BA%E6%99%AF%3A))  
- 登录成功后，返回 JSON 数据，其中包含一个欢迎消息、Token 字符串以及用户名。客户端收到后应保存 Token，例如存储在浏览器的本地存储或内存中，用于调用需要认证的接口。

**关于 Token 验证：** 本项目采用 JWT 作为认证令牌，这是一种无状态的认证方式。服务器生成 Token 后不在服务端保存会话信息，后续请求中只要携带该 Token，服务器通过验证其数字签名即可信任其中包含的用户信息 ([JSON Web Token 入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#:~:text=%E6%95%B0%E6%8D%AE%E6%8C%81%E4%B9%85%E5%8C%96%EF%BC%8C%E5%86%99%E5%85%A5%E6%95%B0%E6%8D%AE%E5%BA%93%E6%88%96%E5%88%AB%E7%9A%84%E6%8C%81%E4%B9%85%E5%B1%82%E3%80%82%E5%90%84%E7%A7%8D%E6%9C%8D%E5%8A%A1%E6%94%B6%E5%88%B0%E8%AF%B7%E6%B1%82%E5%90%8E%EF%BC%8C%E9%83%BD%E5%90%91%E6%8C%81%E4%B9%85%E5%B1%82%E8%AF%B7%E6%B1%82%E6%95%B0%E6%8D%AE%E3%80%82%E8%BF%99%E7%A7%8D%E6%96%B9%E6%A1%88%E7%9A%84%E4%BC%98%E7%82%B9%E6%98%AF%E6%9E%B6%E6%9E%84%E6%B8%85%E6%99%B0%EF%BC%8C%E7%BC%BA%E7%82%B9%E6%98%AF%E5%B7%A5%E7%A8%8B%E9%87%8F%E6%AF%94%E8%BE%83%E5%A4%A7%E3%80%82%E5%8F%A6%E5%A4%96%EF%BC%8C%E6%8C%81%E4%B9%85%E5%B1%82%E4%B8%87%E4%B8%80%E6%8C%82%E4%BA%86%EF%BC%8C%E5%B0%B1%E4%BC%9A%E5%8D%95%E7%82%B9%E5%A4%B1%E8%B4%A5%E3%80%82)) ([JSON Web Token 入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#:~:text=%E4%BB%A5%E5%90%8E%EF%BC%8C%E7%94%A8%E6%88%B7%E4%B8%8E%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%80%9A%E4%BF%A1%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E9%83%BD%E8%A6%81%E5%8F%91%E5%9B%9E%E8%BF%99%E4%B8%AA%20JSON%20%E5%AF%B9%E8%B1%A1%E3%80%82%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%8C%E5%85%A8%E5%8F%AA%E9%9D%A0%E8%BF%99%E4%B8%AA%E5%AF%B9%E8%B1%A1%E8%AE%A4%E5%AE%9A%E7%94%A8%E6%88%B7%E8%BA%AB%E4%BB%BD%E3%80%82%E4%B8%BA%E4%BA%86%E9%98%B2%E6%AD%A2%E7%94%A8%E6%88%B7%E7%AF%A1%E6%94%B9%E6%95%B0%E6%8D%AE%EF%BC%8C%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%9C%A8%E7%94%9F%E6%88%90%E8%BF%99%E4%B8%AA%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E4%BC%9A%E5%8A%A0%E4%B8%8A%E7%AD%BE%E5%90%8D%EF%BC%88%E8%AF%A6%E8%A7%81%E5%90%8E%E6%96%87%EF%BC%89%E3%80%82))。相对于传统的服务器会话(session)方式，使用 Token 更适合前后端分离和分布式集群环境，因为服务端无需维护会话状态，扩展性更好 ([JSON Web Token 入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#:~:text=%E8%BF%99%E7%A7%8D%E6%A8%A1%E5%BC%8F%E7%9A%84%E9%97%AE%E9%A2%98%E5%9C%A8%E4%BA%8E%EF%BC%8C%E6%89%A9%E5%B1%95%E6%80%A7%EF%BC%88scaling%EF%BC%89%E4%B8%8D%E5%A5%BD%E3%80%82%E5%8D%95%E6%9C%BA%E5%BD%93%E7%84%B6%E6%B2%A1%E6%9C%89%E9%97%AE%E9%A2%98%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%98%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%9B%86%E7%BE%A4%EF%BC%8C%E6%88%96%E8%80%85%E6%98%AF%E8%B7%A8%E5%9F%9F%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%AF%BC%E5%90%91%E6%9E%B6%E6%9E%84%EF%BC%8C%E5%B0%B1%E8%A6%81%E6%B1%82%20session%20%E6%95%B0%E6%8D%AE%E5%85%B1%E4%BA%AB%EF%BC%8C%E6%AF%8F%E5%8F%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%83%BD%E8%83%BD%E5%A4%9F%E8%AF%BB%E5%8F%96%20session%E3%80%82)) ([JSON Web Token 入门教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html#:~:text=%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%B0%B1%E4%B8%8D%E4%BF%9D%E5%AD%98%E4%BB%BB%E4%BD%95%20session%20%E6%95%B0%E6%8D%AE%E4%BA%86%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E8%AF%B4%EF%BC%8C%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%98%E6%88%90%E6%97%A0%E7%8A%B6%E6%80%81%E4%BA%86%EF%BC%8C%E4%BB%8E%E8%80%8C%E6%AF%94%E8%BE%83%E5%AE%B9%E6%98%93%E5%AE%9E%E7%8E%B0%E6%89%A9%E5%B1%95%E3%80%82))。

> **小结：**我们实现了用户注册和登录两个 API。注册接口将新用户写入数据库并返回提示；登录接口验证用户凭据，成功则签发一个 JWT Token。我们通过日志记录每次注册和登录的行为（成功或失败），并通过合理的HTTP状态码告知结果。此时可以使用工具（如 **Postman** 或命令行 `curl`）来测试这两个接口。例如：
> ```bash
> # 注册用户
> curl -X POST -H "Content-Type: application/json" \
>      -d '{"username": "test", "password": "123456"}' \
>      http://127.0.0.1:5000/api/register 
> 
> # 登录获取 Token
> curl -X POST -H "Content-Type: application/json" \
>      -d '{"username": "test", "password": "123456"}' \
>      http://127.0.0.1:5000/api/login 
> ```
> 登录成功将返回一个 `token`，可以在后续请求的HTTP头添加 `Authorization: Bearer <token>` 来访问受保护资源。

## 五、实现博客文章的 CRUD 接口

用户登录获取 Token 后，就可以执行受保护的操作，比如发布文章、编辑或删除自己的文章。在实现这些功能之前，我们需要一种机制验证每个请求附带的 Token 是否有效并获取对应的用户。

### 5.1 Token 认证机制

在 `routes.py` 中，我们可以编写一个辅助函数用于从请求头提取 Token、进行验证并获取当前用户。由于多个视图都会用到类似的认证检查逻辑，抽取成函数可以避免重复。示例如下：

```python
def get_current_user_from_token():
    auth_header = request.headers.get('Authorization')
    if not auth_header or not auth_header.startswith('Bearer '):
        return None, jsonify({'message': '缺少认证令牌'}), 401
    token = auth_header.split(' ')[1]  # 提取 "Bearer " 之后的实际token
    try:
        payload = jwt.decode(token, current_app.config['SECRET_KEY'], algorithms=["HS256"])
    except jwt.ExpiredSignatureError:
        return None, jsonify({'message': '令牌已过期'}), 401
    except jwt.InvalidTokenError:
        return None, jsonify({'message': '无效的令牌'}), 401
    user = User.query.get(payload['user_id'])
    if not user:
        return None, jsonify({'message': '用户不存在'}), 401
    return user, None, None
```

这个函数完成了以下工作：
- 从请求头获取 `Authorization` 字段，并检查格式是否为 “Bearer \<Token\>”。如果没有提供或格式不对，返回错误提示和401状态。  
- 使用 `jwt.decode` 解码 Token。若 Token 已过期或无效（篡改/不可信），会抛出异常，分别返回相应提示。  
- 如果解码成功，从 payload 中取出 `user_id`，查询数据库获取 User 对象。如果用户不存在（例如用户已被删除），返回401。  
- 若以上都通过，则返回 User 对象。（函数返回三个值，其中第二、第三个值在出错时用于携带错误响应及状态码。）

有了这个辅助函数，在需要认证的接口里就可以方便地获取当前登录用户，并在Token无效时统一返回错误。接下来实现文章相关的接口。

### 5.2 发布新文章

**接口：** `POST /api/posts` （需要登录）

```python
@bp.route('/posts', methods=['POST'])
def create_post():
    user, error_response, status = get_current_user_from_token()
    if not user:
        return error_response, status
    data = request.get_json()
    if not data or not data.get('title') or not data.get('content'):
        return jsonify({'message': '标题和内容是必需的'}), 400
    title = data['title']
    content = data['content']
    # 创建 Post 对象
    post = Post(title=title, content=content, author=user)
    db.session.add(post)
    try:
        db.session.commit()
    except Exception as e:
        current_app.logger.error(f"Error creating post: {e}")
        return jsonify({'message': '无法发布文章'}), 500
    current_app.logger.info(f"Post created (id={post.id}) by user: {user.username}")
    return jsonify({
        'message': '文章发布成功',
        'post': {
            'id': post.id,
            'title': post.title,
            'content': post.content,
            'author': user.username,
            'created_at': post.created_at.strftime("%Y-%m-%d %H:%M:%S")
        }
    }), 201
```

- 调用前面编写的 `get_current_user_from_token()` 检查请求的认证。如果未通过认证（函数返回 `user=None`），直接返回错误响应。这样确保只有登录用户才能执行后续操作。
- 解析请求 JSON，确保 `title` 和 `content` 字段不为空，否则返回 400。
- 使用提交的数据创建一个新的 `Post` 实例，并通过 `author=user` 将当前用户设置为作者（这利用了我们在模型定义的 `backref`，等价于 `post.user_id = user.id`）。
- 添加到数据库会话并提交。这里用 `try...except` 捕获可能的数据库错误（例如数据库文件写入失败等），如出现异常则记录日志并返回服务器错误。
- 日志记录新文章的创建（包含文章ID和作者用户名）。
- 返回 JSON，包括成功消息以及新创建的文章数据（id、标题、内容、作者、创建时间）。HTTP状态码用201表示资源已创建。

### 5.3 获取文章列表

**接口：** `GET /api/posts` （公共接口，任何人可访问）

```python
@bp.route('/posts', methods=['GET'])
def get_posts():
    posts = Post.query.order_by(Post.created_at.desc()).all()
    result = []
    for post in posts:
        result.append({
            'id': post.id,
            'title': post.title,
            'content': post.content,
            'author': post.author.username if post.author else None,
            'created_at': post.created_at.strftime("%Y-%m-%d %H:%M:%S")
        })
    return jsonify(result), 200
```

- 查询数据库中所有文章 (`Post.query.all()`)，按创建时间倒序排列（最新的文章最先）。  
- 将每篇文章转换为字典并加入结果列表。其中，通过 `post.author.username` 获取作者用户名（这是SQLAlchemy通过关系自动提供的属性）。如果没有作者（理论上不会发生，因为 `user_id` 非空），则返回 None。时间 `created_at` 格式化成可读的字符串。
- 使用 `jsonify` 返回文章列表（JSON数组）。即使列表为空也会返回 `[]`。

这样客户端调用该接口就能获取所有文章的数据，用于展示。

### 5.4 获取单篇文章详情

**接口：** `GET /api/posts/<int:post_id>` （公共接口）

```python
@bp.route('/posts/<int:post_id>', methods=['GET'])
def get_post(post_id):
    post = Post.query.get(post_id)
    if not post:
        return jsonify({'message': '文章未找到'}), 404
    return jsonify({
        'id': post.id,
        'title': post.title,
        'content': post.content,
        'author': post.author.username if post.author else None,
        'created_at': post.created_at.strftime("%Y-%m-%d %H:%M:%S")
    }), 200
```

- 通过 `Post.query.get(post_id)` 按主键查询文章。若不存在，返回404错误。
- 若找到，返回该文章的详细信息。由于我们在列表接口已经返回了全文，这里的实现与列表类似。实际上，在真实应用中列表接口通常只提供摘要，这里简化处理，两者返回内容类似。

### 5.5 编辑文章

**接口：** `PUT /api/posts/<int:post_id>` （需要登录）

```python
@bp.route('/posts/<int:post_id>', methods=['PUT'])
def update_post(post_id):
    user, error_response, status = get_current_user_from_token()
    if not user:
        return error_response, status
    post = Post.query.get(post_id)
    if not post:
        return jsonify({'message': '文章未找到'}), 404
    # 权限检查：只有作者本人可以编辑
    if post.author != user:
        return jsonify({'message': '没有权限编辑这篇文章'}), 403
    data = request.get_json()
    if not data:
        return jsonify({'message': '缺少要更新的数据'}), 400
    # 根据提供的数据更新字段
    if 'title' in data:
        post.title = data['title']
    if 'content' in data:
        post.content = data['content']
    try:
        db.session.commit()
    except Exception as e:
        current_app.logger.error(f"Error updating post {post_id}: {e}")
        return jsonify({'message': '更新文章失败'}), 500
    current_app.logger.info(f"Post id={post.id} updated by user: {user.username}")
    return jsonify({
        'message': '文章更新成功',
        'post': {
            'id': post.id,
            'title': post.title,
            'content': post.content,
            'author': user.username,
            'created_at': post.created_at.strftime("%Y-%m-%d %H:%M:%S")
        }
    }), 200
```

- 通过 `get_current_user_from_token()` 确保用户已登录。
- 查询指定ID的文章，不存在则404。
- 检查当前用户是否是该文章的作者（`post.author != user`），如果不是本人则返回403禁止访问。这保证用户只能编辑自己的文章。
- 获取请求 JSON 数据，允许部分更新：如果提供了 `title` 字段则更新标题，提供了 `content` 则更新内容。没有提供任何内容则返回400。
- 提交数据库事务。若发生错误（如数据库文件锁等），记录日志并返回500。
- 日志记录文章被编辑的事件。
- 返回更新后的文章数据和成功消息。

注意，我们没有更新 `created_at` 字段。这意味着编辑不会改变文章的发布日期。如有需要，可以在模型中增加 `updated_at` 字段来追踪修改时间。

### 5.6 删除文章

**接口：** `DELETE /api/posts/<int:post_id>` （需要登录）

```python
@bp.route('/posts/<int:post_id>', methods=['DELETE'])
def delete_post(post_id):
    user, error_response, status = get_current_user_from_token()
    if not user:
        return error_response, status
    post = Post.query.get(post_id)
    if not post:
        return jsonify({'message': '文章未找到'}), 404
    if post.author != user:
        return jsonify({'message': '没有权限删除这篇文章'}), 403
    db.session.delete(post)
    try:
        db.session.commit()
    except Exception as e:
        current_app.logger.error(f"Error deleting post {post_id}: {e}")
        return jsonify({'message': '删除文章失败'}), 500
    current_app.logger.info(f"Post id={post.id} deleted by user: {user.username}")
    return jsonify({'message': '文章已删除'}), 200
```

- 同样先认证用户，查询文章。
- 权限检查：只有作者可删除自己的文章，否则返回403。
- 调用 `db.session.delete(post)` 将文章从会话中删除，并提交事务。 
- 错误处理和日志记录与前面类似。
- 返回一条简单的成功消息。前端拿到这条消息后，可从界面上移除被删文章。

到此，博客文章相关的 CRUD 接口全部完成。我们已经涵盖了**路由与视图函数**（使用装饰器定义路径，处理 HTTP 方法）、**请求与响应**（接收 JSON，返回 JSON，设置相应状态码）、**数据验证**（如检查必填字段、权限）等 Flask 核心功能点。

> **小结：** 我们实现了文章的新增、查询、修改、删除接口。其中发表、编辑、删除都需要先验证 Token，并且只有文章作者本人可以执行编辑或删除。这体现了**认证和授权**的逻辑：认证通过Token验证用户身份，授权通过检查用户与资源的关系决定操作是否允许。我们使用合理的HTTP状态码：401用于未认证，403用于无权限，404用于资源不存在，200/201用于成功，并以统一的JSON格式返回结果，方便前端处理。

## 六、跨域处理（CORS）

由于我们的项目采用前后端分离架构，后端API可能会被部署在与前端不同的域名或端口下。这种情况下，浏览器出于安全策略（同源策略）的限制，默认会阻止前端页面向不同源的后端发送 AJAX 请求。为了解除这个限制，我们需要在后端启用 **跨源资源共享（CORS）**。 ([跨域资源共享 CORS 详解 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/04/cors.html#:~:text=CORS%E6%98%AF%E4%B8%80%E4%B8%AAW3C%E6%A0%87%E5%87%86%EF%BC%8C%E5%85%A8%E7%A7%B0%E6%98%AF%22%E8%B7%A8%E5%9F%9F%E8%B5%84%E6%BA%90%E5%85%B1%E4%BA%AB%22%EF%BC%88Cross))

在本项目中，我们已经通过 `Flask-CORS` 扩展非常简洁地实现了这一点：

```python
from flask_cors import CORS
# ...
app = Flask(__name__, ...)
CORS(app)
```

`CORS(app)` 默认允许来自任何域的跨域请求。如果需要，可以进行更精细的配置，比如限定允许的域、方法等。跨域启用后，浏览器在发送 AJAX 请求时会先发一个预检请求（OPTIONS），后端返回适当的 CORS 响应头（如 `Access-Control-Allow-Origin`）表明自己接受跨域请求，这样浏览器才会继续进行真正的请求 ([跨域资源共享 CORS 详解 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/04/cors.html#:~:text=CORS%E9%9C%80%E8%A6%81%E6%B5%8F%E8%A7%88%E5%99%A8%E5%92%8C%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8C%E6%97%B6%E6%94%AF%E6%8C%81%E3%80%82%E7%9B%AE%E5%89%8D%EF%BC%8C%E6%89%80%E6%9C%89%E6%B5%8F%E8%A7%88%E5%99%A8%E9%83%BD%E6%94%AF%E6%8C%81%E8%AF%A5%E5%8A%9F%E8%83%BD%EF%BC%8CIE%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8D%E8%83%BD%E4%BD%8E%E4%BA%8EIE10%E3%80%82))。由于我们使用了 Flask-CORS，它自动帮我们处理了这些预检逻辑，我们无需手动设置响应头。

> **扩展阅读：** CORS 是一个 W3C 标准，其作用是允许浏览器向跨域服务器发送 XMLHttpRequest 请求，克服 AJAX 只能同源使用的限制 ([跨域资源共享 CORS 详解 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/04/cors.html#:~:text=CORS%E6%98%AF%E4%B8%80%E4%B8%AAW3C%E6%A0%87%E5%87%86%EF%BC%8C%E5%85%A8%E7%A7%B0%E6%98%AF%22%E8%B7%A8%E5%9F%9F%E8%B5%84%E6%BA%90%E5%85%B1%E4%BA%AB%22%EF%BC%88Cross))。实现CORS通信，浏览器和服务器都需要支持相应规范。服务器通过设置特殊的HTTP响应头来指示浏览器允许跨域请求。有关CORS更详细的原理，可以参考阮一峰的文章 ([跨域资源共享 CORS 详解 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/04/cors.html#:~:text=CORS%E6%98%AF%E4%B8%80%E4%B8%AAW3C%E6%A0%87%E5%87%86%EF%BC%8C%E5%85%A8%E7%A7%B0%E6%98%AF%22%E8%B7%A8%E5%9F%9F%E8%B5%84%E6%BA%90%E5%85%B1%E4%BA%AB%22%EF%BC%88Cross))。在 Flask 中，除了使用 Flask-CORS，也可以手动在响应中添加头部实现跨域，但利用扩展更为便利。

## 七、错误处理与日志记录

一个健壮的应用需要良好的错误处理机制和日志记录。我们已经在部分代码中体现了这些：对常见错误情况返回了适当的HTTP状态码和消息，并使用 `current_app.logger` 记录了重要事件（如用户登录失败、数据库操作异常等）。

### 7.1 全局错误处理

Flask允许我们通过装饰器针对特定错误代码注册全局处理函数。例如，为统一API的错误返回格式，我们可以让所有404错误都返回JSON而不是默认的HTML错误页。在 `routes.py` 末尾添加：

```python
@bp.app_errorhandler(404)
def handle_404(error):
    return jsonify({'message': '接口不存在'}), 404

@bp.app_errorhandler(500)
def handle_500(error):
    current_app.logger.error(f"Internal server error: {error}, path: {request.path}")
    return jsonify({'message': '服务器内部错误'}), 500
```

这里使用了 `bp.app_errorhandler` 将错误处理关联到蓝图上（也可用 `app.errorhandler` 注册全局错误处理）。第一个处理函数拦截404，将返回 JSON `{message: '接口不存在'}`，状态码404。第二个处理函数拦截500服务器错误，先记录错误日志（包含错误信息和请求路径），然后返回 JSON 错误消息和500状态码。

通过这样的全局处理，即使发生未捕获的异常导致500错误，客户端也能收到统一格式的JSON错误响应，而不是看到Flask默认的HTML错误页。

### 7.2 日志配置与使用

Flask 使用 Python 内置的 logging 模块记录应用日志，`app.logger` 是应用自带的日志记录器 ([Logging — Flask Documentation (3.1.x)](https://flask.palletsprojects.com/en/stable/logging/#:~:text=Flask%20uses%20standard%20Python%20logging,to%20log%20your%20own%20messages))。默认情况下，在 debug 模式下，Flask 会输出 INFO 级别以上的日志到控制台，其中包括每个请求的访问日志和我们在代码中调用的 `app.logger.info/error` 日志。

在代码中，我们已经示范了日志的使用：
- `current_app.logger.info(...)` 用于记录一般信息，例如用户注册、登录成功或失败、文章的创建和删除等。
- `current_app.logger.error(...)` 用于记录错误，如数据库操作异常或者500错误处理里捕获的异常。

这些日志默认会打印在开发服务器的控制台中。如果希望将日志保存到文件，可以自行配置 logging。例如，在应用启动时通过 `logging.basicConfig(filename='app.log', level=logging.INFO)` 将日志写入文件，或者使用 `RotatingFileHandler` 等实现更健壮的日志轮转。

日志对于调试和运维非常重要：通过日志我们可以追踪用户行为（如谁登录了，谁发布了文章）以及排查错误原因（例如查看异常栈信息）。本项目中我们简要地展示了如何使用 Flask 的日志系统，更高级的用法可以参阅官方文档 ([Logging — Flask Documentation (3.1.x)](https://flask.palletsprojects.com/en/stable/logging/#:~:text=When%20you%20want%20to%20configure,before%20creating%20the%20application%20object))。

> **小结：** 通过全局错误处理，我们让常见错误返回统一的JSON格式，提升了API的一致性。通过日志，我们可以方便地监控应用的运行状态。在开发过程中，日志输出可以帮助我们定位问题；在生产环境中，可以考虑将重要日志（如500错误）发送到外部监控系统或日志收集系统，以便及时发现并处理问题。

## 八、前端页面与接口联调

后端所有 API 已完成，接下来我们构建一个**简单的前端页面**来调用这些接口，实现博客的基本交互功能。这里我们不使用复杂的前端框架，仅通过纯 HTML + 原生 JavaScript（Fetch API）来演示。

创建并打开 `blog/static/index.html` 文件，并编写以下内容：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <title>个人博客示例</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    #loginSection, #registerSection, #postSection { margin-bottom: 20px; }
    #registerSection, #postSection { display: none; }  /* 初始隐藏注册表单和发帖区 */
    .post { border-bottom: 1px solid #ccc; padding: 10px 0; }
    .post h3 { margin: 0; }
    .post .meta { color: gray; font-size: 0.9em; }
    button { margin-right: 5px; }
  </style>
</head>
<body>
  <h1>个人博客 Demo</h1>
  
  <!-- 登录表单 -->
  <div id="loginSection">
    <h2>登录</h2>
    用户名: <input type="text" id="loginUsername"><br/>
    密码: <input type="password" id="loginPassword"><br/>
    <button id="loginBtn">登录</button>
    <button id="showRegisterBtn">没有账号？注册</button>
  </div>
  
  <!-- 注册表单 -->
  <div id="registerSection">
    <h2>注册</h2>
    用户名: <input type="text" id="registerUsername"><br/>
    密码: <input type="password" id="registerPassword"><br/>
    <button id="registerBtn">注册</button>
    <button id="cancelRegisterBtn">取消</button>
  </div>
  
  <!-- 发帖和注销区域（登录后可见） -->
  <div id="postSection">
    <div>
      <span id="welcomeMsg"></span>
      <button id="logoutBtn">退出登录</button>
    </div>
    <h2>发表新文章</h2>
    标题: <input type="text" id="postTitle"><br/>
    内容:<br/>
    <textarea id="postContent" rows="4" cols="50"></textarea><br/>
    <button id="publishBtn">发布文章</button>
  </div>
  
  <!-- 文章列表 -->
  <h2>文章列表</h2>
  <div id="postsList"><!-- 动态加载文章 --></div>
  
<script>
  const API_BASE = '/api';
  let currentUser = { username: null, token: null };
  
  // 获取页面元素
  const loginSection = document.getElementById('loginSection');
  const registerSection = document.getElementById('registerSection');
  const postSection = document.getElementById('postSection');
  const loginUsername = document.getElementById('loginUsername');
  const loginPassword = document.getElementById('loginPassword');
  const loginBtn = document.getElementById('loginBtn');
  const showRegisterBtn = document.getElementById('showRegisterBtn');
  const registerUsername = document.getElementById('registerUsername');
  const registerPassword = document.getElementById('registerPassword');
  const registerBtn = document.getElementById('registerBtn');
  const cancelRegisterBtn = document.getElementById('cancelRegisterBtn');
  const welcomeMsg = document.getElementById('welcomeMsg');
  const logoutBtn = document.getElementById('logoutBtn');
  const postTitle = document.getElementById('postTitle');
  const postContent = document.getElementById('postContent');
  const publishBtn = document.getElementById('publishBtn');
  const postsList = document.getElementById('postsList');
  
  // 页面加载时检查本地存储的Token，保持登录状态
  if (localStorage.getItem('token')) {
    currentUser.token = localStorage.getItem('token');
    currentUser.username = localStorage.getItem('username');
    updateUIForLogin();
  }
  
  // 显示注册表单
  showRegisterBtn.onclick = () => {
    loginSection.style.display = 'none';
    registerSection.style.display = 'block';
  };
  // 取消注册，返回登录
  cancelRegisterBtn.onclick = () => {
    registerSection.style.display = 'none';
    loginSection.style.display = 'block';
  };
  
  // 登录请求
  loginBtn.onclick = () => {
    const username = loginUsername.value;
    const password = loginPassword.value;
    if (!username || !password) {
      alert('请输入用户名和密码');
      return;
    }
    fetch(`${API_BASE}/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password })
    })
    .then(res => res.json().then(data => ({ status: res.status, body: data })))
    .then(result => {
      if (result.status === 200) {
        // 登录成功
        currentUser.token = result.body.token;
        currentUser.username = result.body.username;
        localStorage.setItem('token', currentUser.token);
        localStorage.setItem('username', currentUser.username);
        updateUIForLogin();
        fetchPosts();  // 刷新文章列表（以显示编辑/删除按钮）
      } else {
        alert(result.body.message || '登录失败');
      }
    })
    .catch(err => console.error('Login error:', err));
  };
  
  // 注册请求
  registerBtn.onclick = () => {
    const username = registerUsername.value;
    const password = registerPassword.value;
    if (!username || !password) {
      alert('请输入用户名和密码');
      return;
    }
    fetch(`${API_BASE}/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password })
    })
    .then(res => res.json().then(data => ({ status: res.status, body: data })))
    .then(result => {
      if (result.status === 201) {
        alert('注册成功，请登录');
        // 切换回登录界面并填充用户名
        registerSection.style.display = 'none';
        loginSection.style.display = 'block';
        loginUsername.value = username;
        loginPassword.value = '';
      } else {
        alert(result.body.message || '注册失败');
      }
    })
    .catch(err => console.error('Register error:', err));
  };
  
  // 退出登录
  logoutBtn.onclick = () => {
    localStorage.removeItem('token');
    localStorage.removeItem('username');
    currentUser.token = null;
    currentUser.username = null;
    updateUIForLogout();
    fetchPosts();  // 刷新列表，去除编辑/删除按钮
  };
  
  // 发布新文章
  publishBtn.onclick = () => {
    const title = postTitle.value;
    const content = postContent.value;
    if (!title || !content) {
      alert('请填写标题和内容');
      return;
    }
    fetch(`${API_BASE}/posts`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + currentUser.token
      },
      body: JSON.stringify({ title, content })
    })
    .then(res => res.json().then(data => ({ status: res.status, body: data })))
    .then(result => {
      if (result.status === 201) {
        alert('文章发布成功');
        postTitle.value = '';
        postContent.value = '';
        fetchPosts();
      } else {
        alert(result.body.message || '发布失败');
      }
    })
    .catch(err => console.error('Publish error:', err));
  };
  
  // 更新UI为登录状态
  function updateUIForLogin() {
    loginSection.style.display = 'none';
    registerSection.style.display = 'none';
    postSection.style.display = 'block';
    welcomeMsg.textContent = `欢迎, ${currentUser.username}!`;
  }
  // 更新UI为登出状态
  function updateUIForLogout() {
    loginSection.style.display = 'block';
    registerSection.style.display = 'none';
    postSection.style.display = 'none';
    welcomeMsg.textContent = '';
  }
  
  // 获取文章列表
  function fetchPosts() {
    fetch(`${API_BASE}/posts`)
      .then(res => res.json())
      .then(data => {
        renderPosts(data);
      })
      .catch(err => console.error('Fetch posts error:', err));
  }
  
  // 渲染文章列表
  function renderPosts(posts) {
    postsList.innerHTML = '';
    if (!posts || posts.length === 0) {
      postsList.innerHTML = '<p>没有文章。</p>';
      return;
    }
    for (let post of posts) {
      const postDiv = document.createElement('div');
      postDiv.className = 'post';
      const titleEl = document.createElement('h3');
      titleEl.textContent = post.title;
      postDiv.appendChild(titleEl);
      const metaEl = document.createElement('div');
      metaEl.className = 'meta';
      metaEl.textContent = `作者: ${post.author} | 发布于: ${post.created_at}`;
      postDiv.appendChild(metaEl);
      const contentEl = document.createElement('p');
      contentEl.textContent = post.content;
      postDiv.appendChild(contentEl);
      // 如果当前用户是文章作者，显示编辑/删除按钮
      if (currentUser.token && currentUser.username === post.author) {
        const editBtn = document.createElement('button');
        editBtn.textContent = '编辑';
        editBtn.onclick = () => { editPost(post.id, post.title, post.content); };
        const deleteBtn = document.createElement('button');
        deleteBtn.textContent = '删除';
        deleteBtn.onclick = () => { deletePost(post.id); };
        postDiv.appendChild(editBtn);
        postDiv.appendChild(deleteBtn);
      }
      postsList.appendChild(postDiv);
    }
  }
  
  // 编辑文章（弹出对话框输入新内容）
  function editPost(postId, currentTitle, currentContent) {
    const newTitle = prompt('编辑标题:', currentTitle);
    if (newTitle === null) return;  // 用户取消编辑
    const newContent = prompt('编辑内容:', currentContent);
    if (newContent === null) return;
    fetch(`${API_BASE}/posts/${postId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + currentUser.token
      },
      body: JSON.stringify({ title: newTitle, content: newContent })
    })
    .then(res => res.json().then(data => ({ status: res.status, body: data })))
    .then(result => {
      if (result.status === 200) {
        fetchPosts();
      } else {
        alert(result.body.message || '更新失败');
      }
    })
    .catch(err => console.error('Edit post error:', err));
  }
  
  // 删除文章
  function deletePost(postId) {
    if (!confirm('确定删除这篇文章吗？')) return;
    fetch(`${API_BASE}/posts/${postId}`, {
      method: 'DELETE',
      headers: { 'Authorization': 'Bearer ' + currentUser.token }
    })
    .then(res => res.json().then(data => ({ status: res.status, body: data })))
    .then(result => {
      if (result.status === 200) {
        fetchPosts();
      } else {
        alert(result.body.message || '删除失败');
      }
    })
    .catch(err => console.error('Delete post error:', err));
  }
  
  // 页面加载完成后，获取文章列表
  fetchPosts();
</script>
</body>
</html>
```

这份前端代码实现了以下功能：

- **登录/注册界面切换：** 初始显示登录表单。点击“没有账号？注册”按钮，隐藏登录表单、显示注册表单；注册表单中点击“取消”则切回登录表单。
- **用户登录：** 点击登录按钮后，通过 `fetch` 向后端发送 `/api/login` 请求（POST，发送 JSON 数据）。收到响应后，如果状态码为200则表示登录成功，将返回的 `token` 和 `username` 保存到 `localStorage` 以及内存变量，并调用 `updateUIForLogin()` 切换前端为登录后的界面（显示发帖区和欢迎信息）。同时调用 `fetchPosts()` 刷新文章列表（这样可以显示出该用户自己文章的编辑/删除按钮）。若登录失败（401），则弹出提示。
- **用户注册：** 点击注册按钮后，发送 `/api/register` 请求。若返回201则注册成功，提示用户并自动切回登录界面（同时将注册的用户名填入登录表单方便登录）。若失败则提示错误（如用户名已存在）。
- **发布文章：** 登录成功后，会显示“发表新文章”表单。用户输入标题和内容，点击“发布”即发送 POST `/api/posts` 请求，携带 Authorization 头字段（`Bearer token`）。后端验证通过则返回201，前端提示发布成功，并清空输入框，刷新文章列表以显示新文章。
- **显示文章列表：** 页面加载时和每次操作后都会调用 `fetchPosts()` 获取最新文章列表。`renderPosts(posts)` 函数将文章数组渲染成HTML：每篇文章一个块，包含标题、作者、时间、内容。如果当前登录用户是该文章作者，则在文章后附加“编辑”和“删除”按钮，分别绑定点击事件。  
  - 编辑按钮点击时，调用 `editPost(post.id, ...)`，该函数使用 `prompt` 弹窗让用户修改标题和内容，然后发送 PUT `/api/posts/<id>` 请求更新文章。更新成功后重新加载列表。这里为了简化，没有单独做一个编辑表单，而是用浏览器的对话框输入；在实际应用中，可以通过打开编辑页面或在当前页面以表单形式编辑。  
  - 删除按钮点击时，先弹出 `confirm` 确认框，确认后调用 `deletePost(post.id)` 发送 DELETE 请求删除文章，成功则刷新列表。
- **保持登录状态：** 页面刷新或重新打开时，如果浏览器 `localStorage` 中保存有先前登录的 Token，则前端自动认为是已登录状态（当然前提是 Token 还有效），直接调用 `updateUIForLogin()` 显示发帖区域，并在加载文章列表后显示相应的编辑/删除按钮。这让用户免于每次刷新都要重新登录。

前端完全采用 AJAX 与后端通信，整个过程并没有页面跳转或重新加载，实现了单页应用的体验。由于我们的 Flask 后端设置了 `static_url_path=''` 并定义了根路由返回 `index.html`，所以只要 Flask 服务启动后，在浏览器访问 **`http://127.0.0.1:5000/`** 就能看到这个界面。在确保后端各接口正常的前提下，您可以：

- 注册一个新用户，然后登录；
- 发布文章，看文章列表实时更新；
- 尝试编辑或删除自己发布的文章；登录其他用户发布一些文章，验证不同用户无法编辑彼此的文章；
- 点击退出登录，确保按钮消失且无法再进行需要认证的操作。

通过这个简易的前端，我们验证了后端 API 的工作流程，也完成了一个可交互的博客最小原型。

> **注意：** 这个前端示例主要用于学习和调试接口，并未关注样式和完整性，例如没有分页功能、没有富文本编辑等。在真实项目中，可以考虑使用更健壮的前端框架（如 React、Vue）来构建界面。本示例旨在降低理解难度，用最简单的方式把后端功能跑通。

## 九、部署与运行建议

在开发完成并测试确认功能后，就可以考虑将这个 Flask 应用部署到生产环境。这里提供一些部署方面的建议：

- **使用生产级 WSGI 服务器：** Flask 自带的开发服务器方便调试，但**不适合直接用于生产环境** ([python - Is the server bundled with Flask safe to use in production? - Stack Overflow](https://stackoverflow.com/questions/12269537/is-the-server-bundled-with-flask-safe-to-use-in-production#:~:text=,at%20the%20Application%20Deployment%20pages))。它在高并发场景下性能不佳，而且开启调试模式可能暴露安全隐患。部署时应选择成熟的 WSGI 服务器（如 **Gunicorn** 或 **uWSGI**）来运行应用 ([python - Is the server bundled with Flask safe to use in production? - Stack Overflow](https://stackoverflow.com/questions/12269537/is-the-server-bundled-with-flask-safe-to-use-in-production#:~:text=The%20recommended%20approach%20is%20to,in%20the%20docs%3A%20Deployment%20Options))。例如，可以使用 Gunicorn 命令：  
  ```bash
  gunicorn -w 4 -b 0.0.0.0:5000 run:app
  ```  
  这将启动4个工作进程，监听5000端口提供服务。这样做可以充分利用多核CPU，提高并发处理能力。
- **前后端部署：** 由于我们的前端是纯静态文件，可以托管在任意静态文件服务器上。如果前后端部署在不同域名/端口下，一定记得在 Flask 中正确设置 CORS（本项目已启用）。如果不想单独部署前端，可以直接让 Flask 提供静态页面（本项目的做法），这种情况下前后端同源，就不涉及跨域问题。
- **反向代理：** 生产环境通常会使用 Nginx 等作为反向代理，将请求转发给后端的 Flask 应用服务。同时可以利用代理服务器处理静态文件、启用HTTPS、多域名路由等功能，从而减轻后端负担并提供更安全的访问。
- **配置管理：** 切勿在生产中使用开发配置。例如，确保关闭 Debug 模式；将 Secret Key 设置为环境变量（不要硬编码在代码中）；根据环境使用不同的数据库（开发用 SQLite，生产可换成更高性能的 MySQL/PostgreSQL 等，Flask-SQLAlchemy 切换数据库只需更改连接URI）。
- **日志和监控：** 在生产环境中，建议将应用日志记录到文件或日志管理系统，以便在出现错误时可以追踪分析。Flask 可与诸如 Sentry 的监控工具集成，在出现未捕获异常时发送告警。
- **依赖冻结：** 使用 `pip freeze > requirements.txt` 将依赖保存，部署时保证安装相同版本的库，避免版本差异造成的问题。
- **安全考虑：** 本项目演示目的，没有深入考虑一些安全细节。例如，可以在登录接口加入**请求频率限制**防止暴力破解密码，或在关键操作（删改）时要求额外确认。部署时也应确保服务器本身的安全（防火墙、HTTPS等）。

部署步骤概要：购买或租用一台服务器，安装 Python 环境，部署代码，使用 Gunicorn 启动 Flask，配置 Nginx 反代，并将服务设置为后台常驻（可用 systemd）。由于部署不是本文重点，这里不展开赘述。

> **参考：** Flask 官方文档提供了关于部署的详细指导和不同方案的比较 ([python - Is the server bundled with Flask safe to use in production? - Stack Overflow](https://stackoverflow.com/questions/12269537/is-the-server-bundled-with-flask-safe-to-use-in-production#:~:text=,at%20the%20Application%20Deployment%20pages)) ([python - Is the server bundled with Flask safe to use in production? - Stack Overflow](https://stackoverflow.com/questions/12269537/is-the-server-bundled-with-flask-safe-to-use-in-production#:~:text=Deploying%20your%20application%20is%20as,instead%20of%20Flask%27s%20development%20server))。简单来说，一定要让 Flask 跑在一个专业的WSGI容器里，并配合可靠的前置服务器，从而保证应用在生产环境中的性能和安全。

## 十、总结与下一步建议

通过本教程，我们从零开始构建了一个功能完整的 Flask 应用，实现了**用户认证**、**文章CRUD**、**RESTful API设计**以及**前后端分离**等关键知识点。整个过程中，我们涵盖了：

- Flask 项目结构的搭建，理解蓝图和工厂函数的作用；
- 路由的定义和视图函数的编写，如何处理请求数据和构造响应；
- Jinja 模板简单提及（本项目主要返回 JSON，前端渲染为主）； 
- 使用 Flask-SQLAlchemy 进行数据库操作以及模型关系配置；
- 基于 JWT 的用户认证机制，实现登录状态的保持和权限验证；
- 解决前后端分离中的跨域问题；
- 统一的错误处理和日志记录，为调试和维护提供支持；
- 简单前端页面的编写，使用 Fetch API 调用 Flask 提供的接口，实现前后端联调；
- 应用部署的基本要点。

完成本项目后，读者已经掌握了 Flask 开发 Web 服务的核心流程。接下来可以考虑进一步改进和扩展这个项目，例如：

- **丰富博客功能：** 添加文章分类、标签、评论功能等；引入富文本编辑器让文章内容支持格式；实现文章分页加载等。
- **完善用户系统：** 实现用户头像、个人资料页面；找回密码功能；权限角色区分（例如管理员可以管理所有文章）。
- **前端升级：** 使用主流前端框架重写前端，提高交互体验；或者使用服务器渲染模板（Flask 的 Jinja2）生成页面来练习传统的多页应用开发。
- **提高安全性：** 使用 Flask-Login 等扩展管理用户会话，结合 SSL 部署保障 Token 安全；对于敏感操作加入CSRF防护等（若采用表单提交）。
- **API优化：** 引入 Flask-RESTful 或 FastAPI 等框架改进 API 实现，自动生成接口文档；使用 Marshmallow 做序列化和校验，使代码更简洁。
- **性能和部署：** 在数据库上进行优化（如建立索引）；将应用部署在云服务器或容器中实战演练线上发布。

希望本教程帮助您入门 Flask 全栈开发。通过实践一个完整项目，您应当对 Flask 的工作方式和 Web 开发流程有了直观认识。祝您在后续的学习和开发中不断进步，打造出更多优秀的 Python Web 应用！

参考资料：
- Pallets 项目官方文档 (Flask/Python)  
- 《Flask Web开发：基于Python的Web应用开发实战》 (Miguel Grinberg)  
- 阮一峰的网络日志：CORS教程、JWT教程等  
- Frost Ming 博客：Flask 应用最佳实践经验 ([使用Flask搭建个人博客 - Frost's Blog](https://frostming.com/2019/08-11/flask-blog/#:~:text=%E4%BD%BF%E7%94%A8%20Flask%20%E6%9D%A5%E5%86%99%E5%8D%9A%E5%AE%A2%EF%BC%8C%E9%A6%96%E5%85%88%E8%A6%81%E8%80%83%E8%99%91%E7%9A%84%E6%98%AF%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E5%AE%83%E4%B8%8D%E5%83%8F%20Django%20%E4%B8%80%E6%A0%B7%EF%BC%8C%E6%9C%89%E5%9B%BA%E5%AE%9A%E7%9A%84%E6%8E%A8%E8%8D%90%E7%BB%93%E6%9E%84%EF%BC%8C%E8%80%8C%E6%98%AF%E7%BB%99%E4%BA%86%E7%94%A8%E6%88%B7%E5%BE%88%E5%A4%A7%E7%9A%84%E8%87%AA%E7%94%B1%E7%A9%BA%E9%97%B4%E6%9D%A5%E7%BB%84%E7%BB%87%E9%A1%B9%E7%9B%AE%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E6%80%BB%E7%9A%84%E6%9D%A5%E8%AF%B4%EF%BC%8C%E6%9C%89%E4%B8%A4%E5%A4%A7%E6%B5%81%E6%B4%BE%EF%BC%9A))  

