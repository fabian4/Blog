---
title: pygame常用函数
date: 2020/4/28
description: 开始你的第一个 python 小游戏——pygame快速上手
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/mtUT9iSPeX8YJav.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/mtUT9iSPeX8YJav.jpg'
categories:
  - python
tags:
  - python
  - pygame
abbrlink: 12400
---

# pygame 会用这些函数就够了

借鉴了pygame的官方文档和一些中译版，对于我们实际写代码时较常用的一些函数和方法进行了整理

前15个可以说是较为常用的函数，后面的由于不太常用就有所省略。

本文档只是作为参考，具体使用还请参照[官方文档](https://www.pygame.org/docs/)。如果整理的哪里有问题还希望大家指正。

1. [pygame.sprite：基本游戏对象](#1)
2. [pygame,event：处理事件](#2)
3. [pygame.key：键盘按键](#3)
4. [pygame：顶层模块](#4)
5. [pygame.time：时间模块](#5)
6. [pygame.diaplay：控制窗口显示](#6)
7. [pygame.Rect：图像矩形模块](#7)
8. [pygame.Color：颜色模块](#8)
9. [pygame.image：图片模块](#9)
10. [pygame.Surface：表示图像](#10)
11. [pygame.mixer：声音模块](#11)
12. [pygame.draw：绘制图像](#12)
13. [pygame.font：字体模块](#13)
14. [pygame.mixer.music：音乐模块](#14)
15. [pygame.mouse：鼠标移动](#15)
16. [pygame.BufferProxy：缓冲对象](#16)
17. [pygame.cdrom：音频光盘](#17)
18. [pygame.PixelArray：像素数组](#18)
19. [pygame.cursor：鼠标光标](#19)
20. [pygame.scrap：剪切板](#20)
21. [pygame.andarray：音频采样](#21)
22. [pygame.transform：改变形态](#22)
23. [pygame.tests：测试模块](#23)
24. [pygame.freetype：计算机字体](#24)
25. [pygame.joystick：外接设备](#25)
26. [pygame.locals：常量定义](#26)
27. [pygame.surfarray：访问像素](#27)
28. [pygame.Overlay：图像叠加](#28)

##  <div id='1'>1. pygame.sprite</div>

该类是pygame中处理基本游戏对象类的模块

- `pygame.sprite.Sprite`：可见游戏对象的简单基类
  - `pygame.sprite.Sprite.update`：更新状态
  - `pygame.sprite.Sprite.add`：加到组中
  - `pygame.sprite.Sprite.remove`：从组中删除
  - `pygame.sprite.Sprite.kill`：从所有组中删除
  - `pygame.sprite.Sprite.groups`：包含此Sprite的组列表
- `pygame.sprite.DirtySprite`：具有更多属性和功能的Sprite的子类
- `pygame.sprite.Group`：保存和管理多个Sprite对象的容器类
  - `pygame.sprite.Group.copy`：复制组
  - `pygame.sprite.Group.add`：添加到该组
  - `pygame.sprite.Group.remove`：从组中删除
  - `pygame.sprite.Group.has`：检测是否包含
  - `pygame.sprite.Group.update`：更新状态
  - `pygame.sprite.Group.draw`：绘制图像
  - `pygame.sprite.Group.empty`：删除所有
- `pygame.sprite.collide_rect`：使用rects检测两个sprite之间的碰撞
- `pygame.sprite.collode_rect_ratio`：使用按比例缩放的rects检测碰撞
- `pygame.sprite.collide_circle`：使用圆来检测碰撞
- `pygame.sprite.collide_circle_ratio`：使用按比例缩放的圆检测
- `pygame.sprite.collide_mask`：使用蒙版检测碰撞

## <div id='2'>2. pygame.event </div>

该类是pygame中处理事件与事件队列的模块

- `pygame.event.pump`：让其内部自动处理事件
- `pygame.event.get`：从队列中获取事件
- `pygame.event.poll`：从队列中获取一个事件
- `pygame.event.wait`：等待并从队列中获取一个事件
- `pygame.event.peek`：检测某个类型事件是否在队列中
- `pygame.event.clear`：从队列中删除所有事件
- `pygame.event.set_allowed`：控制哪些事件禁止进入队列
- `pygame.event.set_blocked`：控制哪些事件允许进入队列
- `pygame.event.post`：放置一个新事件进入队列
- `pygame.event.Event`：创建一个新的事件对象
- 事件种类：
  - QUIT：退出
  - KEYDOWN：按下按键
  - KEYUP：松开按键
  - MOUSEMOTION：鼠标移动
  - 其他游戏杆、游戏手柄、追踪球事件
  - 支持自定义事件

##  <div id='3'>3. pygame.key</div>

该类是pygame处理与键盘有关的模块

- `pygame.key.get_focused`：窗口获得键盘输入焦点返回True
- `pygame.key.get_pressed`：获取键盘所有按键的状态
- key属性：pygame预定义的键盘上的键位
- mod属性：pygame预定义的组合键

## <div id='4'>4. pygame </div>

该类为最顶层的模块

- `pygame.init`：初始化导入所有pygame模块
- `pygame.quit`：卸载所有导入的模块

##  <div id='5'>5. pygame.time</div>

该类是pygame中用于时间管理模块

- `pygame.time.get_ticks`：以毫秒为单位获取时间
- `pygame.time.wait`：暂停程序一段时间
- `pygame.time.delay`：暂停程序一段时间
- `pygame.time.Colck`：创建一个对象来帮助或更新时间
  - `pygame.time.Clock.tick`：更新clock对象

##   <div id='6'>6. pygame.display</div>

该类是pygame中用于控制窗口和屏幕显示的模块

- `pygame.display.init`：初始化模块
- `pygame.display.quit`：取消初始化
- `pygame.display.get_init`：判断是否初始化成功
- `pygame.display.set_mode`：初始化一个准备显示的窗口或者屏幕
  - `pygame.display.set_mode(resolution=(0,0), flags=0, depth=0)`
    - resolution：传入屏幕的长宽，若为 0 则默认当前屏幕分辨率
    - flags：控制显示类型 可以多种类型用管道符 ‘ | ’ 组合
      - pygame.FULLSCREEN 创建一个全屏显示
      - pygame.DOUBLEBUF  双缓冲模式
      - pygame.HWSURFACE 硬件加速，仅在FULLSCREEN下可以使用
      - pygame.RESIZABLE 创建一个可调整尺寸的窗口
      - pygame.NOFRAME 创建一个没有边框和控制按钮的窗口
    - depth：深度参数，一般不传值
- `pygame.display.get_surface`：获取当前显示的 Surface 对象
- `pygame.display.flip`：将**完整**待显示的 Surface 对象更新到屏幕上
- `pygame.display.update`：更新**部分**软件界面显示
- `pygame.display.iconify`：最小化显示 Surface 对象
- `pygame.display.toggle_fullscreen`：切换全屏显示和窗口显示
- `pygame.display.set_icon`：更改显示窗口的系统头像
- `pygame.display.set_caption`：设置当前窗口标题
- `pygame.display.get_caption`：获取当前窗口标题

##  <div id='7'>7. pygame.Rect</div>

该类是pygame中直接操作对象外界矩形的模块

- `pygame.Rect`：创建矩形对象
  - `pygame.Rect(left, top, width, height)`：根据左上角坐标和长宽创建矩形
  - `pygame.Rect((left, top), (width, height))`：根据左上角坐标和长宽创建矩形
  - 该对象具有用来移动和对齐的参数
    - x, y
    - top, left, bottom, right
    - topleft, bottomleft, topright, bottomright
    - midtop, midleft, midbottom, midright
    - center, centerx, centery
    - size, width, height
    - w,h
- `pygame.Rect.copy`：复制矩形
- `pygame.Rect.move`：移动矩形
  - `pygame.Rect.move(x, y)`：传入偏移量
- `pygame.Rect.contains`：测试一个矩形是否在另一个矩形内部，传入矩形对象
- `pygame.Rect.collidepoint`：测试一个点是否在矩形内
- `pygame.Rect.colliderect`：测试两个矩形是否重叠

##   <div id='8'>8. pygame.Color</div>

该类是pygame中用于描述颜色的模块

- `pygame.Color`：返回一个颜色
  - Color(name)：传入名称
  - Color(r, g, b, a)：传入rgba参数
  - Color(rgbvalue)：传入rgb参数
- `pygame.Color.r`：获取或设置其红色值
- `pygame.Color.g`：获取或设置其绿色值
- `pygame.Color.b`：获取或设置其蓝色值
- `pygame.Color.a`：获取或设置其alpha通道值
- `pygame.Color.cmy`：获取或设置其 cmy (青、洋红和黄) 值
- `pygame.Color.normalize`：返回标准化 RGBA 值

##  <div id='9'>9. pygame.image</div>

该类是pygame中处理图像传输的模块

- `pygame.image.load`：从文件中加载新图像
  - 传入文件路径和名称
  - 支持 JPG、PNG、GIF、BMP、PCX、TGA、TIF、LBM、PBM、XPM格式
- `pygame.image.save`：将图像保存到本地
  - 传入 Surface 对象和路径
  - 支持BMP、TGA、PNG、JPEG格式

##  <div id='10'>10. pygame.Surface</div>

该类是pygame中用于表示图像的模块

- `pygame.Surface`：创建对象
  - `pygame.Surface((width, height), flags=0, depth=0, masks=None)`
  - `pygame.Surface((width, height), flags=0, Surface)`
- `pygame.Surface.blit`：将一个图像绘制到另一个上方
- `pygame.Surface.convert`：修改像素格式
- `pygame.Surface.copy`：拷贝对象
- `pygame.Surface.fill`：使用纯色填充对象
- `pygame.Surface.scroll`：移动对象

##  <div id='11'>11. pygame.mixer</div>

该类是pygame中用来加载和播放声音的模块，默认有8个声道

- `pygame.mixer.init`：初始化模块
- `pygame.mixer.stop`：停止播放所有声道
- `pygame.mixer.pause`：暂停播放所有声道
- `pygame.mixer.unpause`：恢复暂停播放所有声道
- `pygame.mixer.fadeout`：停止前淡出所有声音
  - 传入时间参数，设置淡出时间

## <div id='12'>12. pygame.draw </div>

该类是pygame中用于绘制图形的模块

- `pygame.draw.rect`：绘制矩形
  - `pygame.draw.rect(Surface, color, Rect, width=0)`
    - Surface：绘制在 Surface 对象上
    - color：传入 RGB 三元组或 RGBA 四元组
    - width：指定边框宽度，0 表示填充该矩形
- `pygame.draw.polygon`：绘制多边形
  - `pygame.draw.polygon(Surface, color, pointlist, width=0)`
    - pointlist：指定多边形的各个顶点
- `pygame.draw.circle`：绘制圆形
  - `pygame.draw.circle(Surface, color, pos, radius, width=0)`
    - pos：圆心位置
    - radius：半径
- `pygame.draw.ellipse`：绘制椭圆
  - `pygame.draw.ellipse(Surface, color, Rect, width=0)`
    - Rect：椭圆外切矩形
- `pygame.draw.arc`：绘制弧线
  - `pygame.draw.arc(Surface, color, Rect, start_angle, stop_angle, width=1)`
    - Rect：弧线所在椭圆外切矩形
    - angle：弧线开始和结束位置
- `pygame.draw.line`：绘制线段
  - `pygame.draw.line(Surface, color, start_pos, end_pos, width=1)`
- `pygame.draw.lines`：绘制多条线段
  - `pygame.draw.lines(Surface, color, closed, pointlist, width=1)`
    - pointlist：一系列点
    - closed：若为 True 则绘制首尾相连

##  <div id='13'>13. pygame.font</div>

该类是pygame中加载和表示字体的模块

- `pygame.font.init`：初始化
- `pygame.font.quit`：退出
- `pygame.font.get_init`：判断是否初始化
- `pygame.font.get_font`：获取所有可使用的字体
- `pygame.font.match_font`：在系统中搜索字体
- `pygame.font.SysFont`：从系统字体库冲创建一个对象
- `pygame.font.Font`：从字体文件中创建一个对象
  - `pygame.font.Font.render`：在新的 Surface 对象上绘制文本
  - `pygame.font.Font.size`：确定字体大小
  - `pygame.font.Font.set_underline`：是否加下划线
  - `pygame.font.Font.set_bold`：是否加粗
  - `pygame.font.Font.set_italic`：是否设置为斜体

##  <div id='14'>14. pygame.mixer.music</div>

该类是pygame中控制音乐播放的模块，对于MP3格式的支持受限，建议OGG格式

- `pygame.mixer.music.load`：载入一个音乐文件
- `pygame.mixer.music.play`：开始播放
  - `pygame.mixer.music.play(loops=0, start=0.0)`
    - loops：控制**再**循环几次，若为3则一共播放4次
    - start：控制从哪里开始播放，单位为秒
- `pygame.mixer.music.rewind`：重新开始播放
- `pygame.mixer.music.stop`：结束播放
- `pygame.mixer.music.pause`：暂停播放
- `pygame.mixer.music.unpause`：继续播放
- `pygame.mixer.music.set_voiume`：设置音量
- `pygame.mixer.music.get_volume`：获取音量
- `pygame.mixer.music.set_pos`：设置播放位置
  - 传入一个浮点数，控制从什么时候开始播放
- `pygame.mixer.music.queue`：将一个应用文件放入队列，排在当前播放的音乐之后
- `pygame.mixer.music.set_endevent`：设置播放结束事件
- `pygame.mixer.music.get_endevent`：获取播放结束事件

##  <div id='15'>15. pygame.mouse</div>

该类是pygame中与鼠标操作相关的模块

- `pygame.mouse.get_pressed`：判断鼠标按键是否被按下
- `pygame.mouse.get_pos`：获取鼠标光标位置
- `pygame.mouse.set_pos`：设置鼠标光标位置
- `pygame.mouse.set_visible`：设置鼠标光标是否隐藏
- `pygame.mouse.set_cuisor`：设置鼠标光标在程序内的显示图像
- `pygame.mouse.get_cursor`：获取鼠标光标在程序内的显示图像

##  <div id='16'>16. pygame.BufferProxy</div>

该类是 `Surface`对象通过数组协议导出的一个**缓冲对象**

- `pygame.BufferProxy.parent`：返回该 BufferProxy 的 Surface 对象， 或者调用 BufferProxy 的对象
- `pygame.BufferProxy.length`：返回缓冲区的长度，单位是字节。类似于len字段
- `pygame.BufferProxy.length`：将缓冲区的数据拷贝并返回，为**str**或**bytes**对象
- `pygame.BufferProxy.write`：将字节写入缓冲区

## <div id='17'>17. pygame.cdrom </div>

该类是pygame中使用音频cdrom的模块， cdrom 即 Compact Disc Read-only Memory 只读光盘

- `pygame.cdrom.init`：初始化模块
- `pygame.cdrom.quit`：退出模块
- `pygame.cdrom.get_init`：判断是否初始化成功，成功返回 **true**
- `pygame.cdrom.get_count`：返回 cd 驱动器的个数
- `pygame.cdrom.CD`：管理驱动的类
  - `pygame.cdrom.CD.init`：初始化
  - `pygame.cdrom.CD.quit`：退出
  - `pygame.cdrom.CD.get_init`：判断是否完成初始化
  - `pygame.cdrom.CD.play`：开始播放
  - `pygame.cdrom.CD.stop`：停止播放
  - `pygame.cdrom.CD.pause`：暂停播放
  - `pygame.cdrom.CD.resume`：恢复播放

##  <div id='18'>18.  pygame.PixelArray</div>

该类为pygame中直接处理访问像素的模块

- `pygame.PixelArray.surface`：获取使用的 surface 对象
- `pygame.PixelArray.itemsize`：返回像素数组项字节大小
- `pygame.PixelArray.make_surface`：根据当前 PixelArray 创建新的 surface 对象
- `pygame.PixelArray.extract`：从 PixelArray 中提取传递颜色

##  <div id='19'>19. pygame.cursors</div>

该类是pygame中使用光标资源的模块

- `pygame.cursors.compile`：又纯字符串创建二进制光标数据
- `pygame.cursors.load_xbm`：由xbm文件载入光标数据

## <div id='20'>20. pygame.scrap </div>

该类是pygame中用于支持剪切板的模块

- `pygame.scrap.init`：初始化
- `pygame.scrap.get`：从剪切板获取指定类型数据
- `pygame.scrap.put`：将数据放入剪切板
- `pygame.scrap.contains`：检测是否获得

##  <div id='21'>21. pygame.sndarray</div>

该类是pygame中访问音频采样数据的模块

- `pygame.sndarray.array`：将一个音频采样复制到一个数组
- `pygame.sndarray.make_sound`：将一个数组变成一个音频对象

##  <div id='22'>22. pygame.transform</div>

该类是pygame中用于改变surface形态的模块

- `pygame.transform.flip`：垂直和水平翻转
- `pygame.transform.scale`：调整大小到新的分辨率
- `pygame.transform.rotata`：旋转图像

## <div id='23'>23. pygame.tests </div>

该类是pygame中用于单元测试的模块

- `pygame.tests.run`：运行测试套件

##  <div id='24'>24.  pygame.freetype</div>

该类是pygame中用来加载和呈现计算机字体的模块，不常用

##  <div id='25'>25. pygame.joystick</div>

该类是pygame中与游戏杆、游戏手柄、追踪球交互的模块，详情参考文档

## <div id='26'>26. pygame.locals </div>

该类是pygame中定义各种常量的模块

## <div id='27'>27. pygame.surfarray </div>

该类是pygame中使用接口访问 surface 像素的模块，不常用

## <div id='28'>28. pygame.Overlay </div>

该类是pygame中用于处理视频叠加图像的模块，不常用