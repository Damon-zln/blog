---
title: java类初始化顺序
date: 2019-02-17 14:15
categories: Java笔记
tags:
- java
---

### Java类中的初始化顺序

- 静态属性
- 静态代码块
- 普通属性（非静态）
- 普通代码块（非静态）
- 构造函数
- 方法
<!--more-->

```java
package per.dazhan.example;

/**
 * @author Damon-zln
 * @date 2019/2/17 14:15
 * @description demo1
 * @update
 */
public class demo1 {
    /**
     * java类中的初始化顺序
     * 1. 静态属性: 带有static关键字定义的属性
     * 2. 静态代码块: static{}
     * 3. 普通属性: 不带static关键字定义的属性
     * 4. 普通代码块: {}
     * 5. 构造函数: 与类名一样的方法
     * 6. 方法: 普通方法
     * @param args
     */

    public static void main(String[] args) {
        new A();
    }
}

class A {
    // 静态属性
    private static String staticField = getStaticField();
    // 普通属性
    private String field = getField();

    // 静态代码块
    static {
        System.out.println("静态方法块初始化----2");
    }

    // 普通代码块
    {
        System.out.println("普通方法块初始化----4");
    }

    // 构造函数
    public A() {
        System.out.println("构造函数初始化----5");
    }

    private static String getStaticField() {
        String staticField = "staticField";
        System.out.println("静态属性初始化----1");
        return staticField;
    }

    private String getField() {
        String field = "field";
        System.out.println("普通属性初始化----3");
        return field;
    }
}
=================================================================================
执行结果：
静态属性初始化----1
静态方法块初始化----2
普通属性初始化----3
普通方法块初始化----4
构造函数初始化----5

Process finished with exit code 0
```

### 父类与子类 （普通类）

- 父类 （如上）
- 继承的子类
  - 父类静态属性
  - 父类静态代码块
  - 子类静态属性
  - 子类静态代码块
  - 父类普通属性
  - 父类普通代码块
  - 父类构造函数
  - 子类普通属性
  - 子类普通代码块
  - 子类构造函数

```java
class B extends A {
    // 静态属性
    private static String staticField = getStaticField();
    // 普通属性
    private String field = getField();

    // 静态代码块
    static {
        System.out.println("子类--静态方法块初始化----2");
    }

    // 普通代码块
    {
        System.out.println("子类--普通方法块初始化----4");
    }

    public B() {
        super();
        System.out.println("子类--构造函数初始化----5");
    }

    private static String getStaticField() {
        String staticField = "staticField";
        System.out.println("子类--静态属性初始化----1");
        return staticField;
    }

    private String getField() {
        String field = "field";
        System.out.println("子类--普通属性初始化----3");
        return field;
    }
}
=========================================================================
main方法执行new B（）；
执行结果：
静态属性初始化----1
静态方法块初始化----2
子类--静态属性初始化----1
子类--静态方法块初始化----2
普通属性初始化----3
普通方法块初始化----4
构造函数初始化----5
子类--普通属性初始化----3
子类--普通方法块初始化----4
子类--构造函数初始化----5

Process finished with exit code 0
```

### 接口-抽象类-实现类

- 接口静态变量
- 抽象类静态变量
- 抽象类静态代码块
- 实现类静态变量
- 实现类静态代码块
- 抽象类普通变量
- 抽象类普通代码块
- 抽象类构造函数
- 实现类普通变量
- 实现类普通代码块
- 实现类构造函数

注意：

1. 接口中声明的变量都是final，必须实例化，子类无法修改，固定值
2. 接口中可以有静态方法，不能有普通方法，普通方法需要用default添加默认实现
3. 接口中没有静态代码块、普通变量、普通代码块、构造函数
