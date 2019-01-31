---
title: Scrapy框架入门
date: 2018-07-18 17:44:35
categories: 爬虫
tags:
- scrapy
---

### 1. 准备工作

安装Scrapy框架、MongoDB和PyMongo库，如果没有安装，google了解一下~~

### 2. 创建项目

使用命令创建Scrapy项目，命令如下：

`scrapy startproject tutorial`

该命令可以在任意文件夹运行，如果提示权限问题，可以加**sudo**运行。该命令会创建一个名为tutorial的文件夹，结构如下：

![](s1.png)

- scrapy.cfg： Scrapy项目的配置文件，定义了项目的配置文件路径，部署相关信息等
- item.py： 定义item数据结构（爬取的数据结构）
- pipeline.py： 定义数据管道
- settings.py： 配置文件
- middlewares.py： 定义爬取时的中间件
- spiders： 放置Spiders的文件夹

### 3. 创建Spider

Spider是自己定义的类，Scrapy用它来从网页抓取内容，并解析抓取结果。该类必须继承Scrapy提供的Spider类scrapy.Spider。

使用命令创建一个Spider，命令如下：

```python
cd tutorial
scrapy genspider quotes quotes.toscrape.com
```

首先，进入刚才创建的tutorial文件夹，然后执行**genspider**命令。第一个参数是spider的名称，第二个参数是网络域名（要抓取网络的域名）。执行完毕后，spiders文件夹中多了一个quotes.py，它就是刚刚创建的Spider，内容如下：

![](s2.png)

该类中有三个属性 ------ name、allowed_domains、start_urls，一个方法parse。

- name，唯一的名字，用来区分不同的Spider。
- allowed_domains，允许爬取的域名，如果初始或后续的请求链接不是该域名下的，则被过滤掉。
- start_urls，Spider在启动时爬取的url列表，用来定义初始的请求。
- parse，它是spider的一个方法，用来处理start_urls里面的请求返回的响应，该方法负责解析返回的响应，提取数据或进一步生成处理的请求。

### 4. 创建Item

Item是保存爬取数据的容器，使用方法和字典类似。创建Item需要继承scrapy.Item类，并且定义类型为scrapy.Field的字段。

定义Item，将生成的items.py修改如下：

![](s3.png)

这里定义了三个字段，接下来爬取时我们会用到这个Item。

### 5. 解析Response

前面我们看到，parse()方法的参数response是start_urls里面的链接爬取后的结果，所以在parse方法中，可以对response变量包含的内容进行解析。网页结构如下：

![](s4.png)

提取方式可以是CSS选择器或XPath选择器。在这里，使用CSS选择器，parse()方法修改如下：

```python
def parse(self, response):
    quotes = response.css('.quote')
    for quote in quotes:
        text = quote.css('.text::text').extract_first()
        author = quote.css('.author::text').extract_first()
        tag = quote.css('.tags .tag::text').extract()
```

首先，利用选择器选取所有的quote，并将其赋值给quotes变量，然后利用for循环对每一个quote遍历，解析每一个quote的内容。

对text来说，他的class是text，所以用**.text**选择器来选取，这个结果实际上是整个带有标签的节点，要获取它的正文内容，可以加**::text**来获取。这时的结果是长度为1的列表，所以还需要用**extract_first()**方法来获取第一个元素。

### 6. 使用Item

上面定义了Item，这边我们就需要用到它。Item可以理解为一个字典，不过在这里需要先实例化，然后将解析的结果赋值给Item的每一个字段，最后返回Item。

修改QuotesSpider类如下：

![](s5.png)

至此，首页的所有内容被解析出来了，并将结果赋值给一个个TutorialItem。

### 7. 后续Request

上面实现了网页首页的抓取解析，那么下一页怎么抓取呢？我们可以看到网页的翻页结构如下：

![](s6.png)

![](s7.png)

这里有一个Next按钮，查看源码，可以看出下一页的全链接是：http://quotes.toscrape.com/page/2/,通过这个链接我们就可以构造下一个请求。

构造请求需要用到**scrapy.Request**。这里会有两个参数 -------url和callback。

- url，请求链接。
- callback，回调函数。请求完毕后，获取响应，引擎会将该响应作为参数传递给回调函数，回调函数进行解析或生成下一个请求。

在parse()方法中追加如下代码：

```python
next_page = response.css('.pager .next a::attr("href")').extract_first()
url = response.urljoin(next_page)
yield scrapy.Request(url=url, callback=self.parse)
```

第一句，获取下一个页面的链接，即要获取a超链接中的href属性。

第二句，调用**urljoin()**方法，urljoin()方法可以将相对URL构造成一个绝对URL。例如，获取得到下一页的地址是/page/2/，urljoin()方法处理后的结果是：http://quotes.toscrape.com/page/2/。

第三句，通过url和callback变量构造了一个新的请求，回调函数callback依然使用parse()方法。这样，爬虫就进入了一个循环，直到最后一页。

修改之后，整个Spider类如下：

```python
# -*- coding: utf-8 -*-
import scrapy
from tutorial.items import TutorialItem


class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        quotes = response.css('.quote')
        for quote in quotes:
            item = TutorialItem()
            item['text'] = quote.css('.text::text').extract_first()
            item['author'] = quote.css('.author::text').extract_first()
            item['tag'] = quote.css('.tags .tag::text').extract()
            yield item

        next_page = response.css('.pager .next a::attr("href")').extract_first()
        url = response.urljoin(next_page)
        yield scrapy.Request(url=url, callback=self.parse)
```

### 8. 运行

进入目录，运行如下命令：

`scrapy crawl quotes`

就可以看到Scrapy的运行结果了。

### 9. 保存文件

运行完Scrapy后，我们只在控制台看到了输出结果。如何保存结果呢？

Scrapy提供了Feed Exports可以轻松将结果输出。例如，我们想将上面的结果保存成JSON文件，可以执行如下命令：

`scrapy crawl quotes -o quotes.json`

命令运行后，会发现项目内多了一个quotes.json文件，这个文件包含了抓取的所有内容，格式为JSON。

另外，还支持其他格式如下：

```python
scrapy crawl quotes -o quotes.jsonlines   (scrapy crawl quotes -o quotes.jl ,  jl是jsonlines的缩写)
scrapy crawl quotes -o quotes.csv
scrapy crawl quotes -o quotes.xml
scrapy crawl quotes -o quotes.pickle
scrapy crawl quotes -o quotes.marshal
scrapy crawl quotes -o ftp://user:pass@ftp.example.com/path/to/quotes.csv    (远程输出，需要正确配置，否则会报错)
```

### 10. 使用Pipeline

如果想进行复杂额操作，如将结果保存到MongoDB数据库，或者筛选Item，我们可以定义Pipeline来实现。

前面提到，Pipeline是项目管道，当Item生成后，它会自动被送到Pipeline进行处理，主要的操作如下：

- 清理HTML数据
- 验证爬取的数据，检查爬取的字段
- 查重并丢弃重复内容
- 将结果保存到数据库

实现Pipeline，只需要定义一个类并实现process_item()方法即可。启用Pipeline后，Pipline会自动调用这个方法。process_item()方法必须返回包含数据的字典或item对象，或者抛出DropItem异常。

process_item()方法有两个参数，一个参数是item，每次Spider生成的Item都会作为参数传递过来，另一个参数是spider，就是Spider的实例。

接下来，我们实现一个Pipline，筛掉text长度大于50的Item，并将结果保存到MongoDB数据库。

修改pipelines.py如下：

```python
# -*- coding: utf-8 -*-
from scrapy.exceptions import DropItem
import pymongo


class TutorialPipeline(object):
    def __init__(self):
        self.limit = 50

    def process_item(self, item, spider):
        if item['text']:
            if len(item['text']) > self.limit:
                item['text'] = item['text'][0:self.limit].rstrip() + '...'
            return item
        else:
            return DropItem('Missing text')


class MongoPipeline(object):
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DB')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def process_item(self, item, spider):
        name = item.__class__.__name__
        self.db[name].insert(dict(item))
        return item

    def close_spider(self, spider):
        self.client.close()
```

- from_crawler，这是一个类方法，用@classmethod标识，是一种依赖注入。它的参数就是crawler，通过crawler可以拿到全局配置的每一个配置信息。在全局配置settings.py中，可以配置MONGO_UR和MONGO_DB来指定MongoDB连接需要的地址和数据库名称，拿到配置信息之后返回类对象即可。所以这个方法主要是用来获取settings.py中的配置信息。
- open_spider，当Spider开启时，这个方法被调用。进行初始化操作。
- close_spider，当Spider关闭时，这个方法被调用。将数据库连接关闭。

最主要的process_item()方法则进行了数据插入操作。

定义好的TutorialPipeline和MongoPipline这两个类后，我们需要在settings.py中使用它们，MongoDB的连接信息也需要在settings.py中定义。

settings.py中加入如下内容：

```python
ITEM_PIPELINES = {
    'tutorial.pipelines.TutorialPipeline': 300,
    'tutorial.pipelines.MongoPipeline': 400,
}
MONGO_URI = 'localhost'
MONGO_DB = 'tutorial'
```

赋值ITEM_PIPELINES字典，键名是Pipeline的类名称，键值是调用的优先级，是一个数字，数字越小对应的Pipeline越先被调用。

重新执行如下命令进行爬取：

`scrapy crawl quotes`

结束后，MongoDB中会创建了一个tutorial的数据库、TutorialItem的表，如下图：

![](s8.png)

### 11. 结语

至此，一个简单的Scrapy框架爬虫就完成了，这只是一个简单的爬虫例子，Github上面也有许多相关的项目可以去研究~~~