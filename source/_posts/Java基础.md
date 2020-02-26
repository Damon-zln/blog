---
title: Java基础
date: 2020-02-26 14:12:22
categories: Java笔记
tags: 
- java
---

### JDK 和 JRE 有什么区别？

- JDK是Java Development Kit，它是功能齐全的Java SDK。它拥有JRE所拥有的一切，还有编译器（javac）和工具（如javadoc和jdb）。它能够创建和编译程序。
- JRE 是 Java运行时环境。它是运行已编译 Java 程序所需的所有内容的集合，包括 Java虚拟机（JVM），Java类库，java命令和其他的一些基础构件。但是，它不能用于创建新程序。

<!--more-->

### == 和 equals 的区别是什么？

- **==** : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。

- **equals()** : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

  - 情况1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
  - 情况2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

  举例：

  ```java
  public class test1 {
      public static void main(String[] args) {
          String a = new String("ab"); // a 为一个引用
          String b = new String("ab"); // b为另一个引用,对象的内容一样
          String aa = "ab"; // 放在常量池中
          String bb = "ab"; // 从常量池中查找
          if (aa == bb) // true
              System.out.println("aa==bb");
          if (a == b) // false，非同一对象
              System.out.println("a==b");
          if (a.equals(b)) // true
              System.out.println("aEQb");
          if (42 == 42.0) { // true
              System.out.println("true");
          }
      }
  }
  ```

  说明：

  - String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
  - 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。

### hashCode 和 equals

- hashCode介绍

  hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数。

  散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）

- 为什么要有 hashCode

  **我们先以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：** 当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 `equals()`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的Java启蒙书《Head first java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

  通过我们可以看出：`hashCode()` 的作用就是**获取哈希码**，也称为散列码；它实际上是返回一个int整数。这个**哈希码的作用**是确定该对象在哈希表中的索引位置。**`hashCode() `在散列表中才有用，在其它情况下没用**。在散列表中hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

- hashCode() 与 equals() 的相关规定

  1. 如果两个对象相等，则hashcode一定也是相同的
  2. 两个对象相等,对两个对象分别调用equals方法都返回true
  3. 两个对象有相同的hashcode值，它们也不一定是相等的
  4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
  5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

### final 关键字

final关键字主要用在三个地方：变量、方法、类。

- 对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。
- 当用final修饰一个类时，表明这个类不能被继承。final类中的所有成员方法都会被隐式地指定为final方法。
- 使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的Java版本已经不需要使用final方法进行这些优化了）。类中所有的private方法都隐式地指定为final。

### String、StringBuffer 和 StringBuilder 的区别是什么？String 为什么是不可变的？

- 可变性

  简单的来说：String 类中使用 final 关键字修饰字符数组来保存字符串，`private　final　char　value[]`，所以 String 对象是不可变的。而StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串`char[]value` 但是没有用 final 关键字修饰，所以这两种对象都是可变的。

  StringBuilder 与 StringBuffer 的构造方法都是调用父类构造方法也就是 AbstractStringBuilder 实现的。

  AbstractStringBuilder.java

  ```java
  abstract class AbstractStringBuilder implements Appendable, CharSequence {
      char[] value;
      int count;
      AbstractStringBuilder() {
      }
      AbstractStringBuilder(int capacity) {
          value = new char[capacity];
      }
  ```

- 线程安全性

  String 中的对象是不可变的，也就可以理解为常量，线程安全。AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了一些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公共方法。StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。

- 性能

  每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象。StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

- 三者总结

  1. 操作少量的数据: 适用String
  2. 单线程操作字符串缓冲区下操作大量数据: 适用StringBuilder
  3. 多线程操作字符串缓冲区下操作大量数据: 适用StringBuffer

### 自动装箱与拆箱

- 什么是装箱？什么是拆箱？

  装箱：基本类型转变为包装器类型的过程。
  拆箱：包装器类型转变为基本类型的过程。

- 装箱和拆箱的执行过程

  - 装箱是通过调用包装器类的 valueOf 方法实现的
  - 拆箱是通过调用包装器类的 xxxValue 方法实现的，xxx代表对应的基本数据类型。
  - 如int装箱的时候自动调用Integer的valueOf(int)方法；Integer拆箱的时候自动调用Integer的intValue方法。

- 常见问题

  - 整型的包装类 valueOf 方法返回对象时，在常用的取值范围内，会返回缓存对象。
  - 浮点型的包装类 valueOf 方法返回新的对象。
  - 布尔型的包装类 valueOf 方法 Boolean类的静态常量 TRUE | FALSE。
  - 包含算术运算会触发自动拆箱。
  - 存在大量自动装箱的过程，如果装箱返回的包装对象不是从缓存中获取，会创建很多新的对象，比较消耗内存。

### String 类的常用方法都有那些？

- equals：字符串是否相同
- equalsIgnoreCase：忽略大小写后字符串是否相同
- compareTo：根据字符串中每个字符的Unicode编码进行比较
- compareToIgnoreCase：根据字符串中每个字符的Unicode编码进行忽略大小写比较
- indexOf：目标字符或字符串在源字符串中位置下标
- lastIndexOf：目标字符或字符串在源字符串中最后一次出现的位置下标
- valueOf：其他类型转字符串
- charAt：获取指定下标位置的字符
- codePointAt：指定下标的字符的Unicode编码
- concat：追加字符串到当前字符串
- isEmpty：字符串长度是否为0
- contains：是否包含目标字符串
- startsWith：是否以目标字符串开头
- endsWith：是否以目标字符串结束
- format：格式化字符串
- getBytes：获取字符串的字节数组
- getChars：获取字符串的指定长度字符数组
- toCharArray：获取字符串的字符数组
- join：以某字符串，连接某字符串数组
- length：字符串字符数
- matches：字符串是否匹配正则表达式
- replace：字符串替换
- replaceAll：带正则字符串替换
- replaceFirst：替换第一个出现的目标字符串
- split：以某正则表达式分割字符串
- substring：截取字符串
- toLowerCase：字符串转小写
- toUpperCase：字符串转大写
- trim：去字符串首尾空格

### 普通类和抽象类有哪些区别？

- 抽象类不能被实例化
- 抽象类可以有抽象方法，抽象方法只需申明，无需实现
- 含有抽象方法的类必须申明为抽象类
- 抽象类的子类必须实现抽象类中所有抽象方法，否则这个子类也是抽象类
- 抽象方法不能被声明为静态
- 抽象方法不能用 private 修饰
- 抽象方法不能用 final 修饰

### 接口和抽象类有什么区别？

1. 接口的方法默认是 public，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），而抽象类可以有非抽象的方法。

2. 接口中除了static、final变量，不能有其他变量，而抽象类中则不一定。

3. 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过extends关键字扩展多个接口。

4. 接口方法默认修饰符是public，抽象方法可以有public、protected和default这些修饰符（抽象方法就是为了被重写所以不能使用private关键字修饰！）。

5. 从设计层面来说，抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范。

   **备注**：在JDK8中，接口也可以定义静态方法，可以直接用接口名调用。实现类和实现是不可以调用的。如果同时实现两个接口，接口中定义了一样的默认方法，则必须重写，不然会报错。

   关于抽象类
   JDK 1.8以前，抽象类的方法默认访问权限为protected
   JDK 1.8时，抽象类的方法默认访问权限变为default
   关于接口
   JDK 1.8以前，接口中的方法必须是public的
   JDK 1.8时，接口中的方法可以是public的，也可以是default的
   JDK 1.9时，接口中的方法可以是private的

### java 中 IO 流有哪些？

- 按数据流向:输入流和输出流

  输入流：数据流向程序

  输出流：数据从程序流出

- 按处理单位:字节流和字符流

  字节流：一次读入或读出是8位二进制

  字符流：一次读入或读出是16位二进制

  JDK 中后缀是 Stream 是字节流；后缀是 Reader，Writer 是字符流

- 按功能功能:节点流和处理流

  节点流：直接与数据源相连，读入或写出

  处理流：与节点流一块使用，在节点流的基础上，再套接一层

**最根本的四大类：InputStream(字节输入流)，OutputStream（字节输出流），Reader（字符输入流），Writer（字符输出流）**

**四大类的扩展，按处理单位区分**

1. InputStream：FileInputStream、PipedInputStream、ByteArrayInputStream、BufferedInputstream、SequenceInputStream、DataInputStream、ObjectInputStream
2. OutputStream：FileOutputStream、PipedOutputStream、ByteArrayOutputStream、BufferedOutputStream、DataOutputStream、ObjectOutputStream、PrintStream
3. Reader：FileReader、PipedReader、CharArrayReader、BufferedReader、InputStreamReader
4. Writer：FileWriter、PipedWriter、CharArrayWriter、BufferedWriter、InputStreamWriter、PrintWriter

**常用的流**

1. 对文件进行操作：FileInputStream（字节输入流）、FileOutputStream（字节输出流）、FileReader（字符输入流）、FileWriter（字符输出流）
2. 对管道进行操作：PipedInputStream（字节输入流）、PipedOutStream（字节输出流）、PipedReader（字符输入流）、PipedWriter（字符输出流）
3. 字节/字符数组：ByteArrayInputStream、ByteArrayOutputStream、CharArrayReader、CharArrayWriter
4. Buffered 缓冲流：BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter
5. 字节转化成字符流：InputStreamReader、OutputStreamWriter
6. 数据流：DataInputStream、DataOutputStream
7. 打印流：PrintStream、PrintWriter
8. 对象流：ObjectInputStream、ObjectOutputStream
9. 序列化流：SequenceInputStream

**常见问题**

既然有了字节流，为什么还要有字符流？

问题本质想问：不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？

回答：字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

### BIO、NIO、AIO 有什么区别？

- BIO：同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。（JDK1.4以前）

- NIO：同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。（JDK1.4）

- AIO：异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的 IO 请求都是由 OS 先完成了再通知服务器应用去启动线程进行处理。（JDK1.7）

  

- BIO：线程发起 IO 请求，不管内核是否准备好 IO 操作，从发起请求起，线程一直阻塞，直到操作完成。

- NIO：线程发起 IO 请求，立即返回；内核在做好 IO 操作的准备之后，通过调用注册的回调函数通知线程做 IO 操作，线程开始阻塞，直到操作完成。

- AIO：线程发起 IO 请求，立即返回；内存做好 IO 操作的准备之后，做 IO 操作，直到操作完成或者失败，通过调用注册的回调函数通知线程做 IO 操作完成或者失败。

### 深拷贝 vs 浅拷贝

- **浅拷贝**：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。

- **深拷贝**：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

### 序列化

**ProtoBuffer**

`Protocol Buffers` 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 `RPC` 数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。

- 优点

  - **Protobuf 更小、更快、也更简单**。你可以定义自己的数据结构，然后使用代码生成器生成的代码来读写这个数据结构。你甚至可以在无需重新部署程序的情况下更新数据结构。只需使用 `Protobuf` 对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对你的结构化数据轻松读写。
  - **“向后”兼容性好**，人们不必破坏已部署的、依靠“老”数据格式的程序就可以对数据结构进行升级。这样您的程序就可以不必担心因为消息结构的改变而造成的大规模的代码重构或者迁移的问题。因为添加新的消息中的 `field` 并不会引起已经发布的程序的任何改变。
  - **Protobuf 语义更清晰**，无需类似 XML 解析器的东西（因为 `Protobuf` 编译器会将 `.proto` 文件编译生成对应的数据访问类以对 `Protobuf` 数据进行序列化、反序列化操作）。
  - **Protobuf 的编程模式比较友好**，简单易学，同时它拥有良好的文档和示例，对于喜欢简单事物的人们而言，Protobuf 比其他的技术更加有吸引力。

- 不足

  由于文本并不适合用来描述数据结构，所以 `Protobuf` 也不适合用来对基于文本的标记文档（如 HTML）建模。另外，由于 XML 具有某种程度上的自解释性，它可以被人直接读取编辑，在这一点上 `Protobuf` 不行，它以二进制的方式存储，除非你有 `.proto` 定义，否则你没法直接读出 `Protobuf` 的任何内容。

### 运算符优先级

相同优先级中，按结合顺序计算。**大多数运算是从左至右计算，只有三个优先级是从右至左结合的，它们是单目运算符、条件运算符、赋值运算符**。

基本的优先级需要记住：

- 指针最优，单目运算优于双目运算。如正负号。
- 先乘除（模），后加减。
- 先算术运算，后移位运算，最后位运算。请特别注意：`1 << 3 + 2 & 7`等价于 `(1 << (3 + 2)) & 7`.
- 逻辑运算最后计算。

### 异常

Java中有Error和Exception，它们都是继承自Throwable类。

- 区别：

  Exception：

  - 可以是可被控制(checked) 或不可控制的(unchecked)。
  - 表示一个由程序员导致的错误。
  - 应该在应用程序级被处理。

  Error：

  - 总是不可控制的(unchecked)。
  - 经常用来用于表示系统错误或低层资源的错误。
  - 如何可能的话，应该在系统级被捕捉。

  **异常和错误的区别：异常能被程序本身处理，错误是无法处理。**

- 异常分类：

  - Checked exception: 这类异常都是Exception的子类。异常的向上抛出机制进行处理，假如子类可能产生A异常，那么在父类中也必须throws A异常。可能导致的问题：代码效率低，耦合度过高。
  - Unchecked exception: 这类异常都是RuntimeException的子类，虽然RuntimeException同样也是Exception的子类，但是它们是非凡的，它们不能通过client code来试图解决，所以称为Unchecked exception 。

- 异常处理总结：

  - try 块： 用于捕获异常。其后可接零个或多个catch块，如果没有catch块，则必须跟一个finally块。
  - catch 块： 用于处理try捕获到的异常。
  - finally 块： 无论是否捕获或处理异常，finally块里的语句都会被执行。当在try块或catch块中遇到return 语句时，finally语句块将在方法返回之前被执行。

  **注：**当try语句和finally语句中都有return语句时，在方法返回之前，finally语句的内容将被执行，并且finally语句的返回值将会覆盖原始的返回值。如下：

  ```java
  public static int f(int value) {
          try {
              return value * value;
          } finally {
              if (value == 2) {
                  return 0;
              }
          }
      }
  ```

  如果调用 `f(2)`，返回值将是0，因为finally语句的返回值覆盖了try语句块的返回值。

  **在以下4种特殊情况下，finally块不会被执行：**

  1. 在finally语句块第一行发生了异常。 因为在其他行，finally块还是会得到执行
  2. 在前面的代码中用了System.exit(int)已退出程序。 exit是带参函数 ；若该语句在异常语句之后，finally会执行
  3. 程序所在的线程死亡。
  4. 关闭CPU。

