---
title: Django 的快速上手
date: 2020/4/30
description: 快速搭建你的Django
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/x1SbItTOUXs9Lqk.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/tWpPXKnCBcsfY76.png'
categories:
  - python
tags:
  - python
  - Django
abbrlink: 49637
---

# Django 的快速上手

![下载](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/uDcJE8I5QofzZ2t.jpg)

## 一、Django 概述

Python的Web框架有上百个，比它的关键字还要多。所谓Web框架，就是用于开发Web服务器端应用的基础设施，说得通俗一点就是一系列封装好的模块和工具。事实上，即便没有Web框架，我们仍然可以通过socket或CGI来开发Web服务器端应用，但是这样做的成本和代价在商业项目中通常是不能接受的。通过Web框架，我们可以化繁为简，降低创建、更新、扩展应用程序的工作量。刚才我们说到Python有上百个Web框架，这些框架包括Django、Flask、Tornado、Sanic、Pyramid、Bottle、Web2py、web.py等。

在上述Python的Web框架中，Django无疑是最有代表性的重量级选手，开发者可以基于Django快速的开发可靠的Web应用程序，因为它减少了Web开发中不必要的开销，对常用的设计和开发模式进行了封装，并对MVC架构提供了支持（Django中称之为MTV架构）。许多成功的网站和应用都是基于Django框架构建的，国内比较有代表性的网站包括：知乎、豆瓣网、果壳网、搜狐闪电邮箱、101围棋网、海报时尚网、背书吧、堆糖、手机搜狐网、咕咚、爱福窝、果库等。

![mvc](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/bEuM9DPR6yFmOjJ.png)

Django诞生于2003年，它是一个在真正的应用中成长起来的项目，由劳伦斯出版集团旗下在线新闻网站的内容管理系统（CMS）研发团队编写（主要是Adrian Holovaty和Simon Willison），以比利时的吉普赛爵士吉他手Django Reinhardt来命名，在2005年夏天作为开源框架发布。使用Django能用很短的时间构建出功能完备的网站，因为它代替程序员完成了所有乏味和重复的劳动，剩下真正有意义的核心业务给程序员，这一点就是对DRY（Don't Repeat Yourself）理念的最好践行。

## 二、环境准备

### 1. 搭建虚拟环境

随着我们项目的积累，有时候不同项目需要用不到不同版本的包，可能会产生冲突，这时候我们需要一个虚拟环境将每个项目需要的包进行独立，这样就能有效避免冲突。

### 2. 安装数据库

Django支持很多中类型的数据库，默认配置的sqlite3。在这里我们使用MySQL

### 3. 安装工具

- Django2.0和以后的版本不再支持Python2.X，所以我们需要安装Python3.6版本的解释器。

  如图是 Django 和 Python 版本对照表

  | Django 版本 | Python 版本              |
  | :---------- | :----------------------- |
  | 1.8         | 2.7, 3.2 , 3.3, 3.4, 3.5 |
  | 1.9, 1.10   | 2.7, 3.4, 3.5            |
  | 1.11        | 2.7, 3.4, 3.5, 3.6       |
  | 2.0         | 3.4, 3.5, 3.6, 3.7       |
  | 2.1, 2.2    | 3.5, 3.6, 3.7            |

- pip是一个通用的Python包管理工具，可以对包进行查找、安装、卸载

- PyCharm是一种Python IDE，比较时候较大的Python程序，像平时单纯的写一个小工具爬虫脚本什么的也可以使用subline。

- 使用 pip 安装 Django (或者也可以依赖pycharm的内置工具下载)

  ~~~shell
  pip install django
  ~~~

### 4. 用 Pycharm 快速搭建

![image-20200430180231376](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/KAnVf4B6Ye7c1Jd.png)

注意：这里最好是新建一个单独虚拟环境，防止多个项目使用的版本混淆

这里直接可以用 Pycharm 快速搭建，也可以用命令行和 Django 提供的搭建工具

~~~shell
django-admin.py startproject 你的项目名
~~~

## 三、项目目录结构

![image-20200430181710701](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/RrOqm8PB6cfo3wz.png)

### 目录说明

- `__init__.py`：声明这是一个 python 包
- `asgi.py`：一个默认的ASGI配置，用于布置异步服务器
- `settings.py`：管理项目的配置信息
- `urls.py`：声明请求url的映射关系
- `wsgi.py`：python程序和web服务器的通信协议
- `manage.py`： 一个命令行工具，用来和Django项目进行交互
- templates：前端页面和一些静态文件
- venv：虚拟环境的配置文件

###  **项目配置文件-------setting.py**

```python
import os
# 项目的相对路径，启动服务的时候会运行这个文件所在路径的manage.py
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
# 安全密钥
SECRET_KEY = 'l&!v_npes(!j82+x(44vt+h&#ag7io2x&shnf*9^8fv0d63!0r'
# 是否开启Debug
DEBUG = True
# 允许访问的主机ip，可以用通配符*
ALLOWED_HOSTS = []
# Application definition
# 用来注册App 前6个是django自带的应用
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
# 中间件 ,需要加载的中间件。比如在请求前和响应后根据规则去执行某些代码的方法
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
# 指定URL列表文件 父级URL配置
ROOT_URLCONF = 'djangoDemo.urls'
# 加载网页模板路径
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
# WSGI的配置文件路径
WSGI_APPLICATION = 'djangoDemo.wsgi.application'
# 数据库配置 默认的数据库为sqlite
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
# 相关密码验证
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
# 语言设置 默认英语， 中文是zh-hans
LANGUAGE_CODE = 'en-us'
# 时区设置，中国的是：Asia/Shanghai
TIME_ZONE = 'UTC'
# i18n字符集是否支持
USE_I18N = True
USE_L10N = True
# 是否使用timezone
# 保证存储到数据库中的是 UTC 时间；
# 在函数之间传递时间参数时，确保时间已经转换成 UTC 时间；
USE_TZ = True
# 静态文件路径
STATIC_URL = '/static/'
```

## 四、程序启动

### 两种方式启动

- 命令行

  ~~~shell
  (venv)$ python manage.py runserver
  ~~~

- Pycharm

  注意这里不是直接运行`manage.py`，运行之后并没有什么反应。这里使用Pycharm内置的工具

  ![image-20200430183112277](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/FoOsDbH5gXKAw1y.png)

然后下面会弹出一个类似的命令行窗口，输入 runserve 就能启动项目了。还有很贴心的提示功能！

### 访问

项目启动后可以到 http://127.0.0.1:8000 或者 http://localhost:8000 ，就可以访问到我们的服务器。将配置文件的语言和使其进行修改，就有了如下的效果：

![image-20200430183859194](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Aunm1CkzWaEZLQg.png)

几点说明：

- 刚刚启动的是Django自带的用于开发和测试的服务器，它是一个用纯Python编写的轻量级Web服务器，但它并不是真正意义上的生产级别的服务器，千万不要将这个服务器用于和生产环境相关的任何地方。
- 用于开发的服务器在需要的情况下会对每一次的访问请求重新载入一遍Python代码。所以你不需要为了让修改的代码生效而频繁的重新启动服务器。然而，一些动作，比如添加新文件，将不会触发自动重新加载，这时你得自己手动重启服务器。
- 可以通过`python manage.py help`命令查看可用命令列表
- 在启动服务器时，也可以通过`python manage.py runserver 1.2.3.4:5678`来指定将服务器运行于哪个IP地址和端口。



## 五、APP的概念

在Django中，所有的模块都被定义为一个APP，我们需要创建并将其注册到主文件中才能生效。

在下面的命令行中输入

~~~shell
startapp demo
~~~

会自动帮你创建一个新的目录

![image-20200430184554978](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/sqxBeHMlm1RA3nI.png)

- `admin.py`：对应应用后台管理配置文件
- `apps.py`：对应应用的配置文件
- `models.py`：数据模块，用于设计数据库等
- `tests.py`：编写测试代码
- `views.py`：视图层，直接和浏览器进行交互

每次新建一个App我们需要将其在**settings.py**文件中的 INSTALLED_APPS 里进行注册，这样程序才能够找到这个服务

~~~python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'demo', # 注册新创建的应用
]
~~~



## 六、开始网页搭建

### 编写 hello world

1. 在demo文件夹的 views.py 中

   ~~~python
   from django.http import HttpResponse
   """
    django.http模块中定义了HttpResponse 对象的API
    作用：不需要调用模板直接返回数据
    HttpResponse属性：
       content: 返回内容,字符串类型
       charset: 响应的编码字符集
       status_code: HTTP响应的状态码
   hello 为一个视图函数，每个视图函数必须第一个参数为request。哪怕用不到request。
   request是django.http.HttpRequest的一个实例
   """
   def hello(request):
       return HttpResponse('<h1>Hello, Django!</h1>')
   ~~~

2. 修改项目的 urls.py 文件并映射URL

   ~~~python
   from django.conf.urls import url
   from django.contrib import admin
   from django.urls import path
   
   from demo import views
   
   urlpatterns = [
       path('admin/', admin.site.urls),
       url(r'^hello/$', views.hello)
       # 其中'^'和'$'为严格匹配
   ]
   
   ~~~

   注意：如果配成下面的样子，那么只要你的url中含 'hello' 字段就会指向我们的 `views.hello`

   ~~~python
   url('hello/', views.hello)
   ~~~

   

3. 然后启动项目访问http://127.0.0.1:8000/hello/

   ![image-20200430190435560](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/eWKBZj5oCwDFG7H.png)

### 映射静态文件

很明显在实际项目中我们都是通过前端页面文件来进行访问的，下面就来配置一下前端页面映射

我们可以用Django框架中template模块的Template类创建模板对象，通过模板对象的render方法实现对模板的渲染。在Django框架中还有一个名为`render`的便捷函数可以来完成渲染模板的操作。

所谓的渲染就是用数据替换掉模板页中的占位符，当然这里的渲染称为后端渲染，即在服务器端完成页面的渲染再输出到浏览器中。

1. 在template文件夹中创建`hello.html`

   ~~~html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Hello</title>
   </head>
   <body>
       <h1>{{ message }}</h1>
   </body>
   </html>
   ~~~

   在上面的模板页中我们使用了`{{ greeting }}`这样的模板占位符语法。这些都是Django模板语言（DTL）的一部分。

2. 回到demo目录修改 `views.py`

   ~~~python
   from django.shortcuts import render
   
   def hello(request):
       return render(request, 'hello.html', {'message': "Hello, Django!"})
   ~~~

3. 运行服务器打开程序就能看到和刚才一样的效果了