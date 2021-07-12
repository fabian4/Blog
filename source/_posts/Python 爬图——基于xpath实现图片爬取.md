---
title: Python 爬图——基于xpath实现图片爬取
date: 2020/6/4
description: Python 爬图——基于xpath实现图片爬取
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/D7H4KGUmRqxvcSr.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/D7H4KGUmRqxvcSr.jpg'
categories:
  - python
tags:
  - 爬虫
abbrlink: 10071
---



# Python 爬图——基于xpath实现图片爬取

## 写在前面

前段时间在网上冲浪的时候发现了一个有意思的网站：[**`http://turnoff.us/`**](http://turnoff.us/)，是一个极客漫画网站，他的创始人也是一个软件工程师，其漫画内容也别具一格，自嘲暗讽毒舌样样不少。这里贴出来几张：

![the depressed developer 53](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/KGVke6pAaT4vmRQ.png)

![the depressed developer 47](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/7o35TZ6FdYXtalm.png)



![123](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/gP1uoLbZxkpKcAX.png)

![what your code looks like](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/dDRpqIvsEtOLeSQ.png)

**不过由于服务器可能在国外的原因，访问效果很是受限！既然如此那我们就让python来替我们去做吧**

## 环境准备

- 由于我们使用python来编写小脚本，所以这里我们就不使用我们的重型武器`pycharm`了，我们选用轻便小巧的`Subline Text`来作为我们的工具

  这里需要简单配置一下你的`Subline text`的编译环境来更加方便的运行 Python

  ![image-20200604153444094](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/TuKRAi5ph3jwkSY.png)

  写入配置文件并保存到默认目录就行了

  ~~~json
  {
  	"cmd": ["python","-u","$file"],
      "selector": "source.pythona",
      "encoding": "utf-8" ,
      "env": {"PYTHONIOENCODING": "utf8"}
  }
  ~~~

  然后你的`Build System`的选项卡就有python3了

  同时也可以快捷键`Ctrl+B`快速运行

- 依赖导入

  这里我们需要一个基于`xpath`的第三方库—— **lxml**

  ~~~bash
  pip install lxml
  ~~~

- 关于 **xpath**

  > **XPath 是一门在 XML 文档中查找信息的语言。XPath 可用来在 XML 文档中对元素和属性进行遍历。**
  >
  > **XPath 是 W3C XSLT 标准的主要元素，并且 XQuery 和 XPointer 都构建于 XPath 表达之上。**

## 连接测试

这里我们依赖于一个`request`库来发起请求

~~~python
import request # 导入依赖

res = requests.get("http://turnoff.us/")
# 向这个地址发起get请求
print(res.text)
# 确认一下是否返回了网页的html文件
~~~

通过`lxml`来解析页面元素

~~~python
from lxml import etree # 导入依赖

html = etree.HTML(res.text)
# 将文本传入解析
target = html.xpath("//article[@class='post-content']/p/img/@src")
# 通过xpath表达式来检索出我们所想要的元素,返回一个链表
print(target[0].encode('utf-8').decode())
# 将链表第一个元素编码解码输出
~~~

关于xpath的表达式语法这里就不赘述，但这里给大家提供两种方法

- 谷歌插件 [下载地址](https://chrome.zzzmh.cn/#index) (火狐也有对应插件)

  `Ctrl+Shift+x `   调出窗口

  `Ctrl+shift+鼠标左键`   选择元素

  ![image-20200604162400829](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/utEmh97SMFc1Jr3.png)

  

- 谷歌自带调试工具

  ![image-20200604162631362](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/HuTs1yLadOV4cAB.png)

## 开始爬图

~~~python
import requests
import os
import random
from lxml import etree

# 准备一些请求头来对我们的爬虫进行伪装
request_headers = [
    "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36",
    "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_2) AppleWebKit/537.75.14 (KHTML, like Gecko) Version/7.0.3 Safari/537.75.14",
    "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; Win64; x64; Trident/6.0)",
    'Mozilla/5.0 (Windows; U; Windows NT 5.1; it; rv:1.8.1.11) Gecko/20071127 Firefox/2.0.0.11',
    'Opera/9.25 (Windows NT 5.1; U; en)',
    'Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)',
    'Mozilla/5.0 (compatible; Konqueror/3.5; Linux) KHTML/3.5.5 (like Gecko) (Kubuntu)',
    'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.8.0.12) Gecko/20070731 Ubuntu/dapper-security Firefox/1.5.0.12',
    'Lynx/2.8.5rel.1 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/1.2.9',
    "Mozilla/5.0 (X11; Linux i686) AppleWebKit/535.7 (KHTML, like Gecko) Ubuntu/11.04 Chromium/16.0.912.77 Chrome/16.0.912.77 Safari/535.7",
    "Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:10.0) Gecko/20100101 Firefox/10.0",
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36'
]

root = "http://turnoff.us/"
# 请求根路径

def main():
	a = 0
    # 定义循环变量
	root_source = "http://turnoff.us/"
	while a > 0 :
        # 写一个死循环一直爬就是了
		res = requests.get(root_source, headers={'User-Agent': random.choice(request_headers)})
		html = etree.HTML(res.text)
		target = html.xpath("//article[@class='post-content']/p/img/@src")[0].encode('utf-8').decode()[1:]
        # 爬图片地址
		img_name = html.xpath("//h1[@class='post-title']/text()")[0].encode('utf-8').decode()[2:]
        # 爬图片名字
		next = html.xpath("//div[@class='prev']/a[1]/@href")[0].encode('utf-8').decode()[1:]
        # 爬取下一网页的地址
		target = root+target
        # url拼接
		res_img = requests.get(target, headers={'User-Agent': random.choice(request_headers)})
        # 获取图片二进制数据
		if not os.path.exists('./turnoff.us'):
			os.mkdir('./turnoff.us')
            # 创建文件夹
		img_path = 'turnoff.us/' + img_name + '.png'
        # 定义文件路径
		fp = open(img_path,'wb')
		fp.write(res_img.content)
        # 写入操作
		fp.close
		print("<= 第 "+ str(a) +" 张图片： "+img_name+" 下载完成 ---- 图片来源： "+target+"==>")
        # 控制台反馈
		root_source = root + next
        # 拼接下一个页面url
		a += 1



if __name__ == '__main__':
	main()
~~~

然后我们就可以去泡杯茶慢慢等就行了

![image-20200604163321066](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/ErhzSHktQ6WJLnu.png)

>果然懒还是第一生产力