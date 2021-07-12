---
title: 超级玛丽的 python 实现
date: 2020/4/29
description: 超级玛丽的 python 实现
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/cR1JOeNEmYnCBHt.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/cR1JOeNEmYnCBHt.png'
categories:
  - python
tags:
  - pygame
  - python
abbrlink: 2041
---

# 超级玛丽的 python 实现

在经过三四天的摸索，参考了Github上的一个大神的代码，也算是初步搭建起了自己的超级玛丽，下面就给大家分享一些自己踩的坑。

这里是[Github上大神的代码](https://github.com/justinmeister/Mario-Level-1)，对超级玛丽的第一关进行了很好的还原。

推荐一下Github上一个pygame的[游戏仓库](https://github.com/CharlesPikachu/Games)

推荐一本《python和pygame游戏开发指南》，想要深入研究的朋友可以去翻阅一下

推荐一个2D游戏的[素材网站](http://www.aigei.com/game/)

关于 pygame 模块可以查看[官方文档](https://www.pygame.org/docs/)，推荐一下CSDN上的[中译版](https://blog.csdn.net/Enderman_xiaohei/article/details/87708373)，毕竟官网的配色比较靓眼。或者看我前一篇博客的大致整理。

在开始之前你需要：

- 掌握 python 的基本语法
- 熟悉 pygame 模块的基本使用

由于pygame游戏的基本入门在之前一篇博客中有，这里就不再赘述

## 1. 画面和角色的导入

创建屏幕、从图片中导入Mario

~~~python
# 屏幕创建和初始化参数 
self.screen = pygame.display.set_mode((WIDTH, HEIGHT))
self.rect = self.screen.get_rect()
pygame.display.set_caption(TITLE)
~~~

~~~python
# 加载关卡图片
self.background = load_image('level.png')
self.back_rect = self.background.get_rect()
	# 这里载入图片需要乘上特定的系数来适配屏幕的尺寸
self.background = pygame.transform.scale(self.background,
                                     (int(self.back_rect.width * BACKGROUND_SIZE),
                                      int(self.back_rect.height * BACKGROUND_SIZE))).convert()
~~~

~~~python
# 导入Mario
self.sheet = load_image('mario.png')
	# 这里由于Mario会有奔跑和跳跃的速度，所以需要导入一整张图片再裁剪使用。
self.load_from_sheet()
	# 初始化角色的一些基本常量
self.rect = self.image.get_rect()
self.pos = vec(WIDTH * 0.5, GROUND_HEIGHT - 70)
self.vel = vec(0, 0)
self.acc = vec(0, 0)
~~~

## 2. 角色的落地、跳跃和移动

在这之前要解决一下Mario如何才能站在我们定义的地面上

~~~python
self.acc = vec(0, GRAVITY)
if GROUND_HEIGHT < self.mario.pos.y:
    # 如果Mario低于我们定义的地面，就之间将他的所有速度加速度都置零，之间放在我们的地面上
    # 如果速度和加速度不值零，可能会出现Mario卡在地面上抖动的情况，由于y值的不断变化
    self.mario.acc.y = 0
    self.mario.vel.y = 0
    self.mario.pos.y = self.ground_collide.rect.top
 self.mario.landing = True
~~~

正如之前那一篇文章所说，角色的移动如果只是单纯的实现**以像素为单位向左向右移动**，无疑会很影响玩家的游戏体验正如以下![5](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/tcpYBHFKq8ISP54.gif)

可以明显感觉到两个方向的运动的不同体验，下面是两个方向的代码可以简单的对比一下

~~~python
keys = pygame.key.get_pressed()
	if keys[pygame.K_RIGHT]:
        # 向右
   	 	self.pos.x += 5 # ------------------------简单的改变位置
    elif keys[pygame.K_LEFT]:
        # 向左
       if self.vel.x < 0:
        	# 这里很细节的加入了一个转向的速度控制
            self.acc.x = -TURNAROUND
            if self.vel.x >= 0:  # ------------------------改变加速度来改变运动
                self.acc.x = -ACC
        # 这里加入了一个最大速度限制
    if abs(self.vel.x) < MAX_SPEED:
        self.vel.x += self.acc.x
    elif keys[pygame.K_LEFT]:
        self.vel.x = -MAX_SPEED
    elif keys[pygame.K_RIGHT]:
        self.vel.x = MAX_SPEED
    # 这里对加速度和速度进行计算得出位移并在下一帧时改变Mario的位置
    self.acc.x += self.vel.x * FRICTION
    # 同时还要引用一个 摩擦力 的概念，随着速度的增大而增大
    self.vel += self.acc
    self.pos += self.vel + 0.5 * self.acc
    self.rect.midbottom = self.pos
~~~

对于角色的跳跃，一定要对其状态加以限制，让其必须在 "落地" 的状态下才能开始跳跃，不然就会产生下面的情况

![1](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/kLSh19zaAJ5KptX.gif)

为了避免这种情况，我们引入了一个`self.landing`状态，只有但其在为`true`的时候才能响应跳跃事件

~~~ python
if keys[pygame.K_SPACE]:
    if self.landing:
        # 这里跳跃的参数，只是给Mario一个向上的速度，类似于物理中的上抛运动
        self.vel.y = -JUMP
~~~

这里以上的所有大写常量参数都是定义在单独的配置文件中，方便修改。其参数的大小可以自行调节找出最合适的一组

对于这些运动的参数大家可以自己去调试调试，尝试一下不同的操作体验，也可以去文末的代码自寻

## 3. 角色的动作图片的切换

在提供的素材中是张表如图，我们需要自行裁剪下我们所需要的图片

![mario](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/CRtDbnMAKFuGarN.png)

这里可以在工具类中定义一个加载图片的方法

```python
def load_image(filename):
    src = os.path.dirname(os.path.abspath(__file__))
    path = os.path.join(src, 'resources', 'graphics', filename)
    return pygame.image.load(path)
```

从sheet中裁切图片

```python
# 裁切方法
def get_image(self, x, y, width, height):
    image = pg.Surface([width, height])
    rect = image.get_rect()
    image.blit(self.sheet, (0, 0), (x, y, width, height))
    image.set_colorkey(BLACK)
    image = pg.transform.scale(image,
                               (int(rect.width * MARIO_SIZE),
                                int(rect.height * MARIO_SIZE)))
    return image

# 裁切并加入容器
def load_from_sheet(self):
    self.right_frames = []
    self.left_frames = []

    self.right_frames.append(
        self.get_image(178, 32, 12, 16)) # 站立
    self.right_frames.append(
        self.get_image(80, 32, 15, 16))  # 跑 1
    self.right_frames.append(
        self.get_image(96, 32, 16, 16))	 # 跑 2
    self.right_frames.append(
        self.get_image(112, 32, 16, 16))  # 跑 3
    self.right_frames.append(
        self.get_image(144, 32, 16, 16))  # 跳 

    # 将向右的图片水平翻转就是向左的图片了
    for frame in self.right_frames:
        new_image = pg.transform.flip(frame, True, False)
        self.left_frames.append(new_image)

    # 最后全部加入容器方便我们之间通过下标
    self.frames = self.right_frames + self.left_frames
```

图片的切换，一开始我是采用一帧换一张图片的方向，下面是代码

~~~python
# 定义一个方法来通过运动方向更换图片
def walk(self, facing):
    if facing == 'right':
        if self.image_index > 3:
            self.image_index = 0
     if facing == 'left':
        if self.image_index > 8:
            self.image_index = 5
        if self.image_index < 5:
            self.image_index = 5
      self.image_index += 1
~~~

但后来在运行的时候我发现了一个让人懵逼的效果

![2](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/3bdTG8tfQKvqLPE.gif)

后来发现这是由于我的图片是每帧一换，快的让人反应不过来，产生了这种鬼畜的效果。

后来在参考大神的源码的时候，发现了一种控制图片切换速度的方法。不得不说大神还是很细节的，引入了**常量系数**和**时间戳**，通过Mario的移动速度来控制图片切换，让其更加自然平滑。下面是代码

~~~python
    # 改进后的代码
    def walk(self, facing):
        if self.image_index == 0:
            self.image_index += 1
            # 加入一个时间戳
            self.walking_timer = pg.time.get_ticks()
        else:
            # 比较时间变化和当前的Mario的速度
            if (pg.time.get_ticks() - self.walking_timer >
                    self.calculate_animation_speed()):
                self.image_index += 1
                self.walking_timer = pg.time.get_ticks()
        if facing == 'right':
            if self.image_index > 3:
                self.image_index = 0
        if facing == 'left':
            if self.image_index > 8:
                self.image_index = 5
            if self.image_index < 5:
                self.image_index = 5
    
    # 计算速度的方法
    def calculate_animation_speed(self):
        if self.vel.x == 0:
            animation_speed = 130
        elif self.vel.x > 0:
            animation_speed = 130 - (self.vel.x * 12)
        else:
            animation_speed = 130 - (self.vel.x * 12 * -1)
~~~

## 4. 背景图片的滚动

本来背景的移动还是比较简单的，一如飞机大战和之前的那个jumpy的游戏：
	**只需要当角色的位置超过屏幕参数的某个值(如2/3)时，角色的位置不再相对的移动，而是将角色是位移反向的加到背景上，通过背景的后退就可以实现角色的相对移动。**

但在超级玛丽中很明显是不适用的，因为整个关卡上定位了很多砖块，管道和阶梯，背景的后退只能改变Mario相对的坐标，但绝对的坐标是不发生任何变化的，这就很大程度上影响了Mario和一些物体的位置的碰撞的判断。

在这里就需要用到一种暂且称为**镜头移动**的技术，也就是**对surface类的blit方法的参数的调整**

```python
# 先定义好 镜头的位置移动规则 即self.viewpoint
if self.level.mario.pos.x < self.viewpoint.x + 15:
    self.level.mario.pos.x -= self.level.mario.vel.x
if self.level.mario.vel.x > 0:
    if self.level.mario.pos.x > WIDTH * 0.55 + self.viewpoint.x:
        # 1.1 这个系数是为了屏幕的顺滑后期加上去的
        self.viewpoint.x += int(self.level.mario.vel.x * 1.1)
```

~~~python
# self.viewpoint是一个根据Mario的移动而改变参数的矩形
self.screen.blit(self.background, (0, 0), self.viewpoint)
self.all_group.draw(self.screen)
~~~

让我们来看一下上面两端代码的执行效果

![5](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/PLv1cryt9oYlJb5.gif)

很明显背景也移动了，Mario也移动了，但看上去好像是背景移动的太慢了追不上Mario。在调整了Mario的速度后发现问题也不是出在这里。

问题的根源在于：
		我们将背景绘制在屏幕上，再将Mario绘制在屏幕上，这样Mario就是相对于屏幕的速度，那么他是肯定会永远超过屏幕的。这里我们需要做的是**将Mario绘制到背景上，再将背景绘制到屏幕上**

![3](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/qziLdsQtelDJIOY.gif)

那么这就又出现了一个很魔性的效果，问题出现的根源就是**我们每一帧都将Mario的状态绘制到背景上**，每一帧都被我们背景保留了下来。不过这验证了我们之前的想法，至少我们的屏幕滚动跟上了

即然是这样，那我们就每次都弄一个新背景不就好了！一开始我是采用每一次都导入背景，新建并放大到屏幕大小，但这样工作量太大就会产生我们游戏中一个很讨厌的情景：掉帧。那么我们就可以用`pygame.Surface.copy()`这个方法，只在加载游戏的时候加载背景，每一帧只需要对背景进行拷贝一份就可以了

下面就是问题的解决方法

~~~python
 def draw(self):
        pg.display.flip()
        # 每次都将背景拷贝一份，并且每一次都绘制在新的背景上，那么之前的就会被覆盖
        self.background_clean = self.background.copy()
        self.all_group.draw(self.background_clean)
        self.screen.blit(self.background, (0, 0), self.viewpoint)
        self.all_group.draw(self.screen)
~~~

## 5. 项目的重构

在完成了大部分的基础的工作之后，就不得不需要考虑一下整个项目的重新架构了。毕竟这个项目在我们开始着手之后才发现他的逻辑还是比较繁杂的，重新的构架可以帮我们更好的模块解耦，方便以后增加新的板块。

原项目的代码估计得有三四千行吧，构建可以说是很细致，大致划分了十几个文件。我这边也给代码大致分了一下类，理一下整个游戏的流程

- `main.py`：整个游戏的主入口，控制整个游戏的循环
- `sprites.py`：定义整个游戏的所有精灵类及其所有方法
- `level.py`：规范整个关卡，创建所有精灵实体类，规定管道，台阶，砖块等物体的位置，对各种事件的判断
- `settings.py`：规定所有参数，方便调整
- `tools.py`：工具类，定义一些必要的方法，例如图片、声音、背景音乐的载入

## 6. 地面、管道和台阶

虽然在整个背景中存在这三样东西，但我们并没有他们的数据，也就不能进行碰撞检测等操作

这里我们定义了一个类来创建这三个实例对象，获得他们的矩形边框参数

~~~python
class Collider(pg.sprite.Sprite):
    def __init__(self, x, y, width, height):
        pg.sprite.Sprite.__init__(self)
        self.image = pg.Surface((width, height)).convert()
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
~~~

然后在`level.py`中创建他们的实例并加入精灵组

~~~python
    def set_ground(self):
        ground_rect1 = Collider(0, GROUND_HEIGHT, 2953, 60)
			# 其余的省略
        self.ground_group = pg.sprite.Group(ground_rect1,
                                            ground_rect2,
                                            ground_rect3,
                                            ground_rect4)

    def set_pipes(self):
        pipe1 = Collider(1202, 452, 83, 80)
        # 其余的省略
        self.pipe_group = pg.sprite.Group(pipe1, pipe2,
                                          pipe3, pipe4,
                                          pipe5, pipe6)

    def set_steps(self):
        step1 = Collider(5745, 495, 40, 44)
        # 其余的省略
        self.step_group = pg.sprite.Group(step1, step2,
                                          step3, step4,
                                          step5, step6,
                                          step7, step8,
                                          step9, step10,
                                          step11, step12,
                                          step13, step14,
                                          step15, step16,
                                          step17, step18,
                                          step19, step20,
                                          step21, step22,
                                          step23, step24,
                                          step25, step26,
                                          step27)

~~~

## 7. 碰撞的检测和处理

先创建一个方法来对三种精灵对象和Mario的碰撞检测

~~~python
    def check_collide(self):
        self.ground_collide = pg.sprite.spritecollideany(self.mario, self.ground_group)
        self.pipe_collide = pg.sprite.spritecollideany(self.mario, self.pipe_group)
        self.step_collide = pg.sprite.spritecollideany(self.mario, self.step_group)
~~~

然后就是对检测到的碰撞进行处理

这里的处理过程是比较复杂的，所以分为两个方向分别处理

```python
# 处理 x 方向上的碰撞
def adjust_x(self):
    if self.pipe_collide:
        if self.mario.pos.y > self.pipe_collide.rect.y + 10:
            if self.mario.vel.x > 0:
                self.mario.pos.x -= 5
                self.mario.vel.x = 0
            if self.mario.vel.x < 0:
                self.mario.pos.x = 5
                self.mario.vel.x = 0
    if self.step_collide:
        if self.mario.pos.y > self.step_collide.rect.y + 10:
            if self.mario.vel.x > 0:
                self.mario.pos.x -= 5
                self.mario.vel.x = 0
            if self.mario.vel.x < 0:
                self.mario.pos.x = 5
                self.mario.vel.x = 0
# 处理 y 方向上的碰撞
def adjust_y(self):
    if self.ground_collide:
        if self.ground_collide.rect.top < self.mario.pos.y:
            self.mario.acc.y = 0
            self.mario.vel.y = 0
            self.mario.pos.y = self.ground_collide.rect.top
        self.mario.landing = True
    else:
        self.mario.landing = False
    if self.pipe_collide:
        if self.mario.vel.y > 0:
            if self.pipe_collide.rect.top < self.mario.pos.y:
                self.mario.acc.y = 0
                self.mario.vel.y = 0
                self.mario.pos.y = self.pipe_collide.rect.top
            self.mario.landing = True
    if self.step_collide:
        if self.mario.vel.y > 0:
            if self.step_collide.rect.top < self.mario.pos.y:
                self.mario.acc.y = 0
                self.mario.vel.y = 0
                self.mario.pos.y = self.step_collide.rect.top
            self.mario.landing = True
```

碰撞的处理是十分讲究细节的，这里就不过多的赘述，如果没有处理好就会产生很多奇怪的的东西如

![4](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/P2vLlFTexoUzb1Y.gif)

## 8. 最后

到了这里相信大家已经搭建好了整个游戏的框架和基本逻辑，对于砖块、金币、蘑菇和乌龟，还有变大变小和火球效果这边就不赘述了，接着填入框架里就行了。

有了这些基础就可以进一步完善自己的超级玛丽了，你甚至可以对其进行魔改一番，来体验一下上帝视角的快乐

![5](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/i1X4mxqeJhjE5KO.gif)

也可以将其改的十分刁钻，如之前的 猫里奥 游戏

最后可以用`pyinstaller`打成exe文件来分享给你的朋友们

## 9. 代码

这里我贴一下自己的源代码，因为很多细节都在之前提到了，这里就不加注释了

图片我这里贴一张第一关的地图![level](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/eBWZqv8Pco4DLfl.png)

### `main.py`

```python
from level import *
from sprites import *


class Game:
    def __init__(self):
        pg.init()
        self.screen = pg.display.set_mode((WIDTH, HEIGHT))
        self.rect = self.screen.get_rect()
        pg.display.set_caption(TITLE)
        self.clock = pg.time.Clock()
        self.playing = True
        self.all_group = pg.sprite.Group()
        self.viewpoint = self.rect

    def new(self):
        self.level_surface = pg.Surface((WIDTH, HEIGHT)).convert()
        self.background = load_image('level.png')
        self.back_rect = self.background.get_rect()
        self.background = pg.transform.scale(self.background,
                                             (int(self.back_rect.width * BACKGROUND_SIZE),
                                              int(self.back_rect.height * BACKGROUND_SIZE))).convert()
        self.level = Level()
        self.all_group.add(self.level.mario)

    def run(self):
        while self.playing:
            self.clock.tick(FPS)
            self.events()
            self.update()
            self.draw()

    def update(self):
        self.all_group.update()
        self.level.update()
        if self.level.mario.pos.x < self.viewpoint.x + 15:
            self.level.mario.pos.x -= self.level.mario.vel.x
        if self.level.mario.vel.x > 0:
            if self.level.mario.pos.x > WIDTH * 0.55 + self.viewpoint.x:
                self.viewpoint.x += int(self.level.mario.vel.x * 1.1)
        if self.level.mario.dead:
            self.playing = False

    def events(self):
        for event in pg.event.get():
            if event.type == pg.QUIT:
                self.playing = False

    def draw(self):
        pg.display.flip()
        self.background_clean = self.background.copy()
        self.all_group.draw(self.background_clean)
        self.screen.blit(self.background, (0, 0), self.viewpoint)
        self.all_group.draw(self.screen)

    def show_start_screen(self):
        pass

    def show_end_screen(self):
        pass


game = Game()
game.show_start_screen()
game.new()
game.run()
game.show_end_screen()
```

### `sprites.py`

```python
import random
from tools import *
from settings import *

vec = pg.math.Vector2


class Mario(pg.sprite.Sprite):
    def __init__(self):
        pg.sprite.Sprite.__init__(self)
        self.sheet = load_image('mario.png')
        self.load_from_sheet()
        self.walking_timer = pg.time.get_ticks()
        self.image_index = 4
        self.image = self.frames[0]
        self.rect = self.image.get_rect()
        self.pos = vec(WIDTH * 0.5, GROUND_HEIGHT - 70)
        self.vel = vec(0, 0)
        self.acc = vec(0, 0)
        self.landing = False
        self.dead = False

    def update(self):
        self.acc = vec(0, GRAVITY)
        keys = pg.key.get_pressed()
        if keys[pg.K_RIGHT]:
            self.walk('right')
            if self.vel.x > 0:
                self.acc.x = TURNAROUND
            if self.vel.x <= 0:
                self.acc.x = ACC
            self.pos.x += 5
        elif keys[pg.K_LEFT]:
            self.walk('left')
            if self.vel.x < 0:
                self.acc.x = -TURNAROUND
            if self.vel.x >= 0:
                self.acc.x = -ACC
        else:
            self.image_index = 0
        if abs(self.vel.x) < MAX_SPEED:
            self.vel.x += self.acc.x
        elif keys[pg.K_LEFT]:
            self.vel.x = -MAX_SPEED
        elif keys[pg.K_RIGHT]:
            self.vel.x = MAX_SPEED
        if keys[pg.K_SPACE]:
            if self.landing:
                self.vel.y = -JUMP
        if not self.landing:
            self.image_index = 4
        self.image = self.frames[self.image_index]
        self.acc.x += self.vel.x * FRICTION
        self.vel += self.acc
        self.pos += self.vel + 0.5 * self.acc

        self.rect.midbottom = self.pos

    def calculate_animation_speed(self):
        if self.vel.x == 0:
            animation_speed = 130
        elif self.vel.x > 0:
            animation_speed = 130 - (self.vel.x * 12)
        else:
            animation_speed = 130 - (self.vel.x * 12 * -1)
        return animation_speed

    def walk(self, facing):
        if self.image_index == 0:
            self.image_index += 1
            self.walking_timer = pg.time.get_ticks()
        else:
            if (pg.time.get_ticks() - self.walking_timer >
                    self.calculate_animation_speed()):
                self.image_index += 1
                self.walking_timer = pg.time.get_ticks()
        if facing == 'right':
            if self.image_index > 3:
                self.image_index = 0
        if facing == 'left':
            if self.image_index > 8:
                self.image_index = 5
            if self.image_index < 5:
                self.image_index = 5

    def load_from_sheet(self):
        self.right_frames = []
        self.left_frames = []

        self.right_frames.append(
            self.get_image(178, 32, 12, 16))
        self.right_frames.append(
            self.get_image(80, 32, 15, 16))
        self.right_frames.append(
            self.get_image(96, 32, 16, 16))
        self.right_frames.append(
            self.get_image(112, 32, 16, 16))
        self.right_frames.append(
            self.get_image(144, 32, 16, 16))

        for frame in self.right_frames:
            new_image = pg.transform.flip(frame, True, False)
            self.left_frames.append(new_image)

        self.frames = self.right_frames + self.left_frames

    def get_image(self, x, y, width, height):
        image = pg.Surface([width, height])
        rect = image.get_rect()
        image.blit(self.sheet, (0, 0), (x, y, width, height))
        image.set_colorkey(BLACK)
        image = pg.transform.scale(image,
                                   (int(rect.width * MARIO_SIZE),
                                    int(rect.height * MARIO_SIZE)))
        return image


class Collider(pg.sprite.Sprite):
    def __init__(self, x, y, width, height):
        pg.sprite.Sprite.__init__(self)
        self.image = pg.Surface((width, height)).convert()
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
```

### `level.py`

```python
from sprites import *


class Level(pg.sprite.Sprite):
    def __init__(self):
        self.set_mario()
        self.set_ground()
        self.set_pipes()
        self.set_steps()
        self.set_group()

    def set_group(self):
        self.ground_step_pipe_group = pg.sprite.Group(self.ground_group,
                                                      self.pipe_group,
                                                      self.step_group)

    def update(self):
        self.check_collide()
        self.adjust_x()
        self.adjust_y()
        self.check_dead()
        print(self.mario.pos)

    def set_mario(self):
        self.mario = Mario()

    def set_ground(self):
        ground_rect1 = Collider(0, GROUND_HEIGHT, 2953, 60)
        ground_rect2 = Collider(3048, GROUND_HEIGHT, 635, 60)
        ground_rect3 = Collider(3819, GROUND_HEIGHT, 2735, 60)
        ground_rect4 = Collider(6647, GROUND_HEIGHT, 2300, 60)

        self.ground_group = pg.sprite.Group(ground_rect1,
                                            ground_rect2,
                                            ground_rect3,
                                            ground_rect4)

    def set_pipes(self):
        pipe1 = Collider(1202, 452, 83, 80)
        pipe2 = Collider(1631, 409, 83, 140)
        pipe3 = Collider(1973, 366, 83, 170)
        pipe4 = Collider(2445, 366, 83, 170)
        pipe5 = Collider(6989, 452, 83, 82)
        pipe6 = Collider(7675, 452, 83, 82)

        self.pipe_group = pg.sprite.Group(pipe1, pipe2,
                                          pipe3, pipe4,
                                          pipe5, pipe6)

    def set_steps(self):
        step1 = Collider(5745, 495, 40, 44)
        step2 = Collider(5788, 452, 40, 88)
        step3 = Collider(5831, 409, 40, 132)
        step4 = Collider(5874, 366, 40, 176)

        step5 = Collider(6001, 366, 40, 176)
        step6 = Collider(6044, 408, 40, 40)
        step7 = Collider(6087, 452, 40, 40)
        step8 = Collider(6130, 495, 40, 40)

        step9 = Collider(6345, 495, 40, 40)
        step10 = Collider(6388, 452, 40, 40)
        step11 = Collider(6431, 409, 40, 40)
        step12 = Collider(6474, 366, 40, 40)
        step13 = Collider(6517, 366, 40, 176)

        step14 = Collider(6644, 366, 40, 176)
        step15 = Collider(6687, 408, 40, 40)
        step16 = Collider(6728, 452, 40, 40)
        step17 = Collider(6771, 495, 40, 40)

        step18 = Collider(7760, 495, 40, 40)
        step19 = Collider(7803, 452, 40, 40)
        step20 = Collider(7845, 409, 40, 40)
        step21 = Collider(7888, 366, 40, 40)
        step22 = Collider(7931, 323, 40, 40)
        step23 = Collider(7974, 280, 40, 40)
        step24 = Collider(8017, 237, 40, 40)
        step25 = Collider(8060, 194, 40, 40)
        step26 = Collider(8103, 194, 40, 360)

        step27 = Collider(8488, 495, 40, 40)

        self.step_group = pg.sprite.Group(step1, step2,
                                          step3, step4,
                                          step5, step6,
                                          step7, step8,
                                          step9, step10,
                                          step11, step12,
                                          step13, step14,
                                          step15, step16,
                                          step17, step18,
                                          step19, step20,
                                          step21, step22,
                                          step23, step24,
                                          step25, step26,
                                          step27)

    def check_collide(self):
        self.ground_collide = pg.sprite.spritecollideany(self.mario, self.ground_group)
        self.pipe_collide = pg.sprite.spritecollideany(self.mario, self.pipe_group)
        self.step_collide = pg.sprite.spritecollideany(self.mario, self.step_group)

    def adjust_x(self):
        if self.pipe_collide:
            if self.mario.pos.y > self.pipe_collide.rect.y + 10:
                if self.mario.vel.x > 0:
                    self.mario.pos.x -= 5
                    self.mario.vel.x = 0
                if self.mario.vel.x < 0:
                    self.mario.pos.x = 5
                    self.mario.vel.x = 0
        if self.step_collide:
            if self.mario.pos.y > self.step_collide.rect.y + 10:
                if self.mario.vel.x > 0:
                    self.mario.pos.x -= 5
                    self.mario.vel.x = 0
                if self.mario.vel.x < 0:
                    self.mario.pos.x = 5
                    self.mario.vel.x = 0

    def adjust_y(self):
        if self.ground_collide:
            if self.ground_collide.rect.top < self.mario.pos.y:
                self.mario.acc.y = 0
                self.mario.vel.y = 0
                self.mario.pos.y = self.ground_collide.rect.top
            self.mario.landing = True
        else:
            self.mario.landing = False
        if self.pipe_collide:
            if self.mario.vel.y > 0:
                if self.pipe_collide.rect.top < self.mario.pos.y:
                    self.mario.acc.y = 0
                    self.mario.vel.y = 0
                    self.mario.pos.y = self.pipe_collide.rect.top
                self.mario.landing = True
        if self.step_collide:
            if self.mario.vel.y > 0:
                if self.step_collide.rect.top < self.mario.pos.y:
                    self.mario.acc.y = 0
                    self.mario.vel.y = 0
                    self.mario.pos.y = self.step_collide.rect.top
                self.mario.landing = True

    def check_dead(self):
        if self.mario.pos.y > GROUND_HEIGHT + 50:
            self.mario.dead = True
```

### `tools.py`

```python
import os
import pygame as pg


def load_image(filename):
    src = os.path.dirname(os.path.abspath(__file__))
    path = os.path.join(src, 'resources', 'graphics', filename)
    return pg.image.load(path)
```

### `settings.py`

```python
# 标题和窗口大小
TITLE = 'Mario'
WIDTH = 800
HEIGHT = 600

FPS = 60

# 定义颜色
GRAY = (100, 100, 100)
BLACK = (0, 0, 0)

# 图片缩放比例
MARIO_SIZE = 2.5
BACKGROUND_SIZE = 2.679

# Mario 运动系数
ACC = 0.3
GRAVITY = 1
FRICTION = -0.12
JUMP = 20
TURNAROUND = 0.7
MAX_SPEED = 6

# 地面高度
GROUND_HEIGHT = HEIGHT - 66
```