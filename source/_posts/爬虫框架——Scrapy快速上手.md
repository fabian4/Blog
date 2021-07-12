---
title: 爬虫框架——Scrapy快速上手
date: 2020/6/14
description: 爬虫框架——Scrapy快速上手
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/QKkrGgWpRlzFATx.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/QKkrGgWpRlzFATx.jpg'
categories:
  - python
tags:
  - 爬虫
  - Scrapy
abbrlink: 1097
---

# 爬虫框架——Scrapy快速上手

## 一、Scrapy 简介

>Scrapy 框架
>		Scrapy是用纯Python实现一个为了爬取网站数据、提取结构性数据而编写的应用框架，用途非常广泛。框架的力量，用户只需要定制开发几个模块就可以轻松的实现一个爬虫，用来抓取网页内容以及各种图片，非常之方便。Scrapy 使用了 Twisted 异步网络框架来处理网络通讯，可以加快我们的下载速度，不用自己去实现异步框架，并且包含了各种中间件接口，可以灵活的完成各种需求。

- 虽然我们可以使用独立的 python 脚本来编写我们的爬虫，但框架的使用无疑会大大**简化我们的爬虫开发**以及**加快我们的爬取速度**。我们也可以在在框架中自定义组件来控制和伪装我们的爬虫，以达到反反爬的目的。

我们这里以一个图片网站为例进行爬取 -------> https://www.mzitu.com/ (由于一些原因我就不把爬取的结果图贴上去了🤐)

## 二、安装依赖创建项目

~~~shell
# 下载安装依赖(如果速度太慢可以配置镜像)
pip install Scrapy
# 创建爬虫项目
scrapy startproject MeiziTu(你的爬虫项目名称)
# 进入文件目录
cd xxxxxx
# 新建爬虫文件
scrapy genspider meizitu(你的爬虫名称) mzitu.com(待爬取的域名)
# 启动爬虫
scrapy crawl meizitu(你的爬虫名称)
~~~

这次因为是一个爬虫工程，我们用 **Pycharm** 打开，然后你就可以看见以下目录文件

![image-20200614195326900](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/3msfunzHy7jCT6k.png)

- `spiders`：放置你的爬虫脚本的文件夹
  - `_init_.py`：包的声明文件
  - `meizitu.py`：主要的爬虫脚本文件，你的大部分业务代码在这里编写
- `_init_.py`：包的声明文件
- `items.py`：资源封装文件
- `middlewares.py`：爬虫中间件文件
- `pipelines.py`：管道处理文件
- `settings.py`：配置文件
- `scrapy.cfg`：爬虫的主配置文件

## 三、Scrapy 框架架构

### Scrapy 架构图

![image-20200614200344091](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/6cl7CsITXOnb1Ao.png)

### Scrapy 流程图

![image-20200614200428620](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/6EhAkxmZ8BUiqVM.png)

### Scrapy框架模块功能

- `Scrapy Engine`（引擎）：Scrapy框架的核心部分。负责在Spider和ItemPipeline、Downloader、Scheduler中间通信、传递数据等。
- `Spider`（爬虫）：发送需要爬取的链接给引擎，最后引擎把其他模块请求回来的数据再发送给爬虫，爬虫就去解析想要的数据。这个部分是我们开发者自己写的，因为要爬取哪些链接，页面中的哪些数据是我们需要的，都是由程序员自己决定。
- `Scheduler`（调度器）：负责接收引擎发送过来的请求，并按照一定的方式进行排列和整理，负责调度请求的顺序等。
- `Downloader`（下载器）：负责接收引擎传过来的下载请求，然后去网络上下载对应的数据再交还给引擎。
- `Item Pipeline`（管道）：负责将Spider（爬虫）传递过来的数据进行保存。具体保存在哪里，应该看开发者自己的需求。
- `Downloader Middlewares`（下载中间件）：可以扩展下载器和引擎之间通信功能的中间件。
- `Spider Middlewares`（Spider中间件）：可以扩展引擎和爬虫之间通信功能的中间件。

## 四、爬虫文件

### 1. 默认生成的爬虫文件

```python
import scrapy

class MeizituSpider(scrapy.Spider):
    # 继承于Spider类
    name = 'meizitu'
    # 爬虫名字
    allowed_domains = ['mzitu.com']
    # 爬虫允许爬取的访问
    start_urls = ['http://mzitu.com/']
    # 爬虫的开始路径

    def parse(self, response):
        # 爬取的回调函数
        pass
```

### 2. 两个常用的爬虫类

- `scrapy.Spider`：scrapy.Spider是所有爬虫类的父类。
- `scrapy.spiders.CrawlSpider`：规则爬虫是省略了一般的解析工作，它完成了感兴趣的连接`<a href="">文本</a>`提取。按rules中给定的Rule规则进行提取连接标签中的信息（href, text）。

### 3. 开始解析结果

~~~python
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from MeiziTu.items import MeizituItem


class MeizituSpider(CrawlSpider):
    # 这里由于页面特点我们将爬虫继承规则爬虫类
    name = 'meizitu'
    allowed_domains = ['mzitu.com']
    start_urls = ['https://www.mzitu.com/']
    rules = (
        # 定义爬虫规则
        Rule(
            LinkExtractor(allow=r'https://www.mzitu.com/page/\d+/'),
            # 链接选择器 采用正则表达式进行匹配
            follow=True
            # 跟进链接
        ),
        Rule(
            LinkExtractor(allow=r'https://www.mzitu.com/\d+'),
            callback='parse_detail',
            # 指明回调函数来处理爬取结果
            follow=False
            # 不继续跟进链接
        )
    )

    def parse_detail(self, response):
        # 处理结果的回调函数
        title = response.xpath("//h2[@class='main-title']/text()").get()
        # xpath语法提取结果
        urls = []
        # 准备空列表封装爬取到的图片链接
        amount = response.xpath("//div[@class='pagenavi']//span/text()").getall()[-2]
        # 找到每一套的数量
        base_url = response.xpath("//div[@class='main-image']/p/a/img/@src").get()
        # 找到每一套图的url规则
        for i in range(1, int(amount) + 1):
            # 对url进行按规律的拼接
            if i < 10:
                url = base_url.replace("1.jpg", str(i) + ".jpg")
            else:
                url = base_url.replace("01.jpg", str(i) + ".jpg")
            urls.append(url)
            # 加入集合
        item = MeizituItem(title=title, urls=urls)
        # 封装到 item
        yield item
        # 传到下载组件

~~~

>结果的解析这里可以使用 **正则表达式** 、 **xpath表达式** 和 **css选择器** 这些的具体语法这里不过多赘述
>
>然后根据浏览器的调试工具来抓取自己所需要的信息

### 4. 爬虫配置

~~~python
# 关闭对爬虫协议的遵守
ROBOTSTXT_OBEY = False

# 开启延时，降低爬取频率，避免给对方服务器造成太大压力，并且防止频率太高被识别为爬虫
DOWNLOAD_DELAY = 1

# 配置随机请求头进行伪装 越多越好
USER_AGENT_LIST = [
    "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 (KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",
    ...
]
USER_AGENT = random.choice(USER_AGENT_LIST)
DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
    'User-Agent': USER_AGENT
}

# 有必要的话也可以在这里配置代理ip
~~~

### 5. 爬虫调试

- 爬虫启动：因为scrapy是从命令行启动，这边为了方便调试，我们通过 python 脚本启动

  ~~~python
  # 目录下新建一个start.py文件
  from scrapy import cmdline
  
  # 启动命令行并输入命令
  cmdline.execute('scrapy crawl meizitu'.split())
  
  # 这样我们就可以通过运行这个脚本来启动爬虫
  ~~~

- 爬虫测试：由于爬虫的框架在调试时频繁启动很麻烦，我们这里可以使用 scrapy 提供的命令行工具进行测试

  ~~~shell
  scrapy shell http://xxxxxxx.com(你的目标url)
  # 接下来就可以开始测试你的正则表达式或者xpath语法
  ~~~

## 五、资源封装下载

### 1. 封装

- 我们在`item.py`中定义我们需要传递的资源

  ~~~python
  import scrapy
  
  class MeizituItem(scrapy.Item):
      title = scrapy.Field()
      urls = scrapy.Field()
  ~~~

- 然后就可以在爬虫文件中将其传递过来

  ~~~python
  from MeiziTu.items import MeizituItem
  
  def prase(self, response):
      ...    
      item = MeizituItem(title=title, urls=urls)
      # 封装到 item
      yield item
      # 传到下载组件
  ~~~

- 在管道文件中接收

  ~~~python
  class MeizituPipeline:
      def process_item(self, item, spider):
          title = item["title"]
          urls = item["urls"]
  ~~~

### 2. 文件下载

接下来我们就可以在管道文件中定义下载过程了

~~~python
import os
import random
import requests

# 配置随机请求头进行伪装 越多越好
request_headers = [
    "Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:10.0) Gecko/20100101 Firefox/10.0",
    ...
]


class MeizituPipeline:
    def process_item(self, item, spider):
        title = item["title"]
        urls = item["urls"]
        for url in urls:
            # 处理每一个图片的资源链接
            os.chdir("D:/pic")
            # 改变工作路径
            file_name = url.split(".com/")[1].replace("/", "-")
            # 提取文件名称
            image = requests.get(url,
                                 headers={'User-Agent': random.choice(request_headers),
                                          'referer': 'https://www.mzitu.com/'})
            # 在请求头中添加 referer 伪装爬虫
            f = open(file_name, 'wb')
            f.write(image.content)
            f.close()
~~~

> 然后就可以开始享受成果了！

