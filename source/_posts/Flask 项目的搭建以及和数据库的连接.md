---
title: Flask 项目的搭建以及和数据库的连接
date: 2020/11/27
description: Flask 项目的搭建以及和数据库的连接
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/VdizxyQl9RSAkJg.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/VdizxyQl9RSAkJg.png'
categories:
  - python
tags:
  - python
  - Flask
abbrlink: 27824
---

# Flask 项目的搭建以及和数据库的连接

## 一、基于Python的后端框架

这里就只列举 **Django**、**Flask** 两个web框架

### 1. Django

- Django是一个开源的Python Web应用框架，采用了**MVT的框架模式，即模型M，视图V和模版T**，最早于2005年发布。
- Django被认为是"大而全"的重量级Web框架，其自带大量的常用工具和组件（比如数据库ORM组件、用户认证、权限管理、分页、缓存), 甚至还自带了管理后台Admin，适合快速开发功能完善的企业级网站。
- Django自带免费的数据SQLite，同时支持MySQL与PostgreSQL等多种数据库。
- Django项目的结构布局是刚性的，每个人写的项目结构最后都差不多，你可以清楚地知道在哪个APP的哪个文件夹里找到哪个文件(media目录, static目录, template目录，views.py, models.py, forms.py, etc)。

### 2. Flask

- Flask是一个由Python语言写成的轻量级Web框架，最早由奥地利人Armin Ronacher于2010年发布。
- Flask最显著的特点是它是一个“微”框架，轻便灵活，但同时又易于扩展。
- 默认情况下，Flask 只相当于一个内核，不包含数据库抽象层(ORM)、用户认证、表单验证、发送邮件等其它Web框架经常包含的功能。
- Flask依赖用各种灵活的扩展（比如邮件Flask-Mail，用户认证Flask-Login，数据库Flask-SQLAlchemy）来给Web应用添加额外功能。**Flask的这种按需扩展的灵活性是很多程序员喜欢它的地方**
- Flask没有指定的数据库，可以用MySQL，也可以用 NoSQL。
- 在项目结构上，Flask是很灵活的，你可以随意地组织自己的代码，1000个APP说不定就有有1000种组织代码的方式。

> 如果你只需要开发一个轻量级网站或者特定的微服务（比如API)，你根本用不上Django自带的大而全的组件和功能，这时你应该毫不犹豫地选择Flask。当你想尝试新的技术时，使用Flask也会是个更好的选择，轻便而灵活。如果你的开发项目目标明确，就是要开发包含各种功能的传统企业级网站（比如电商，新闻内容管理，社交网站，办公OA)，使用Django能帮你节省不少寻找或开发第三方扩展的精力。开发企业级网站通常由一个团队来进行，Django可插拔式的APP设计思想和刚性的项目结构便于团队后期维护项目代码。从个性上而言，如果你喜欢自由灵活，你就选Flask。

## 二、开始你的第一个 Flask

使用 pip 安装 flask

~~~bash
pip install flask
~~~

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/2xcZ6NQPXslqdvo.png" alt="image-20201202200814274" style="zoom: 33%;" />

~~~python
# app.py
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
~~~

运行程序之后，就默认在 5000 端口，访问 http://127.0.0.1:5000

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/sJ5yCWcr76f4VIj.png" alt="image-20201202202330844" style="zoom: 33%;" />

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/87RhVXkDZ46fzgB.png" style="zoom: 33%;" />

**项目启动参数：**

- debug：是否开启调试模式，默认为False；开启后会有出错调试信息，文件会自动加载。
- threaded：是否开启多线程，默认为Flase。
- host：指定主机，默认为’127.0.0.1’，设置为’0.0.0.0’后可以通过IP进制访问
- port：指定端口，默认为5000。

~~~python
app.run(debug=True, threaded=True, host=‘0.0.0.0’, port=5555)
~~~

## 三、使用蓝图 Blueprint	项目模块化

**随着flask程序越来越复杂,我们需要对程序进行模块化的处理**

简单来说，Blueprint 是一个存储视图方法的容器，这些操作在这个Blueprint 被注册到一个应用之后就可以被调用，Flask 可以通过Blueprint来组织URL以及处理请求。

Flask使用Blueprint让应用实现模块化，在Flask中，Blueprint具有如下属性：

- 一个项目可以具有多个Blueprint
- 可以将一个Blueprint注册到任何一个未使用的URL下比如 “/”、“/sample”或者子域名
- 在一个应用中，一个模块可以注册多次
- Blueprint可以单独具有自己的模板、静态文件或者其它的通用操作方法，它并不是必须要实现应用的视图和函数的
- 在一个应用初始化时，就应该要注册需要使用的Blueprint

~~~python
pip install Blueprint
~~~

### `controller.py`

~~~python
from flask import Blueprint

user_route = Blueprint('user_route', '__name__')


@user_route.route('/get', methods=['GET'])
def get():
    return 'ok'
~~~

### `app.py`

~~~python
from flask import Flask

from controller import user_route

app = Flask(__name__)

app.register_blueprint(user_route, url_prefix='/user')


if __name__ == '__main__':
    app.run()
~~~

现在就能在 http://127.0.0.1:5000/user/get 访问到这个接口了

## 四、接口参数和返回值

### 1. GET 请求参数

~~~python
from flask import Flask, request

@app.route('/', methods=['GET'])
def hello_world():
    request.get_data()
    return 'ok'
# The return type must be a string, dict, tuple, Response instance, or WSGI callable
~~~

### 2. POST 请求参数

~~~python
from flask import Flask, request

@app.route('/', methods=['POST'])
def hello_world():
    # 从form表单中获取参数
    request.form.get('name')
    request.form.get('password')
    return 'ok'
~~~

### 3. 路径请求参数

~~~python
@app.route('/<int:param> ', methods=['GET'])
def hello_world(param):
    return param
# 这里也可以不指定参数类型，只写<param>
~~~

## 五、Mysql 数据库连接

### 1. 安装依赖

~~~bash
pip install flask_sqlalchemy
pip install pymysql
~~~

### 2. 创建表

- `app.py`

  ~~~python
  from flask import Flask
  from flask_sqlalchemy import SQLAlchemy
  
  app = Flask(__name__)
  # 读取配置文件
  app.config.from_pyfile('config.py')
  # 初始化数据库连接
  db = SQLAlchemy(app)
  
  
  @app.route('/')
  def hello_world():
      return 'ok'
  
  
  if __name__ == '__main__':
      app.run()
  ~~~

- `config.py`

  ~~~python
  # 数据库配置
  USERNAME = 'root'
  PASSWORD = '123456'
  HOST = '127.0.0.1'
  PORT = '3306'
  DATABASE = 'raspberry'  # 数据库名
  SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://{}:{}@{}:{}/{}?charset=utf8'.format(
      USERNAME, PASSWORD, HOST, PORT, DATABASE)
  ~~~

- `models.py`

  ~~~python
  from app import db
  
  
  class User(db.Model):
      user_id = db.Column(db.Integer, autoincrement=True, primary_key=True)
      username = db.Column(db.String(20), nullable=False)
      password = db.Column(db.String(20), nullable=False)
  
      # 类似Java中重写toString方法
      def __repr__(self):
          info = {
              'user_id': self.user_id,
              'username': self.username,
              'password': self.password
          }
          return str(info)
  ~~~

- 在控制台中

  <img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/fzByh1NcdlXSRpv.png" alt="image-20201203103424014" style="zoom:33%;" />

  ~~~python
  import app
  app.db.create_all()
  ~~~

  此时flask就会自动帮我们在数据库中建立数据表

### 3. 增删改查

  **只要在项目中引入 `app.py` 中的 db 对象后，就可以通过 db.session 这一对象来执行对数据库的增删改查操作**

  - 增加数据

    ~~~python
    add_user = User(1, 'admin', '123456')
    db.session.add(add_user)
    db.session.commit()
    ~~~

  - 查询数据

    ~~~Python
    # 返回一个list
    db.session.query(User).all()
    # filter 条件查询
    db.session.query(User).filter(User.id >= 3).all()
    # 只取第一个
    db.session.query(User).filter(User.id >= 3).first()
    ~~~

  - 删除数据

    ~~~python
    # 需要先查询拿出对象
    db.session.delete(article)
    db.session.commit()
    ~~~

  - 修改数据

    ~~~python
    db.session.query(User).filter(User.id == 1).update({'name':'root'})
    db.session.commit()
    ~~~

## 六、MQTT 消息队列连接

### 1. 安装依赖

~~~python
pip install paho-mqtt
~~~

### 2. 配置参数  	`config.py`

~~~python
# mqtt配置
instanceId = 'XXXXXXXXXXXXX'
accessKey = 'XXXXXXXXXX'
secretKey = 'XXXXXXXXXXXXXXXXXXX'
groupId = 'XXXXXXXXXXXX'
client_id = groupId + '@@@' + 'XXX'
topic = 'XXXXXXXXXXXXXXXXXXX'
brokerUrl = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
targetId = 'XXXXXXXXXXXXXXXXXXXXXXXXXX'
username = 'Signature' + '|' + accessKey + '|' + instanceId
~~~

### 3. 连接服务器

- `mqtt.py`

  ~~~python
  import hmac
  import base64
  from hashlib import sha1
  from paho.mqtt import client as mqtt
  
  from config import client_id, secretKey, brokerUrl, topic, targetId, username
  
  
  def on_connect(client, userdata, flags, rc):
      print('成功连接到MQTT服务器，用户ID：' + client_id)
  
  
  def on_message(client, userdata, msg):
      print('收到了：' + msg.topic + ' ' + str(msg.payload))
  
  
  def on_disconnect(client, userdata, rc):
      if rc != 0:
          print('意外失去连接 %s' % rc)
  
  
  client = mqtt.Client(client_id, protocol=mqtt.MQTTv311, clean_session=True)
  client.on_connect = on_connect
  client.on_message = on_message
  client.on_disconnect = on_disconnect
  password = base64.b64encode(hmac.new(secretKey.encode(), client_id.encode(), sha1).digest()).decode()
  client.username_pw_set(username, password)
  client.connect(brokerUrl, 1883, 60)
  
  
  def send_msg(msg):
      client.publish(topic + '/p2p/' + targetId, msg, qos=2)
  ~~~

- `app.py`

  ~~~python
  from flask import Flask
  
  from mqtt import client, send_msg
  
  app = Flask(__name__)
  # 读取配置文件
  app.config.from_pyfile('config.py')
  # 连接MQTT
  client.loop_start()
  
  
  @app.route('/')
  def hello_world():
      send_msg('msg')
      return 'ok'
  
  
  if __name__ == '__main__':
      app.run()
  ~~~

此时启动项目就会自动连接上服务器，访问接口就可以给指定主题发送信息
