---
title: Scrapy+Scrapy-redis+Scrapyd+Gerapy分布式爬虫框架整合
date: 2018-09-07 17:44:35
categories: 爬虫
tags:
- scrapy
---

### 前言

首先，介绍一下这个标题吧。

- Scrapy：是一个基于Twisted的异步IO框架，有了这个框架，我们就不需要等待当前URL抓取完毕之后在进行下一个URL的抓取，抓取效率可以提高很多。
- Scrapy-redis：虽然Scrapy框架是异步加多线程的，由于我们只能在一台主机上运行，爬取效率还是有限的，Scrapy-redis库为我们提供了Scrapy分布式的队列，调度器，去重等等功能，有了它，我们就可以将多台主机组合起来，共同完成一个爬取任务，抓取的效率又提高了。
- Scrapyd：分布式爬虫完成之后，接下来就是代码部署，如果我们有很多主机，那就要逐个登录服务器进行部署，万一代码有所改动..........可以想象，这个过程是多么繁琐。Scrapyd是专门用来进行分布式部署的工具，它提供HTTP接口来帮助我们部署，启动，停止，删除爬虫程序，利用它我们可以很方便的完成Scrapy爬虫项目的部署。
- Gerapy：是一个基于Scrapyd，Scrapyd API，Django，Vue.js搭建的分布式爬虫管理框架。简单点说，就是用上述的Scrapyd工具是在命令行进行操作，而Gerapy将命令行和图形界面进行了对接，我们只需要点击按钮就可完成部署，启动，停止，删除的操作。
<!--more-->
### 创建Scrapy项目（参考Scrapy框架入门）

- 项目结构如下（这个项目里包含了多个spider，接下来，就以我圈出来的为例讲解）

  ![](ss1.png)

  因为我这个需要用到固定的代理，在这个讲一下代理如何使用：

  代理在middlewares.py这个模块中的process_request方法中进行配置（如果想知道为什么在这里配置，可以去google一下scrapy框架爬虫的整体结构流程），如下：

  ```python
  class ProxyMiddleware(object):
      def __init__(self, proxy_url):
          self.proxy_url = proxy_url
  
      def process_request(self, request, spider):
          request.meta['proxy'] = self.proxy_url
  
      @classmethod
      def from_crawler(cls, crawler):
          return cls(
              proxy_url=crawler.settings.get('PROXY_URL')
          )
  ```

  然后在settings.py中指定具体的代理是什么,如下：

  ```python
  PROXY_URL = 'http://10.51.1.140:8080'
  ```

  如果要设置动态代理，参考这里：<https://github.com/Damon-zln/ProxyPool>

### Scrapy-redis分布式配置

1. 首先，确认Scrapy-redis库已经安装，如果未安装，可以 **pip install scrapy-redis** 进行安装。

2. 搭建Redis服务器，如果未安装redis数据库的，请自行google。

   Redis安装完成后，就可以远程连接了，如果不能，可能是服务器上没有开放端口（redis默认端口6379）

   记录服务器的IP，端口（默认是6379），密码为foobared（安装的时候自己设置的密码），供后面分布式爬虫使用。

3. 配置Scrapy-redis（只需要修改settings.py文件即可）

   将调度器的类和去重的类替换为Scrapy-redis提供的类，在settings.py中配置如下：

   ```python
   SCHEDULER = 'scrapy_redis.scheduler.Scheduler'
   DUPEFILTER_CLASS = 'scrapy_redis.dupefilter.RFPDupeFilter'
   ```

   Redis连接配置：

   ```python
   REDIS_URL = 'redis://:foobared@10.255.1.175:6379'
   ```

   其他的都使用默认配置（如：调度对列，持久化，重爬，管道）

4. 配置存储目标：

   搭建一个MongoDB服务，将多台主机的数据都存在同一个MongoDB数据库中

   配置如下：

   ```python
   MONGO_URI = '10.255.1.175'
   MONGO_DB = 'webscrape'
   ```

   综上，分布式完整配置如下：

   ```python
   MONGO_URI = '10.255.1.175'
   MONGO_DB = 'webscrape'
   PROXY_URL = 'http://10.51.1.140:8080'
   SCHEDULER = 'scrapy_redis.scheduler.Scheduler'
   DUPEFILTER_CLASS = 'scrapy_redis.dupefilter.RFPDupeFilter'
   REDIS_URL = 'redis://:foobared@10.255.1.175:6379'
   ```

### Scrapyd安装

1. 安装：`pip install scrapyd`

2. 配置：安装完毕后，需要新建一个配置文件/etc/scrapyd/scrapyd.conf，scrapyd在运行时会读取此配置文件

   在Scrapyd1.2版本后，不会自动创建该文件，需要我们自行添加。

   首先，执行如下命令新建文件：

   ```shell
   sudo mkdir /etc/scrapyd
   sudo vi /etc/scrapyd/scrapyd.conf
   ```

   然后在文件中写入如下内容：

   ```shell
   [scrapyd]
   eggs_dir    = eggs
   logs_dir    = logs
   items_dir   =
   jobs_to_keep = 5
   dbs_dir     = dbs
   max_proc    = 0
   max_proc_per_cpu = 4
   finished_to_keep = 100
   poll_interval = 5.0
   bind_address = 127.0.0.1
   http_port   = 6800
   debug       = off
   runner      = scrapyd.runner
   application = scrapyd.app.application
   launcher    = scrapyd.launcher.Launcher
   webroot     = scrapyd.website.Root
   
   [services]
   schedule.json     = scrapyd.webservice.Schedule
   cancel.json       = scrapyd.webservice.Cancel
   addversion.json   = scrapyd.webservice.AddVersion
   listprojects.json = scrapyd.webservice.ListProjects
   listversions.json = scrapyd.webservice.ListVersions
   listspiders.json  = scrapyd.webservice.ListSpiders
   delproject.json   = scrapyd.webservice.DeleteProject
   delversion.json   = scrapyd.webservice.DeleteVersion
   listjobs.json     = scrapyd.webservice.ListJobs
   daemonstatus.json = scrapyd.webservice.DaemonStatus
   ```

   该内容参见官方文档：<https://scrapyd.readthedocs.io/en/stable/config.html#example-configuration-file>

   这里的配置文件有所修改：max_proc_per_cpu官方默认是4，即一台主机每个CPU最多运行4个Scrapy任务，在此提高为10，另外，bind_address，默认为本地127.0.0.1，在此修改为0.0.0.0,这样外网可以访问。

3. 后台运行scrapyd

   ```python
   scrapyd > /dev/null &
   ```

   运行之后，便可在浏览器的6800端口访问WEB UI了，如下：

   ![](ss2.png)

### Gerapy分布式管理

1. 安装：`pip install gerapy`

2. 使用说明：

   利用gerapy命令创建一个项目

   ```shell
   gerapy init
   ```

   在当前目录下生成一个gerapy文件夹，进入gerapy文件夹，会发现一个空的projects文件夹，后面后用到.

   初始化：

   ```
   gerapy migrate
   ```

   这样会生成一个SQLite数据库，用于保存各个主机的配置信息等.

   启动Gerapy服务:

   ```
   gerapy runserver host:port (默认端口是8000)
   ```

   这样，我们就可以通过*http://host:8000*进入Gerapy管理页面

3. 在主机管理中添加安装了Scrapyd环境的机器（机器IP和Scrapyd运行的端口，默认是6800），如下图：

   ![](ss3.png)

4. 上面创建gerapy项目的时候，会生成一个projects文件夹，这个文件夹就是用来存放你的Scrapy项目的（直接将整个项目拷贝到这个文件夹下即可）

   ![](ss4.png)

5. 可以点击上图中的编辑，在线编辑项目，如果项目没有问题，可以点击部署进行打包和部署，在部署之前要打包项目（打包成一个egg文件），可以部署到多台主机，如下所示：

   ![](ss5.png)

6. 部署完毕后，可以回到主机管理页面进行任务调度。点击调度即可进入任务管理页面，可以查看当前主机所有任务的运行状态。

   ![](ss6.png)

   通过点击运行，停止按钮来实现任务的启动和停止，同时可以展开任务条目查看日志详情。

7. 下面是抓取的数据存入MongoDB数据库。

   ![](ss7.png)

### Docker部署

在讲docker部署之前，先简单总结一下上面我们做了什么事吧。

- 创建Scrapy项目：里面存放的就是抓取数据的一些逻辑
- Scrapy-redis分布式配置：这是是在Scrapy项目中的settings.py文件中配置的，作用就是让Scrapy支持分布式
- Scrapyd安装：分布式抓取时，分布节点（node机器）所需要的环境
- Gerapy分布式管理：这个就是类似于我们webscraping的管理平台

我们这里的docker部署，是指将我们的分布节点的scrapyd环境，使用docker来安装，具体如下：

1. 首先在每台节点上安装docker（最新的稳定的版本）
2. 执行docker命令完成环境安装：`docker run --name scrapyd -p 6800:6800 -d damon7096/scrapyd:latest`



