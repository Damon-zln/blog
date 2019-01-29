---
title: shell脚本中遇到的问题及解决方案
date: 2018-10-19 12:01:18
categories: Linux笔记
tags:
- shell
- linux
---

### 1. `||`运算符

- 问题场景：shell脚本中定义了一个方法，方法里面有一个for循环，依次请求10个服务器并将结果保存到文件。试想，如果其中一个服务器的地址写错了，shell脚本执行到这里就会中断，不会继续往下执行了。

- 解决方案：

  - 第一种方法，就是使用if判断，这个在这里不细说。

  - 第二种方法是使用`||`运算符。

    - command1 `||` command2，如果command1未执行成功，那么就执行command2

    - 这样，我们在循环中请求服务器的代码就可以写成如下形式：

      ```shell
      curl --max-time 120 --user ${user}:${pwd} -s -k --request POST --header "Content-type: text/xml; charset=utf-8" --data @${requestFile} ${url} | iconv -t utf8 | xmllint --format - > ${responseFile} || true
      ```

      只需要保证`||` 右边的command2为真就可以了，当然你也可以写成`i=1` 这种为真的等式，但是不推荐这么做，因为这样在别人看代码的时候，会造成误解。

### 2. 关于getopts的使用方法

- 例子：

  ```shell
  while getopts ":e:s:pd:l" arg; 
  do
      case $arg in
      e)
          ghs_env=$OPTARG
          ;;
      s)
          ghs_service=$OPTARG
          ;;
      p)
          proxy=`sed '/^PROXY=/!d;s/.*=//' config.properties`
          ;;
      d)
          dir_path=$OPTARG
          ;;
      l)
          if [ -d "log" ]; then
                  rm -r log
                  mkdir log
          else
                  mkdir log
          fi
          log_dir=$(cd `dirname $0`; pwd)/log
          ;;
      ?)
          echo "Usage: $(basename $0) [-e somevalue] [-s somevalue] [-p somevalue] [-d somevalue]" 1>&2
          exit 1
          ;;
          esac
  done
  ```

- getopts有两个参数，第一个参数是一个字符串，包括字符和" **:** ", 每一个字符都是一个有效的选项，如果字符后面带有" **:** ", 表示这个字符有自己的参数。getopts从命令中获取这些参数，并且删去了" **-** ", 并将其赋值在第二个参数中，即"**OPTARG**"中，在例子中，**$OPTARG**存储相应选项的参数。

- `while getopts ":e:s:pd:l" arg;` 这行代码中，第一个冒号表示忽略系统报错信息，使用自定义的报错信息；字符后面的冒号表示该选项必须自己的参数。

  - 第一个冒号存在时（自定义报错信息）：

    1. 当指定的参数不存在时，variable设置为" **:** ", 对应的**$OPTARG**为此时的选项  （这个尚未搞明白是什么意思）
    2. 当指定的选项是带参数的而没有提供参数或是非法选项（指定的选项没有定义），variable设置为" **?** ", 对应的**$OPTARG**为此时的选项

  - 第一个冒号不存在时（会按照系统的定义报错）：

    1. 指定了非法选项（不存在的选项或者说是没有定义的选项），会报错：`scriptname:illegal option — 选项`
    2. 选项需要参数但没有指定，会报错：`scriptname: option requires an argument –选项`

    ---

  - 单个字符后面接一个冒号，表示选项必须自己的参数，参数可以紧跟选项后或者以空格隔开，该参数的指针赋给**OPTARG**。

  - 单个字符后面接两个冒号表示该选项必须自己的参数且参数紧跟选项后不能以空格隔开（我试过，感觉有没有空格都可以，因此，我觉得如果要加参数，字符后只要写一个冒号就可以了），该参数的指针赋给**OPTARG**。