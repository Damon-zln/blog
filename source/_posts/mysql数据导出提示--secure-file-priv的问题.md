---
title: mysql数据导出提示--secure-file-priv的问题
date: 2018-10-19 10:55:16
categories: Mysql笔记
tags: mysql
---

### Mysql数据导出提示--secure-file-priv的问题

- mysql可使用`into outfile`参数把表中的数据到处到csv，示例如下：

  ```mysql
  select user_id from weibo_comment into outfile '/home/dazhan/comment.csv' lines terminated by '\n';
  ```

  执行后，weibo_comment表中的user_id字段就会被导出`/home/dazhan/comment.csv`

- 参数说明：

  - `into outfile`导出的目录和文件名

    指定导出的目录和文件名

  - `fields terminated by`字段间分隔符

    定义字段间的分隔符

  - `optionally enclosed by`字段包围符

    定义包围字段的字符（数值型字段无效）

  - `lines terminated by`行间分隔符

    定义每行的分隔符
<!--more-->
- 问题分析：

  - 以上实例在实际的运行中出现如下错误：

    ```mysql
    mysql> select user_id from weibo_comment into outfile '/home/dazhan/comment.txt' lines terminated by '\n';
    ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
    ```

  - 查看官方文档，`--secure-file-priv`参数用于LOAD DATA, SELECT …OUTFILE, LOAD_FILE()传到哪个指定目录。

    - `secure-file-priv`为`NULL`时，表示限制mysqld不允许导入或导出。
    - `secure-file-priv`为`/var/lib/mysql-files/`时，表示限制mysqld只能在`/var/lib/mysql-files/`目录中执行导入导出，其他目录不行。
    - `secure-file-priv`没有值时，表示不限制mysqld在任意目录的导入导出。

  - 查看`secure-file-priv`的值（默认值是NULL）的命令如下：

    ```mysql
    mysql> show global variables like '%secure_file_priv%';
    +------------------+-----------------------+
    | Variable_name    | Value                 |
    +------------------+-----------------------+
    | secure_file_priv | /var/lib/mysql-files/ |
    +------------------+-----------------------+
    1 row in set (0.01 sec)
    ```

    这是我的查询结果，所以，我只需要把上述的输出目录改为`/var/lib/mysql-files/`就可以进行数据导出。

- 修改`secure-file-priv`参数值的方法：

  - 打开`my.cnf或my.ini`，加入如下语句后重启mysql服务。

    ```mysql
    secure_file_priv=''
    ```

  - 查看修改后的值：

    ```mysql
    mysql> show global variables like '%secure_file_priv%';
    +------------------+-------+
    | Variable_name    | Value |
    +------------------+-------+
    | secure_file_priv |       |
    +------------------+-------+
    1 row in set (0.01 sec)
    ```

    这样修改后，就可以在任意目录进行数据导入导出了。

