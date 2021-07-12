---
title: Django 的员工信息系统
date: 2020/5/3
description: Django 的数据库操作
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/WGjQk25ap7NhlUS.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/WGjQk25ap7NhlUS.jpg'
categories:
  - python
tags:
  - python
  - Django
abbrlink: 47578
---
# Django 的员工信息系统

将之前用 ssm 框架搭建的系统再次用 Django 的框架来实现

**码云地址在文末**

**功能实现：**

- 登录权限验证
- 登录状态检测
- 员工信息分页查询显示
- 多条件联合查询
- 增删改数据

## 一、登录验证

### 1. 创建管理员模型和用户模型

~~~python
# user\models.py
from django.db import models


class User(models.Model):
    """用户类"""

    id = models.AutoField(primary_key=True, db_column='id', verbose_name='编号')
    name = models.CharField(max_length=20, db_column='name', verbose_name='名字')
    gender = models.CharField(max_length=20, db_column='gender', verbose_name='性别')
    age = models.CharField(max_length=20, db_column='age', verbose_name='年龄')
    address = models.CharField(max_length=20, db_column='address', verbose_name='籍贯')
    qq = models.CharField(max_length=20, db_column='qq', verbose_name='QQ')
    email = models.CharField(max_length=20, db_column='email', verbose_name='邮箱')

    class Meta:
        db_table = 'user'


class Admin(models.Model):
    """管理员"""

    id = models.AutoField(primary_key=True, db_column='id', verbose_name='编号')
    username = models.CharField(max_length=20, db_column='username', verbose_name='用户名')
    password = models.CharField(max_length=20, db_column='password', verbose_name='密码')

    class Meta:
        db_table = 'manager'

~~~

### 2. 配置MySQL数据库

> 参考上[一篇博客](https://fabian4.gitee.io/55582.html)

### 3. 迁移数据模型

~~~bash
# 迁移 Django 内置的管理数据表
(venv)$ python manage.py migrate
# 生成我们的数据迁移文件
(venv)$ python manage.py makemigrations user
# 迁移我们的数据表
(venv)$ python manage.py migrate
~~~

### 3. 映射 URL

~~~python
# UserSystem\urls.py
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('user.urls')),
]
~~~

~~~python
# user\urls.py
urlpatterns = [
    # 登录的url
    path('login/', views.login, name='login'),
    # 获取图片验证码
    path('get_captcha/', views.get_captcha, name='get_captcha'),
]
~~~

### 4. 配置全局静态文件

~~~python
# UserSystem\settings.py
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]
~~~

### 5. 配置全局默认基本 url

~~~python
# user\context_processors.py
from UserSystem import settings

def base_url(request):
    if settings.DEBUG:
        return {'BASE_URL': 'http://localhost:8000/'}
~~~

~~~python
# 注册到 UserSystem\settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                # 注册我们自己配置的文件
                'user.context_processors.base_url'
            ],
        },
    },
]
~~~

### 6. 关闭 xss token

> 这是 Django 中为了避免存在跨站脚本攻击漏洞而对网页表单的强制要求，我们在开发过程中可以先将这个模块注释掉

~~~python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 注释 网页表单验证组件
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
~~~

### 7. 编写前端页面

~~~html
<!-- templates\login.html -->
<div class="container" style="width: 400px;">
  		<h3 style="text-align: center;">管理员登录</h3>
        <form action="/login/" method="post">
	      <div class="form-group">
	        <label for="username">用户名：</label>
	        <input type="text" name="username" class="form-control" id="username" placeholder="请输入用户名"/>
	      </div>
	      
	      <div class="form-group">
	        <label for="password">密码：</label>
	        <input type="password" name="password" class="form-control" id="password" placeholder="请输入密码"/>
	      </div>
	      
	      <div class="form-inline">
	        <label for="vcode">验证码：</label>
	        <input type="text" name="verifycode" class="form-control" id="verifycode" placeholder="请输入验证码" style="width: 120px;"/>
	        <a href="javascript:refreshCode()">
				<img src="{{pageContext.request.contextPath}}/get_captcha/" title="看不清点击刷新" id="vcode"/></a>
	      </div>
	      <hr/>
	      <div class="form-group" style="text-align: center;">
	        <input class="btn btn btn-primary" type="submit" value="登录">
	       </div>
	  	</form>
		
		<!-- 出错显示的信息框 -->
        {% if request.session.error %}
        <div class="alert alert-warning alert-dismissible" role="alert">
		    <button type="button" class="close" data-dismiss="alert" >
		  	<span>&times;</span></button>
		    <strong>{{request.session.error}}</strong>
        </div>
        {% endif %}
    </div>
~~~

### 8. 编写验证码生成

~~~python
# user\captcha.py
import os
import random

from io import BytesIO

from PIL import Image
from PIL import ImageFilter
from PIL.ImageDraw import Draw
from PIL.ImageFont import truetype

from UserSystem.settings import BASE_DIR


class Bezier(object):
    """贝塞尔曲线"""

    def __init__(self):
        self.tsequence = tuple([t / 20.0 for t in range(21)])
        self.beziers = {}

    def make_bezier(self, n):
        """绘制贝塞尔曲线"""
        try:
            return self.beziers[n]
        except KeyError:
            combinations = pascal_row(n - 1)
            result = []
            for t in self.tsequence:
                tpowers = (t ** i for i in range(n))
                upowers = ((1 - t) ** i for i in range(n - 1, -1, -1))
                coefs = [c * a * b for c, a, b in zip(combinations,
                                                      tpowers, upowers)]
                result.append(coefs)
            self.beziers[n] = result
            return result


class Captcha(object):
    """验证码"""

    def __init__(self, width, height, fonts=None, color=None):
        self._image = None
        self._fonts = fonts if fonts else \
            [os.path.join(BASE_DIR, 'static/fonts', font).replace('\\', '/')
             for font in ['Sujeta.ttf', 'Sewer Sys.ttf']]
        self._color = color if color else random_color(0, 200, random.randint(220, 255))
        self._width, self._height = width, height

    @classmethod
    def instance(cls, width=200, height=75):
        prop_name = f'_instance_{width}_{height}'
        if not hasattr(cls, prop_name):
            setattr(cls, prop_name, cls(width, height))
        return getattr(cls, prop_name)

    def background(self):
        """绘制背景"""
        Draw(self._image).rectangle([(0, 0), self._image.size],
                                    fill=random_color(230, 255))

    def smooth(self):
        """平滑图像"""
        return self._image.filter(ImageFilter.SMOOTH)

    def curve(self, width=4, number=6, color=None):
        """绘制曲线"""
        dx, height = self._image.size
        dx /= number
        path = [(dx * i, random.randint(0, height))
                for i in range(1, number)]
        bcoefs = Bezier().make_bezier(number - 1)
        points = []
        for coefs in bcoefs:
            points.append(tuple(sum([coef * p for coef, p in zip(coefs, ps)])
                                for ps in zip(*path)))
        Draw(self._image).line(points, fill=color if color else self._color, width=width)

    def noise(self, number=50, level=2, color=None):
        """绘制扰码"""
        width, height = self._image.size
        dx, dy = width / 10, height / 10
        width, height = width - dx, height - dy
        draw = Draw(self._image)
        for i in range(number):
            x = int(random.uniform(dx, width))
            y = int(random.uniform(dy, height))
            draw.line(((x, y), (x + level, y)),
                      fill=color if color else self._color, width=level)

    def text(self, captcha_text, fonts, font_sizes=None, drawings=None, squeeze_factor=0.75, color=None):
        """绘制文本"""
        color = color if color else self._color
        fonts = tuple([truetype(name, size)
                       for name in fonts
                       for size in font_sizes or (65, 70, 75)])
        draw = Draw(self._image)
        char_images = []
        for c in captcha_text:
            font = random.choice(fonts)
            c_width, c_height = draw.textsize(c, font=font)
            char_image = Image.new('RGB', (c_width, c_height), (0, 0, 0))
            char_draw = Draw(char_image)
            char_draw.text((0, 0), c, font=font, fill=color)
            char_image = char_image.crop(char_image.getbbox())
            for drawing in drawings:
                d = getattr(self, drawing)
                char_image = d(char_image)
            char_images.append(char_image)
        width, height = self._image.size
        offset = int((width - sum(int(i.size[0] * squeeze_factor)
                                  for i in char_images[:-1]) -
                      char_images[-1].size[0]) / 2)
        for char_image in char_images:
            c_width, c_height = char_image.size
            mask = char_image.convert('L').point(lambda i: i * 1.97)
            self._image.paste(char_image,
                              (offset, int((height - c_height) / 2)),
                              mask)
            offset += int(c_width * squeeze_factor)

    @staticmethod
    def warp(image, dx_factor=0.3, dy_factor=0.3):
        """图像扭曲"""
        width, height = image.size
        dx = width * dx_factor
        dy = height * dy_factor
        x1 = int(random.uniform(-dx, dx))
        y1 = int(random.uniform(-dy, dy))
        x2 = int(random.uniform(-dx, dx))
        y2 = int(random.uniform(-dy, dy))
        warp_image = Image.new(
            'RGB',
            (width + abs(x1) + abs(x2), height + abs(y1) + abs(y2)))
        warp_image.paste(image, (abs(x1), abs(y1)))
        width2, height2 = warp_image.size
        return warp_image.transform(
            (width, height),
            Image.QUAD,
            (x1, y1, -x1, height2 - y2, width2 + x2, height2 + y2, width2 - x2, -y1))

    @staticmethod
    def offset(image, dx_factor=0.1, dy_factor=0.2):
        """图像偏移"""
        width, height = image.size
        dx = int(random.random() * width * dx_factor)
        dy = int(random.random() * height * dy_factor)
        offset_image = Image.new('RGB', (width + dx, height + dy))
        offset_image.paste(image, (dx, dy))
        return offset_image

    @staticmethod
    def rotate(image, angle=25):
        """图像旋转"""
        return image.rotate(random.uniform(-angle, angle),
                            Image.BILINEAR, expand=1)

    def generate(self, captcha_text='', fmt='PNG'):
        """生成验证码(文字和图片)"""
        self._image = Image.new('RGB', (self._width, self._height), (255, 255, 255))
        self.background()
        self.text(captcha_text, self._fonts,
                  drawings=['warp', 'rotate', 'offset'])
        self.curve()
        self.noise()
        self.smooth()
        image_bytes = BytesIO()
        self._image.save(image_bytes, format=fmt)
        return image_bytes.getvalue()


def pascal_row(n=0):
    """生成Pascal三角第n行"""
    result = [1]
    x, numerator = 1, n
    for denominator in range(1, n // 2 + 1):
        x *= numerator
        x /= denominator
        result.append(x)
        numerator -= 1
    if n & 1 == 0:
        result.extend(reversed(result[:-1]))
    else:
        result.extend(reversed(result))
    return result


def random_color(start=0, end=255, opacity=255):
    """获得随机颜色"""
    red = random.randint(start, end)
    green = random.randint(start, end)
    blue = random.randint(start, end)
    if opacity is None:
        return red, green, blue
    return red, green, blue, opacity
~~~

### 9. 编写视图函数

~~~python
# user\views.py
# 登录函数
def login(request):
    if request.method == 'GET':
        return render(request, 'login.html')
    if request.method == 'POST':
        verifycode = request.POST.get('verifycode')
        if verifycode == request.session.get('captcha'):
            username = request.POST.get('username')
            password = request.POST.get('password')
            try:
                login_user = models.Admin.objects.get(username=username, password=password)
                request.session['login_user'] = login_user.username
                return render(request, 'index.html')
            except ObjectDoesNotExist:
                request.session['error'] = '账号或密码错误'
                return render(request, 'login.html')
        else:
            request.session['error'] = '验证码错误'
            return render(request, 'login.html')


ALL_CHARS = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'

# 获取验证码文本
def get_captcha_text(length=4):
    selected_chars = random.choices(ALL_CHARS, k=length)
    return ''.join(selected_chars)

# 返回验证码文件
def get_captcha(request):
    """验证码"""
    captcha_text = get_captcha_text()
    request.session['captcha'] = captcha_text
    image_data = Captcha.instance().generate(captcha_text)
    print(captcha_text)
    return HttpResponse(image_data, content_type='image/png')
~~~

## 二、分页查询和多条件查询

### 1.前端页面

~~~html
<div>{{request.session.login_user}}欢迎您</div>
<div class="container">
    <h3 style="text-align: center">用户信息列表</h3>
    <div style="float: left;">
        <form class="form-inline" action="/find_all/?currentPage=1" method="post">
            <div class="form-group">
                <label for="exampleInputName2">姓名</label>
                <input type="text" name="name" value="{{ page_info.name|default_if_none:"" }}" class="form-control" id="exampleInputName2" >
            </div>
            <div class="form-group">
                <label for="exampleInputName3">籍贯</label>
                <input type="text" name="address" value="{{ page_info.address|default_if_none:"" }}" class="form-control" id="exampleInputName3" >
            </div>

            <div class="form-group">
                <label for="exampleInputEmail2">邮箱</label>
                <input type="text" name="email" value="{{ page_info.email|default_if_none:"" }}" class="form-control" id="exampleInputEmail2"  >
            </div>
            <button type="submit" class="btn btn-default">查询</button>
        </form>
    </div>
    <div style="float: right; margin: 5px">
        <td colspan="8" align="center"><a class="btn btn-primary" href="{{ RequestContext.BASE_URL }}/add/">添加联系人</a></td>
        <td colspan="8" align="center"><a class="btn btn-primary" href="javascript:void(0);" id="delSelected">删除选中</a></td>
    </div>
    <form id="form" action="/delSelected/" method="post">
        <table border="1" class="table table-bordered table-hover">
            <tr class="success">
                <th><input type="checkbox" id="firstCb"></th>
                <th>编号</th>
                <th>姓名</th>
                <th>性别</th>
                <th>年龄</th>
                <th>籍贯</th>
                <th>QQ</th>
                <th>邮箱</th>
                <th>操作</th>
            </tr>
            {% for user in user_list %}
                <tr>
                    <th><input type="checkbox" name="uid" value="{{ user.pk }}"></th>
                    <td>{{ forloop.counter }}</td>
                    <td>{{ user.fields.name }}</td>
                    <td>{{ user.fields.gender }}</td>
                    <td>{{ user.fields.age }}</td>
                    <td>{{ user.fields.address }}</td>
                    <td>{{ user.fields.qq }}</td>
                    <td>{{ user.fields.email }}</td>
                    <td><a class="btn btn-default btn-sm" href="{{ RequestContext.BASE_URL }}/findUserById/?id={{ user.pk }}">修改</a>&nbsp;
                        <a class="btn btn-default btn-sm" href="javascript:deleteUser({{ user.pk }});">删除</a></td>
                </tr>
            {% endfor %}
        </table>
    </form>
    <div>
        <nav aria-label="Page navigation">
            <ul class="pagination">
                <li>
                    <a href="{{ RequestContext.BASE_URL }}/find_all/?currentPage={{ page_info.currentPage|add:-1 }}&name={{ page_info.name }}&address={{ page_info.address }}&email={{ page_info.email }}" aria-label="Previous">
                        <span aria-hidden="true">&laquo;</span>
                    </a>
                </li>
                {% for i in page_info.pages %}
                    {% if i == page_info.currentPage %}
                        <li class="active"><a href="{{ RequestContext.BASE_URL }}/find_all/?currentPage={{ i }}&name={{ page_info.name }}&address={{ page_info.address }}&email={{ page_info.email }}">{{ i }}</a></li>
                    {% endif %}
                    {% if i != page_info.currentPage %}
                        <li><a href="{{ RequestContext.BASE_URL }}/find_all/?currentPage={{ i }}&name={{ page_info.name }}&address={{ page_info.address }}&email={{ page_info.email }}">{{ i }}</a></li>
                    {% endif %}
                {% endfor %}
                <li>
                    <a href="{{ RequestContext.BASE_URL }}/find_all/?currentPage={{ page_info.currentPage|add:1 }}&name={{ page_info.name }}&address={{ page_info.address }}&email={{ page_info.email }}" aria-label="Next">
                        <span aria-hidden="true">&raquo;</span>
                    </a>
                </li>
                <span style="font-size: 25px;margin-left: 5px;">
                    共{{ page_info.total }}条记录，共{{ page_info.totalPage }}页
                </span>

            </ul>
        </nav>
    </div>
</div>
~~~

### 2. 分页展示

> 这里我们采用一个 **django** 的 **Paginator** 类来实现自动的分页结果显示

~~~python
# 查询所有并返回序列化队列
# 将查询条件封装到一个字典作为参数传入查询方法
user_list = serializers.serialize("json", models.User.objects.filter(**search_dict))
# 将队列反序列化以便渲染到页面上
user_list = json.loads(user_list)
# 每10个为1页
p = Paginator(user_list, 10)
# 以字典格式返回给 list.html 进行渲染
return render(request, "list.html", {'user_list': p.page(current_page).object_list, 'page_info': page_info})
~~~

### 3. 前端参数获取

~~~python
# 如果是通过 get 方法
if request.method == 'GET':
    name = request.GET.get("name", default='')
    address = request.GET.get("address", default='')
    email = request.GET.get("email", default='')
    current_page = int(request.GET.get("currentPage", default=1))
# 如果是通过 post 方法
if request.method == 'POST':
    name = request.POST.get("name", default='')
    address = request.POST.get("address", default='')
    email = request.POST.get("email", default='')
    current_page = int(request.POST.get("currentPage", default=1))
~~~

### 4. 参数处理封装和返回

~~~python
# 将查询条件封装为字典
search_dict = dict()
	# 判空防止误传
    if name:
        search_dict['name'] = name
    if address:
        search_dict['address'] = address
    if email:
        search_dict['email'] = email
    user_list = serializers.serialize("json", models.User.objects.filter(**search_dict))
    user_list = json.loads(user_list)
    p = Paginator(user_list, 10)
	# 防止出现参数超出范围的情况
    if current_page < 1:
        current_page = 1
    if current_page > p.num_pages:
        current_page = p.num_pages
	# 将页码封装到一个列表
    pages = []
    i = 1
    while i <= p.num_pages:
        pages.append(i)
        i += 1
    # 将所有信息封装为字典返回给前端
    page_info = {"total": len(user_list),
                 "totalPage": p.num_pages,
                 "currentPage": current_page,
                 "name": name,
                 "address": address,
                 "email": email,
                 "pages": pages
                 }
    return render(request, "list.html", {'user_list': p.page(current_page).object_list, 'page_info': page_info})
~~~

## 三、增删改数据

### 1. 配置 url 路径

~~~python
# user\urls.py
urlpatterns = [
    path('login/', views.login, name='login'),
    path('get_captcha/', views.get_captcha, name='get_captcha'),
    path('find_all/', views.find_all, name='find_all'),
    path('add/', views.add, name='add'),
    path('findUserById/', views.findUserById, name='findUserById'),
    path('update/', views.update, name='update'),
    path('delUser/', views.delUser, name='delUser'),
    path('delSelected/', views.delSelected, name='delSelected'),
]
~~~

### 2. 前端界面

~~~html
<!-- add.html -->
<div>{{ loginUser.name }}欢迎您</div>
<div class="container">
    <center><h3>添加联系人页面</h3></center>
    <form action="{{ RequestContext.BASE_URL }}/add/" method="post">
        <div class="form-group">
            <label for="name">姓名：</label>
            <input type="text" class="form-control" id="name" name="name" placeholder="请输入姓名">
        </div>

        <div class="form-group">
            <label>性别：</label>
            <input type="radio" name="gender" value="男" checked="checked"/>男
            <input type="radio" name="gender" value="女"/>女
        </div>

        <div class="form-group">
            <label for="age">年龄：</label>
            <input type="text" class="form-control" id="age" name="age" placeholder="请输入年龄">
        </div>

        <div class="form-group">
            <label for="address">籍贯：</label>
            <input type="text" class="form-control" id="address" name="address" placeholder="请输入籍贯">
        </div>

        <div class="form-group">
            <label for="qq">QQ：</label>
            <input type="text" class="form-control" id="qq" name="qq" placeholder="请输入QQ号码"/>
        </div>

        <div class="form-group">
            <label for="email">Email：</label>
            <input type="text" class="form-control" id="email" name="email" placeholder="请输入邮箱地址"/>
        </div>

        <div class="form-group" style="text-align: center">
            <input class="btn btn-primary" type="submit" value="提交" />
            <input class="btn btn-default" type="reset" value="重置" />
            <a class="btn btn-default" href="list.jsp" role="button">返回</a>
        </div>
    </form>
</div>
~~~

~~~html
<!-- update.html -->
<div>{{request.session.login_user}}欢迎您</div>
        <div class="container" style="width: 400px;">
        <h3 style="text-align: center;">修改联系人</h3>
        <form action="{{ RequestContext.BASE_URL }}/update/" method="post">
            <!--  隐藏域 提交id-->
            <input type="hidden" name="id" value="{{ u.pk }}">
          <div class="form-group">
            <label for="name">姓名：</label>
            <input type="text" class="form-control" id="name" name="name" value="{{ u.fields.name }}" placeholder="请输入姓名" />
          </div>

          <div class="form-group">
            <label>性别：</label>
              {% if u.fields.gender == '男' %}
                    <input type="radio" name="gender" value="男" checked />男
                    <input type="radio" name="gender" value="女"/>女
              {% endif %}
              {% if u.fields.gender == '女' %}
                    <input type="radio" name="gender" value="男"/>男
                    <input type="radio" name="gender" value="女" checked />女
              {% endif %}
          </div>

          <div class="form-group">
            <label for="age">年龄：</label>
            <input type="text" class="form-control" id="age" value="{{ u.fields.age }}" name="age" placeholder="请输入年龄" />
          </div>

          <div class="form-group">
            <label for="address">籍贯：</label>
              <input type="text" class="form-control" id="address" name="address" value="{{ u.fields.address }}" placeholder="请输入籍贯" />
            </select>
          </div>

          <div class="form-group">
            <label for="qq">QQ：</label>
            <input type="text" class="form-control" id="qq" value="{{ u.fields.qq }}" name="qq" placeholder="请输入QQ号码"/>
          </div>

          <div class="form-group">
            <label for="email">Email：</label>
            <input type="text" class="form-control" id="email" value="{{ u.fields.email }}" name="email" placeholder="请输入邮箱地址"/>
          </div>

             <div class="form-group" style="text-align: center">
                <input class="btn btn-primary" type="submit" value="提交" />
                <input class="btn btn-default" type="reset" value="重置" />
                 <a class="btn btn-default" href="{{ RequestContext.BASE_URL }}/find_all/?currentPage=1" role="button">返回</a>
             </div>
        </form>
        </div>
    </body>
~~~

### 3. 后端响应

~~~python
# user\views.py
def add(request):
    if request.method == 'GET':
        return render(request, "add.html")
    if request.method == 'POST':
        models.User.objects.create(name=request.POST.get('name'),
                                   gender=request.POST.get('gender'),
                                   age=request.POST.get('age'),
                                   address=request.POST.get('address'),
                                   qq=request.POST.get('qq'),
                                   email=request.POST.get('email'))
        return redirect("find_all")


def findUserById(request):
    no = request.GET.get("id")
    user = serializers.serialize("json", models.User.objects.all().filter(id=no))
    user = json.loads(user)
    return render(request, 'update.html', {'u': user[0]})


def update(request):
    no = request.POST.get("id")
    models.User.objects.filter(id=no).update(name=request.POST.get('name'),
                                             gender=request.POST.get('gender'),
                                             age=request.POST.get('age'),
                                             address=request.POST.get('address'),
                                             qq=request.POST.get('qq'),
                                             email=request.POST.get('email'))
    return redirect("find_all")


def delUser(request):
    no = request.GET.get("id")
    models.User.objects.filter(id=no).delete()
    return redirect("find_all")


def delSelected(request):
    no = request.POST.getlist("uid")
    for n in no:
        models.User.objects.filter(id=n).delete()
    return redirect("find_all")
~~~

## 四、登录检测和拦截

### 1. 编写拦截中间件

~~~python
# user\middleware.py
from django.shortcuts import HttpResponseRedirect

try:
    from django.utils.deprecation import MiddlewareMixin
except ImportError:
    MiddlewareMixin = object


class SimpleMiddleware(MiddlewareMixin):
    @staticmethod
    def process_request(request):
		# 只对两个路径放行
        if request.path != '/login/' and request.path != '/get_captcha/':
            if request.session.get('login_user', None):
                pass
            else:
                return HttpResponseRedirect('/login/')

~~~

### 2. 注册中间件

~~~python
# UserSystem\settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # 注册自己编写的中间件
    'user.middleware.SimpleMiddleware'
]
~~~

## 五、总结

- 基于 **Django** 框架搭建的后台还是具有很多鲜明的特色的
- 注意 **Django**自带的引擎模板的语法，或者也可以更换其他模板引擎
- **Django** 模板引擎中过滤器的使用
- **Django**中间件的自定义和注册



