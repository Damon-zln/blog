---
title: python3 使用pip安装（命令行中）失败或 “not a supported wheel” 解决方案！
date: 2018-07-17 18:04:18
categories: Python3笔记
tags: 
- python3
---

### python3 使用pip安装（命令行中）失败或 “not a supported wheel” 解决方案！

- 原因1：安装的不是对应python版本的库，下载的库名中cp36代表python3.6,其它同理。

- 原因2：（我遇到的情况----下载的是对应版本的库，然后仍然提示不支持当前平台）

  百度了一下，说法如下：

  在shell中输入import pip; print(pip.pep425tags.get_supported())可以获取到pip支持的文件名还有版本

  然而，很悲催。。。。出现了

  ![img](1.png)

  然后，google了一下，解决了~

  因为我的pip的版本是V10，应该使用下面的命令：

  import pip._internal; print(pip._internal.pep425tags.get_supported())

  ![img](2.png)

  嘿嘿~~

