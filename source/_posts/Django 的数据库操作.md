---
title: Django 的数据库操作
date: 2020/5/1
description: Django 的数据库操作
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/rt4ubo7Phv38CmZ.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/rt4ubo7Phv38CmZ.jpg'
categories:
  - python
tags:
  - python
  - Django
abbrlink: 55582
---



# Django 的数据库操作

有之前的基础，那么我们就可以开始对数据库进行操作。

## 一、数据库配置

### 配置  MySql

在**主目录**的 `settings.py`  中修改

~~~python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',  # 数据库驱动
        'NAME': '', # 被操作数据库名称
        'HOST': '127.0.0.1', # 数据库位置
        'PORT': 3306, # 端口号
        'USER': '', # 用户名
        'PASSWORD': '',	# 密码
    }
}
~~~

> 数据库驱动
>
> - `'django.db.backends.sqlite3'`：SQLite嵌入式数据库。
> - `'django.db.backends.postgresql'`：BSD许可证下发行的开源关系型数据库产品。
> - `'django.db.backends.mysql'`：转手多次目前属于甲骨文公司的经济高效的数据库产品。
> - `'django.db.backends.oracle'`：甲骨文公司的关系型数据库旗舰产品。

如果你没有安装数据库依赖还需要：

~~~bash
(venv)$ pip install pymysql
~~~

然后在**项目目录**的`__init__.py`：中对数据库驱动初始化。
从而避免Django找不到连接MySQL的客户端工具而询问你：“Did you install mysqlclient? ”

~~~python
import pymysql

pymysql.install_as_MySQLdb()
~~~

### 创建模型构建数据表

- 创建一个app应用

  ~~~bash
  (venv)$ python manage.py startapp user
  ~~~

- 将应用注册到项目中去

  ~~~python
  # 在项目目录中的 settings.py
  INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      'user',
  ]
  ~~~

- 注册路由跳转

  ~~~python
  # 项目目录中的 urls.py
  from django.contrib import admin
  from django.urls import path, include
  
  urlpatterns = [
      path('admin/', admin.site.urls),
      path('', include('user.urls')),
  ]
  ~~~

- 构建数据模型

  ~~~python
  # 应用目录的 models.py
  from django.db import models
  
  
  class User(models.Model):
      """用户类"""
  
      no = models.IntegerField(primary_key=True, db_column='no', verbose_name='编号')
      name = models.IntegerField(db_column='name', verbose_name='名字')
      gender = models.CharField(max_length=20, db_column='gender', verbose_name='性别')
      age = models.CharField(max_length=20, db_column='age', verbose_name='年龄')
      address = models.CharField(max_length=20, db_column='address', verbose_name='籍贯')
      qq = models.IntegerField(db_column='qq', verbose_name='QQ')
      email = models.CharField(max_length=20, db_column='email', verbose_name='邮箱')
  
      class Meta:
          db_table = 'user'
  
  
  class Admin(models.Model):
      """管理员"""
  
      no = models.IntegerField(primary_key=True, db_column='no', verbose_name='编号')
      username = models.CharField(max_length=20, db_column='username', verbose_name='用户名')
      password = models.CharField(max_length=20, db_column='password', verbose_name='密码')
  
      class Meta:
          db_table = 'manager'
  ~~~

- 生成数据表

  ~~~bash
  # 迁移 Django 内置的管理数据表
  (venv)$ python manage.py migrate
  # 生成我们的数据迁移文件
  (venv)$ python manage.py makemigrations user
  # 迁移我们的数据表
  (venv)$ python manage.py migrate
  ~~~

  ![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Ar2xiTqSnLYwENW.png)

  >  可以看出除了最后两张表，上面都是Django为我们默认配置的数据表

  

  

## 二、登录 Django 的后台管理系统

>Django框架有自带的后台管理系统来实现对模型的管理。虽然实际应用中，这个后台可能并不能满足我们的需求，但是在学习Django框架时，我们暂时可以利用Django自带的后台管理系统来管理我们的模型，同时也可以了解一个项目的后台管理系统到底需要哪些功能。

#### 注册一个超级管理员账号

  ~~~bash
  (venv)$ python manage.py createsuperuser
  Username :
  Email address:
  Password: 
  Password (again): 
  Superuser created successfully.
  ~~~

  

#### 然后我们启动 Django

  ~~~bash
  (venv)$ python manage.py runserver
  ~~~

  

#### 访问 http://127.0.0.1:8000/admin

  ![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/a7sgOn2oUDvt5Ld.png)

  

  

输入刚刚注册的账号密码就可以用 Django 提供的图形界面来操作我们的数据库了  ![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/k2S8ADFn4BKib1f.png)

  这里会记录我们对数据库的所有操作

  

## 三、模型定义参考

#### 字段

  对字段名称的限制

  - 字段名不能是Python的保留字，否则会导致语法错误
  - 字段名不能有多个连续下划线，否则影响ORM查询操作

  Django模型字段类

| 字段类                | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| AutoField             | 自增ID字段                                                   |
| BigIntegerField       | 64位有符号整数                                               |
| BinaryField           | 存储二进制数据的字段，对应Python的bytes类型                  |
| BooleanField          | 存储True或False                                              |
| CharField             | 长度较小的字符串                                             |
| DateField             | 存储日期，有auto_now和auto_now_add属性                       |
| DateTimeField         | 存储日期和日期，两个附加属性同上                             |
| DecimalField          | 存储固定精度小数，有max_digits（有效位数）和decimal_places（小数点后面）两个必要的参数 |
| DurationField         | 存储时间跨度                                                 |
| EmailField            | 与CharField相同，可以用EmailValidator验证                    |
| FileField             | 文件上传字段                                                 |
| FloatField            | 存储浮点数                                                   |
| ImageField            | 其他同FileFiled，要验证上传的是不是有效图像                  |
| IntegerField          | 存储32位有符号整数。                                         |
| GenericIPAddressField | 存储IPv4或IPv6地址                                           |
| NullBooleanField      | 存储True、False或null值                                      |
| PositiveIntegerField  | 存储无符号整数（只能存储正数）                               |
| SlugField             | 存储slug（简短标注）                                         |
| SmallIntegerField     | 存储16位有符号整数                                           |
| TextField             | 存储数据量较大的文本                                         |
| TimeField             | 存储时间                                                     |
| URLField              | 存储URL的CharField                                           |
| UUIDField             | 存储全局唯一标识符                                           |

#### 字段属性

  通用字段属性

| 选项           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| null           | 数据库中对应的字段是否允许为NULL，默认为False                |
| blank          | 后台模型管理验证数据时，是否允许为NULL，默认为False          |
| choices        | 设定字段的选项，各元组中的第一个值是设置在模型上的值，第二值是人类可读的值 |
| db_column      | 字段对应到数据库表中的列名，未指定时直接使用字段的名称       |
| db_index       | 设置为True时将在该字段创建索引                               |
| db_tablespace  | 为有索引的字段设置使用的表空间，默认为DEFAULT_INDEX_TABLESPACE |
| default        | 字段的默认值                                                 |
| editable       | 字段在后台模型管理或ModelForm中是否显示，默认为True          |
| error_messages | 设定字段抛出异常时的默认消息的字典，其中的键包括null、blank、invalid、invalid_choice、unique和unique_for_date |
| help_text      | 表单小组件旁边显示的额外的帮助文本。                         |
| primary_key    | 将字段指定为模型的主键，未指定时会自动添加AutoField用于主键，只读。 |
| unique         | 设置为True时，表中字段的值必须是唯一的                       |
| verbose_name   | 字段在后台模型管理显示的名称，未指定时使用字段的名称         |

  ForeignKey属性

  1. limit_choices_to：值是一个Q对象或返回一个Q对象，用于限制后台显示哪些对象。
  2. related_name：用于获取关联对象的关联管理器对象（反向查询），如果不允许反向，该属性应该被设置为`'+'`，或者以`'+'`结尾。
  3. to_field：指定关联的字段，默认关联对象的主键字段。
  4. db_constraint：是否为外键创建约束，默认值为True。
  5. on_delete：外键关联的对象被删除时对应的动作，可取的值包括django.db.models中定义的：
     - CASCADE：级联删除。
     - PROTECT：抛出ProtectedError异常，阻止删除引用的对象。
     - SET_NULL：把外键设置为null，当null属性被设置为True时才能这么做。
     - SET_DEFAULT：把外键设置为默认值，提供了默认值才能这么做。

  ManyToManyField属性

  1. symmetrical：是否建立对称的多对多关系。
  2. through：指定维持多对多关系的中间表的Django模型。
  3. throughfields：定义了中间模型时可以指定建立多对多关系的字段。
  4. db_table：指定维持多对多关系的中间表的表名。

  

#### 模型元数据选项

| 选项                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| abstract              | 设置为True时模型是抽象父类                                   |
| app_label             | 如果定义模型的应用不在INSTALLED_APPS中可以用该属性指定       |
| db_table              | 模型使用的数据表名称                                         |
| db_tablespace         | 模型使用的数据表空间                                         |
| default_related_name  | 关联对象回指这个模型时默认使用的名称，默认为<model_name>_set |
| get_latest_by         | 模型中可排序字段的名称。                                     |
| managed               | 设置为True时，Django在迁移中创建数据表并在执行flush管理命令时把表移除 |
| order_with_respect_to | 标记对象为可排序的                                           |
| ordering              | 对象的默认排序                                               |
| permissions           | 创建对象时写入权限表的额外权限                               |
| default_permissions   | 默认为`('add', 'change', 'delete')`                          |
| unique_together       | 设定组合在一起时必须独一无二的字段名                         |
| index_together        | 设定一起建立索引的多个字段名                                 |
| verbose_name          | 为对象设定人类可读的名称                                     |
| verbose_name_plural   | 设定对象的复数名称                                           |

  

### 查询参考

  按字段查找可以用的条件：

  1. exact / iexact：精确匹配/忽略大小写的精确匹配查询
  2. contains / icontains / startswith / istartswith / endswith / iendswith：基于`like`的模糊查询
  3. in：集合运算
  4. gt / gte / lt / lte：大于/大于等于/小于/小于等于关系运算
  5. range：指定范围查询（SQL中的`between…and…`）
  6. year / month / day / week_day / hour / minute / second：查询时间日期
  7. isnull：查询空值（True）或非空值（False）
  8. search：基于全文索引的全文检索
  9. regex / iregex：基于正则表达式的模糊匹配查询

  

  

## 四、静态页面映射

  我们将我们的实现准备好的 html 文件反正根目录的 **templates** 文件夹里面，再创建一个 **static** 文件夹来存放我们的 js、css、font之类的资源文件。最后配置我们的资源映射

  ~~~bash
  # 在项目目录的 settings.py
  STATIC_URL = '/static/'
  STATICFILES_DIRS = [
      os.path.join(BASE_DIR, 'static'),
  ]
  ~~~

  在 html 中引入资源文件

  ~~~html
  <!-- 在最前面加载静态文件 -->
  {% load static %}
  
  <!-- 在html的head标签中 -->
      <!-- 1. 导入CSS的全局样式 -->
      <link href="{% static "css/bootstrap.min.css" %}" rel="stylesheet">
      <!-- 2. jQuery导入 -->
      <script src="{% static "js/jquery-2.1.0.min.js" %}"></script>
      <!-- 3. 导入bootstrap的js文件 -->
      <script src="{% static "js/bootstrap.min.js" %}"></script>
  ~~~

## 五、基本增删改查语句

  ~~~python
  from user import models
  
  def add(request):
      # 第一种方法：
      models.Admin.objects.create(username='userme', password='1234')
      
      # 第二种方法：添加数据，实例化表类，在实例化里传参为字段和值
      obj = models.Admin(username='userme', password='1234')
      # 写入数据库
      obj.save()
      
      # 第三种方法：将要写入的数据组合成字典，键为字段值为数据
      dic = {'username':'userme', 'password':'1234'}
      # 添加到数据库，注意字典变量名称一定要加**
      models.Admin.objects.create(**dic)
      
  def delete(request):
      # 删除数据
      models.Admin.objects.filter(username='userme').delete()
      
  def update(request):
      # 修改数据
      models.Admin.objects.filter(username='userme').update(username='admin')
      
  
  ~~~

下面是 Django框架的一些基本查询 API 整理

```yml
all(): 查询所有结果
filter(**kwargs): 它包含了与所给筛选条件相匹配的对象
get(**kwargs): 返回与所给筛选条件相匹配的对象，返回结果有且只有一个，如果符合筛选条件的对象超过一个或者没有都会抛出错误。
exclude(**kwargs): 它包含了与所给筛选条件不匹配的对象
values(*field): 返回一个ValueQuerySet 一个特殊的QuerySet，运行后得到的并不是一系列model的实例化对象，而是一个可迭代的字典序列
values_list(*field): 它与values()非常相似，它返回的是一个元组序列，values返回的是一个字典序列
order_by(*field): 对查询结果排序
reverse(): 对查询结果反向排序
distinct(): 从返回结果中剔除重复纪录
count(): 返回数据库中匹配查询(QuerySet)的对象数量。
first(): 返回第一条记录
last(): 返回最后一条记录
exists(): 如果QuerySet包含数据，就返回True，否则返回False
annotate(): 使用聚合函数
dates(): 根据日期获取查询集
datetimes(): 根据时间获取查询集
none(): 创建空的查询集
union(): 并集
intersection(): 交集
difference(): 差集
select_related(): 附带查询关联对象
prefetch_related(): 预先查询
extra(): 附加SQL查询
defer(): 不加载指定字段
only(): 只加载指定的字段
using(): 选择数据库
select_for_update(): 锁住选择的对象，直到事务结束。
raw(): 接收一个原始的SQL查询
```
