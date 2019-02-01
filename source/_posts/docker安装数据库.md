---
title: Docker安装数据库
date: 2019-01-24 15:13:25
categories: Docker笔记
tags:
- docker
- mysql
- redis
---

### 1. docker安装mysql5.7

- 查看镜像

  `docker search mysql`

- 拉取镜像

  `docker pull mysql:5.7`
<!--more-->
- 运行镜像

  `docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7`

  - 参数说明：

    ```
    -d 标识是让 docker 容器在后台运行。
    -p 标识通知 Docker 将容器内部使用的网络端口映射到我们使用的主机上。
    –name 定义一个容器的名字，如果在执行docker run时没有指定Name，那么deamon会自动生成一个随机数字符串当做UUID。
    -e 设置环境变量，或者覆盖已存在的环境变量
    3306:3306 将容器的3306端口映射到本机的3306端口 (左边是宿主机端口，右边是容器内端口)
    ```

### 2. docker安装redis

- 查看镜像

  `docker search redis`

- 拉取镜像

  `docker pull redis`

- 运行镜像

  `docker run --name redis -p 6379:6379 -d --restart=always redis:latest redis-server --appendonly yes --requirepass "foobared"`

  - 参数说明：

    ```
    -p 6379:6379 :将容器内端口映射到宿主机端口（左边是宿主机端口，右边是容器内端口）
    redis-server –appendonly yes : 在容器执行redis-server启动命令，并打开redis持久化配置 
    --requirepass “your passwd” :设置认证密码 
    –-restart=always : 随docker启动而启动
    --name redis ：指定容器的名字
    -d : 后台运行
    ```




