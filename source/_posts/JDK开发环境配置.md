---
title: JDK开发环境配置
date: 2018-10-15 14:57:02
categories: Java笔记
tags: jdk
---

### JDK开发环境配置

#### 1. 新建 -> 变量名“**JAVA_HOME**”, 变量值“**C:\Program Files\Java\jdk1.8.0_112**”（即JDK的安装路径）

#### 2. 编辑 -> 变量名“**Path**”, 在原变量值的最后加上“**;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin**”

#### 3. 新建 -> 变量名“**CLASSPATH**”, 变量值“**.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar**”

#### 4. 测试：

- 在CMD中分别输入`java,javac,java -version`命令，出现JDK的编译信息，包括修改命令的语法和参数选项的信息，即环境配置成功。

