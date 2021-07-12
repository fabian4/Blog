---
title: pygame快速上手
date: 2020/4/27
description: 开始你的第一个 python 小游戏——pygame快速上手
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/FmqHTOS6Nz4b9jo.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/FmqHTOS6Nz4b9jo.jpg'
categories:
  - python
tags:
  - python
  - pygame
abbrlink: 12739
---

# 开始你的第一个 python 小游戏——pygame快速上手

## pygame

​		pygame作为一款优秀的python第三方库，包含了很多游戏方面的函数。今天我们就开始来快速上手做一个自己的小游戏。这里是 [pygame官网](https://www.pygame.org/) 和 [官方文档](https://www.pygame.org/docs/)，下面是游戏的效果图

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/ZmMVSek4cqFJ2zw.gif" alt="jumpy.gif" style="zoom:25%;" />

- 在使用 pygame 的同时，可以帮我们更好的熟悉python的语法，为后面的爬虫、web和数据分析学习做铺垫
- 这个游戏中涉及的一些基本的角色移动跳跃和碰撞，也是大家开始编写自己的小游戏的基础
- pyinstaller 也是一个在打包脚本，跨平台使用的便捷方式
- 游戏的素材和代码来自于油管，大家一起共同学习和借鉴

## 一、游戏的基本逻辑和游戏循环

下面是一个游戏的最基本的流程

- 游戏开始
- 游戏循环
  1. 监听鼠标键盘事件
  2. 更新元素的状态
  3. 绘制屏幕
- 游戏结束

而根据游戏的基本流程，下面就是一个很常用的模板，根据模板来将这个这个游戏的流程更加具象

~~~python
import pygame

# 定义窗口大小和FPS
WIDTH = 360
HEIGHT = 400
FPS = 30

# 定义颜色
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)

# 初始化pygame
pygame.init()
# 初始化声音组件
pygame.mixier.init()
# 根据长宽创建窗口
screen = pygame.display.set_mode([WIDTH, HEIGHT])
# 窗口展示游戏的名字
pygame.display.set_caption("My Game")
clock = pygame.time.Clock()

all_sprites = pygame.sprite.Group()
# 游戏循环
running = True
while running:
	# 保持一定速度
	clock.tick(FPS)
	# 事件输入
	for event in pygame.event.get():
		# 检查是否关闭窗口
		if event.type == pygame.QUIT:
			running = False
	# 更新状态
	all_sprites.update()

	# 绘制窗口
	screen.fill(BLACK)
	all_sprites.draw(screen)
	pygame.display.flip()

pygame.quit()

~~~

## 二、模块划分和框架的构建

有了上面的模板我们就可以开始我们的游戏创作了，然而随着我们游戏的逻辑的复杂，代码全部挤在一个文件中很明显就不利于我们的后期改进和调整，在这里我们将其拆分为大致三个文件：

### main.py    用于控制整个游戏循环和游戏的开始和结束

~~~python
import random
from sprites import *

class Game:
    def __init__(self):
        # 初始化窗口
        # 初始化pygame创建窗口
        pg.init()
        self.screen = pg.display.set_mode([WIDTH, HEIGHT])
        pg.display.set_caption(TITLE)
        self.clock = pg.time.Clock()
    def new(self):
        # 创建游戏所需的对象
    def run(self):
        # 游戏循环
        self.playing = True
        while self.playing:
            self.clock.tick(FPS)
            self.events()
            self.update()
            self.draw()
    def update(self):
        # 更新状态
    def events(self):
        # 检测响应事件
        for event in pg.event.get():
            # 检查是否关闭窗口
            if event.type == pg.QUIT:
                if self.playing:
                    self.playing = False
                self.running = False
   def draw(self):
    	# 绘制窗口
        self.screen.fill(BACKGROUND_COLOR)
        self.all_sprites.draw(self.screen)
        pg.display.flip()
    def show_start_screen(self):
        # 展示开始屏幕
    def show_go_screen(self):
        # 展示结束屏幕
        
g = Game()
g.show_start_screen()
while g.running:
    g.new()
    g.show_go_screen()
~~~

### sprite.py   用于我们创建角色，敌人之类的游戏元素

~~~python
from settings import *
import pygame as pg

class Player(pg.sprite.Sprite):
    # 角色类
class Platform(pg.sprite.Sprite):
    # 阶梯类
~~~

### settings.py  用于来专门放置我们游戏中的参数，起到一个配置类的作用

~~~python
# 定义游戏名字
TITLE = "Jumpy !"
# 定义窗口大小和FPS
WIDTH = 400
HEIGHT = 600
FPS = 60
# 定义颜色
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
BACKGROUND_COLOR = (0, 155, 155)
# 阶梯的位置
PLATFORM_LIST = [(WIDTH / 2 - 30, HEIGHT * 0.8, 150, 20),
                 (25, HEIGHT - 350, 100, 20),
                 (350, 200, 100, 20),
                 (175, 100, 50, 20)]
~~~

## 三、角色、阶梯的创建

​		在pygame中有一个很重要的类就是`sprite`类，可以说是所有元素类的父类，为每一种元素创建一个类继承该类，并将所有创建的元素注册到`sprite`的组件中。

### 角色类

~~~python
class Player(pg.sprite.Sprite):
    def __init__(self, game):
        pg.sprite.Sprite.__init__(self)
        self.image = pg.Surface((30, 40))
        self.image.fill(YELLOW) 
        # 角色为一个黄色的方块
        self.rect = self.image.get_rect()
        self.rect.center = (WIDTH / 2, HEIGHT / 2)
        self.pos = vec(WIDTH / 2, HEIGHT / 2)
        # 角色的初始位置

    def update(self):
~~~

### 阶梯类

~~~python
class Platform(pg.sprite.Sprite):
    def __init__(self, x, y, w, h):
        pg.sprite.Sprite.__init__(self)
        self.image = pg.Surface((w, h))
        self.image.fill(GREEN)
        # 填充绿色
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
~~~

### 加入`sprite`组件

~~~python
def new(self):
        self.all_sprites = pg.sprite.Group()
        self.platforms = pg.sprite.Group()
        self.player = Player(self)
        self.all_sprites.add(self.player)
        self.ground = Platform(0, HEIGHT - 20, WIDTH, 20)
        self.all_sprites.add(self.ground)
        self.platforms.add(self.ground)
        for plat in PLATFORM_LIST:
            p = Platform(*plat)
            self.all_sprites.add(p)
            self.platforms.add(p)
~~~

## 四、定义碰撞和重力效果

### 坐标系

在这之前要先简单的说明一下pygame中对坐标的定义，即顶点在游戏窗口左上角的直角坐标系。他的正方向分别是向右和向下的。所以比如当角色要向上跳跃的时候**参数应该是负的**

<img src=https://fabian.oss-cn-hangzhou.aliyuncs.com/img/tiExBwSvADjqcQY.png style="zoom: 33%;" />

### 加入重力

~~~python
# 在角色类中定义
def update(self):
    	# 加入重力效果
        self.acc = vec(0, PLAYER_GRAVITY)
~~~

但是运行游戏之后你会发现小方块会不停的下落直接穿过了我们定义的"地面"

### 定义碰撞

~~~python
# 在整个游戏类中定义
def update(self):
        self.all_sprites.update()
        # 确保只有角色是在向下落的时候撞到平台才能站在平台上
        if self.player.vel.y > 0:
            hits = pg.sprite.spritecollide(self.player, self.platforms, False)
            if hits:
                if self.player.pos.y < hit[0].rect.bottom:
                    self.player.pos.y = hit[0].rect.top
                    self.player.vel.y = 0
                self.player.jumping = False
~~~

碰撞类是pygame中另一个比较重要的函数，角色的很多操作和状态都会涉及到

## 五、角色的左右移动和跳跃

### 移动

在我们对角色进行左右移动的操作时，绝不能仅仅的只是角色的速度向左和向右的增加。而应该对他的加速度参数进行调整，并且加上一定的阻力，这样角色才不会一直运动下去，确保速度存在一个峰值。这其中就涉及到了运动学的公式，毕竟这个公式是可以约束地球上所有的物体的，不然整个游戏给人的感觉就会有失违和。

~~~python
def update(self):
    	# 加入重力效果
        self.acc = vec(0, PLAYER_GRAVITY)
        keys = pg.key.get_pressed()
        if keys[pg.K_LEFT]:
            self.acc.x = -PLAYER_ACC
        if keys[pg.K_RIGHT]:
            self.acc.x = PLAYER_ACC
        # 增加摩擦
        self.acc.x += self.vel.x * PLAYER_FRICTION
        # 移动公式
        self.vel += self.acc
        self.pos += self.vel + 0.5 * self.acc
        # 让角色保持在屏幕里
        if self.pos.x > WIDTH:
            self.pos.x = 0
        if self.pos.x < 0:
            self.pos.x = WIDTH
~~~

### 跳跃

对于角色的跳跃操作我们也需要加以一些公式的约束从而使其达到一个合理的地球表面的跳跃效果

~~~python
 def jump(self):
        # 引入一个角色是否在跳跃状态的判断，防止出现角色可以在空中一直跳跃类似起飞的效果
        if not self.jumping:
            self.jumping = True
            self.vel.y = -PLAYER_JUMP
~~~

## 六、屏幕的滚动

- 正如前面一开始的gif中所展示的样子，随着小方块的不断的上移，我们整个游戏的所覆盖的画面也应该向上移动。当由于屏幕是不可能如我们所要求的样子向上滚动的，所以我们采用**让背景相对向下移动**来达到同样的效果
- 随着屏幕的不断上移我们就要不断的来产生新的台阶来让我们这个游戏继续下去，这里用了随机生成的方式
- 当越来越多的台阶的产生无疑对我们真的游戏的运行来说是一种负担，尤其是那些已经滚动到屏幕下方的台阶，虽然已经不在我们的视野里面，但依然存在我们的内存里。这里我们就需要清楚哪些已经没用的台阶

~~~python
# 定义在游戏类中
def update(self):
        # 向上滚动屏幕即背景下移
        if self.player.rect.top <= HEIGHT * 0.4:
            self.player.pos.y += abs(int(self.player.vel.y))
            for plat in self.platforms:
                plat.rect.y += abs(int(self.player.vel.y))
                if plat.rect.top > HEIGHT:
                    plat.kill()
                    # 当有台阶在屏幕的下方被清除就加分
                    self.score += 10
        # 产生新平面
        while len(self.platforms) < 6:
            width = random.randrange(50, 100)
            p = Platform(random.randrange(0, WIDTH - width),
                         random.randrange(-75, -30),
                         width, 20)
            self.platforms.add(p)
            self.all_sprites.add(p)
~~~

## 七、细节的处理和完善

### 长按和短按空格达成不一样的跳跃效果

这里需要对键盘的按下和松开即`KEYDOWN`和`KEYUP`进行监控，同时引入跳跃的状态来控制

~~~python
# 定义在游戏类中
def events(self):
    if event.type == pg.KEYDOWN:
        if event.key == pg.K_SPACE:
            self.player.jump()
    if event.type == pg.KEYUP:
        if event.key == pg.K_SPACE:
            self.player.jump_cut()
~~~

~~~python
# 定义在角色类中
def jump(self):
        if not self.jumping:
            self.jumping = True
            self.vel.y = -PLAYER_JUMP

def jump_cut(self):
        if self.jumping:
            if self.vel.y < -3:
                self.vel.y = -3
~~~

### 加入一个角色死亡效果

~~~python
# 定义在游戏类中
def update(self):
       # 当角色落入屏幕下方时，就回滚还剩下的屏幕下方的台阶
        if self.player.rect.bottom > HEIGHT:
            for sprite in self.all_sprites:
                sprite.rect.y -= int(max(self.player.vel.y, 10))
                if sprite.rect.bottom < 0:
                    sprite.kill()
        if len(self.platforms) == 0:
            self.playing = False
~~~

### 开始和结束屏幕的绘制

~~~python
# 定义在游戏类中
def show_start_screen(self):
    	# 绘制开始屏幕
        self.screen.fill(BACKGROUND_COLOR)
        self.draw_text(TITLE, 48, WHITE, WIDTH*0.5, HEIGHT*0.25)
        self.draw_text("Arrows to move, Space to jump", 22, WHITE, WIDTH*0.5, HEIGHT*0.5)
        self.draw_text("Press a key to play", 22, WHITE, WIDTH * 0.5, HEIGHT * 0.75)
        pg.display.flip()
        self.wait_for_key()

 def show_go_screen(self):
    	# 绘制结束屏幕
        self.screen.fill(BACKGROUND_COLOR)
        self.draw_text("Congratulations !", 48, WHITE, WIDTH * 0.5, HEIGHT * 0.25)
        self.draw_text("Score: " + str(self.score), 22, WHITE, WIDTH * 0.5, HEIGHT * 0.5)
        self.draw_text("Press a key to play again", 22, WHITE, WIDTH * 0.5, HEIGHT * 0.75)
        pg.display.flip()
        self.wait_for_key()

 def wait_for_key(self):
    	# 等待玩家确认
        self.waiting = True
        while self.waiting:
            self.clock.tick(FPS)
            for event in pg.event.get():
                if event.type == pg.QUIT:
                    self.waiting = False
                    self.running = False
                if event.type == pg.KEYDOWN:
                    self.waiting = False

 def draw_text(self, text, size, color, x, y):
    	# 屏幕上绘制文字
        font = pg.font.SysFont("arial", size)
        text_surface = font.render(text, True, color)
        text_rect = text_surface.get_rect()
        text_rect.midtop = (int(x), int(y))
        self.screen.blit(text_surface, text_rect)
~~~

## 八、程序打包

​		当你编写完一个小程序肯定迫切的想分享给朋友们去试一下，但不是大家的电脑上都有python环境，这里就需要我们来使用**`pyinstaller`**来对我们的程序进行打包

1. 使用`pip`安装 `pyinstaller`（如果安装速度太慢可以配置一下镜像）

   ~~~shell
   pip install pyinstaller
   ~~~

2. 进入程序所处目录开始打包

   ~~~shell
   pyinstaller -F -w main.py
   ~~~

   - `-F`：只生产一个exe文件
   - `-w`：不创建cmd命令窗口，直接进入游戏窗口
   
3. 然后你就可以去 disc 目录下找到你的exe文件分享给你的朋友们了

## 九、源码

  ### main.py

  ~~~python
  import random
  from sprites import *
  
  class Game:
      def __init__(self):
          # 初始化窗口
          # 初始化pygame创建窗口
          pg.init()
          # pygame.mixier.init()
          self.screen = pg.display.set_mode([WIDTH, HEIGHT])
          pg.display.set_caption(TITLE)
          self.clock = pg.time.Clock()
          self.running = True
  
      def new(self):
          self.score = 0
          self.all_sprites = pg.sprite.Group()
          self.platforms = pg.sprite.Group()
          self.player = Player(self)
          self.all_sprites.add(self.player)
          self.ground = Platform(0, HEIGHT - 20, WIDTH, 20)
          self.all_sprites.add(self.ground)
          self.platforms.add(self.ground)
          for plat in PLATFORM_LIST:
              p = Platform(*plat)
              self.all_sprites.add(p)
              self.platforms.add(p)
          self.run()
  
      def run(self):
          # 游戏循环
          self.playing = True
          while self.playing:
              self.clock.tick(FPS)
              self.events()
              self.update()
              self.draw()
  
      def update(self):
          self.all_sprites.update()
          if self.player.vel.y > 0:
              hits = pg.sprite.spritecollide(self.player, self.platforms, False)
              if hits:
                  lowest = hits[0]
                  for hit in hits:
                      if hit.rect.bottom > lowest.rect.centery:
                          lowest = hit
                  if self.player.pos.y < lowest.rect.bottom:
                      self.player.pos.y = lowest.rect.top
                      self.player.vel.y = 0
                  self.player.jumping = False
  
          # 向上滚动屏幕即背景下移
          if self.player.rect.top <= HEIGHT * 0.4:
              self.player.pos.y += abs(int(self.player.vel.y))
              for plat in self.platforms:
                  plat.rect.y += abs(int(self.player.vel.y))
                  if plat.rect.top > HEIGHT:
                      plat.kill()
                      self.score += 10
  
          if self.player.rect.bottom > HEIGHT:
              for sprite in self.all_sprites:
                  sprite.rect.y -= int(max(self.player.vel.y, 10))
                  if sprite.rect.bottom < 0:
                      sprite.kill()
          if len(self.platforms) == 0:
              self.playing = False
  
          # 产生新平面
          while len(self.platforms) < 6:
              width = random.randrange(50, 100)
              p = Platform(random.randrange(0, WIDTH - width),
                           random.randrange(-75, -30),
                           width, 20)
              self.platforms.add(p)
              self.all_sprites.add(p)
  
      def events(self):
          for event in pg.event.get():
              # 检查是否关闭窗口
              if event.type == pg.QUIT:
                  if self.playing:
                      self.playing = False
                  self.running = False
              if event.type == pg.KEYDOWN:
                  if event.key == pg.K_SPACE:
                      self.player.jump()
              if event.type == pg.KEYUP:
                  if event.key == pg.K_SPACE:
                      self.player.jump_cut()
  
      def draw(self):
          self.screen.fill(BACKGROUND_COLOR)
          self.all_sprites.draw(self.screen)
          self.draw_text(str(self.score)+"  score", 22, WHITE, WIDTH / 2, 25)
          pg.display.flip()
  
      def show_start_screen(self):
          self.screen.fill(BACKGROUND_COLOR)
          self.draw_text(TITLE, 48, WHITE, WIDTH*0.5, HEIGHT*0.25)
          self.draw_text("Arrows to move, Space to jump", 22, WHITE, WIDTH*0.5, HEIGHT*0.5)
          self.draw_text("Press a key to play", 22, WHITE, WIDTH * 0.5, HEIGHT * 0.75)
          pg.display.flip()
          self.wait_for_key()
  
      def show_go_screen(self):
          self.screen.fill(BACKGROUND_COLOR)
          self.draw_text("Congratulations !", 48, WHITE, WIDTH * 0.5, HEIGHT * 0.25)
          self.draw_text("Score: " + str(self.score), 22, WHITE, WIDTH * 0.5, HEIGHT * 0.5)
          self.draw_text("Press a key to play again", 22, WHITE, WIDTH * 0.5, HEIGHT * 0.75)
          pg.display.flip()
          self.wait_for_key()
  
      def wait_for_key(self):
          self.waiting = True
          while self.waiting:
              self.clock.tick(FPS)
              for event in pg.event.get():
                  if event.type == pg.QUIT:
                      self.waiting = False
                      self.running = False
                  if event.type == pg.KEYDOWN:
                      self.waiting = False
  
      def draw_text(self, text, size, color, x, y):
          font = pg.font.SysFont("arial", size)
          text_surface = font.render(text, True, color)
          text_rect = text_surface.get_rect()
          text_rect.midtop = (int(x), int(y))
          self.screen.blit(text_surface, text_rect)
  
  
  g = Game()
  g.show_start_screen()
  while g.running:
      g.new()
      g.show_go_screen()
  
  ~~~

  

  ### sprites.py

  ~~~python
  from settings import *
  import pygame as pg
  
  vec = pg.math.Vector2
  
  class Player(pg.sprite.Sprite):
      def __init__(self, game):
          pg.sprite.Sprite.__init__(self)
          self.game = game
          self.jumping = False
          self.image = pg.Surface((30, 40))
          self.image.fill(YELLOW)
          self.rect = self.image.get_rect()
          self.rect.center = (WIDTH / 2, HEIGHT / 2)
          self.pos = vec(WIDTH / 2, HEIGHT / 2)
          self.vel = vec(0, 0)
          self.acc = vec(0, 0)
  
      def jump(self):
          if not self.jumping:
              self.jumping = True
              self.vel.y = -PLAYER_JUMP
  
      def jump_cut(self):
          if self.jumping:
              if self.vel.y < -3:
                  self.vel.y = -3
  
      def update(self):
          self.acc = vec(0, PLAYER_GRAVITY)
          keys = pg.key.get_pressed()
          if keys[pg.K_LEFT]:
              self.acc.x = -PLAYER_ACC
          if keys[pg.K_RIGHT]:
              self.acc.x = PLAYER_ACC
  
          # 增加摩擦
          self.acc.x += self.vel.x * PLAYER_FRICTION
          # 移动公式
          self.vel += self.acc
          self.pos += self.vel + 0.5 * self.acc
  
          if self.pos.x > WIDTH:
              self.pos.x = 0
          if self.pos.x < 0:
              self.pos.x = WIDTH
  
          self.rect.midbottom = self.pos
  
  
  class Platform(pg.sprite.Sprite):
      def __init__(self, x, y, w, h):
          pg.sprite.Sprite.__init__(self)
          self.image = pg.Surface((w, h))
          self.image.fill(GREEN)
          self.rect = self.image.get_rect()
          self.rect.x = x
          self.rect.y = y
  
  ~~~

  ### settings.py

  ~~~python
  TITLE = "Jumpy !"
  # 定义窗口大小
  WIDTH = 400
  HEIGHT = 600
  FPS = 60
  
  PLAYER_ACC = 0.5
  PLAYER_FRICTION = -0.12
  PLAYER_GRAVITY = 0.8
  PLAYER_JUMP = 20
  
  # 定义颜色
  WHITE = (255, 255, 255)
  BLACK = (0, 0, 0)
  RED = (255, 0, 0)
  GREEN = (0, 255, 0)
  BLUE = (0, 0, 255)
  YELLOW = (255, 255, 0)
  BACKGROUND_COLOR = (0, 155, 155)
  
  # 平面
  PLATFORM_LIST = [(WIDTH / 2 - 30, HEIGHT * 0.8, 150, 20),
                   (25, HEIGHT - 350, 100, 20),
                   (350, 200, 100, 20),
                   (175, 100, 50, 20)]
  ~~~

 