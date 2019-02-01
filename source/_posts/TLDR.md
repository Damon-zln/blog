---
title: 快速查看linux命令常用用法---------tldr
date: 2018-07-05 15:13:25
categories: Linux笔记
tags:
- linux
---

### 前言

之前我们如果用一个命令，但是忘了具体的参数是什么的时候，通常会用man，比如：

`man tar`

这样，man就会告诉我们**tar**的具体用法，但是man有时候特别的冗长，你要找到想要的例子非常困难，所以tldr命令就是一个很好的补充，tldr会告诉你一些**tar**的常用例子和用法。

### 安装tldr

安装tldr特别简单，具体命令如下：

`sudo curl -o /usr/local/bin/tldr https://raw.githubusercontent.com/raylee/tldr/master/tldr && sudo chmod +x /usr/local/bin/tldr`

这样，tldr就安装完成了，下面我们来使用tldr查看find命令的常用用法：

```shell
[dazhan@ecs47.101.138.203 ~]$ tldr find
find

Find files or directories under the given directory tree, recursively.

- Find files by extension:
  find root_path -name '*.ext'

- Find files by matching multiple patterns:
  find root_path -name '*pattern_1*' -or -name '*pattern_2*'

- Find directories matching a given name:
  find root_path -type d -name *lib*

- Find files matching path pattern:
  find root_path -path '**/lib/**/*.ext'

- Run a command for each file, use {} within the command to access the filename:
  find root_path -name '*.ext' -exec wc -l {} \;

- Find files modified in the last 24-hour period:
  find root_path -mtime -1

- Find files using case insensitive name matching, of a certain size:
  find root_path -size +500k -size -10M -iname '*.TaR.gZ'

- Delete files by name, older than 180 days:
  find root_path -name '*.ext' -mtime +180 -delete

- Find files matching a given pattern, while excluding specific paths:
  find root_path -name '*.py' -not -path '*/site-packages/*'

```



