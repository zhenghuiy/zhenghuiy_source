title: "Java中类的构造顺序"
date: 2014-08-01 14:56:08
tags: 类的构造顺序
categories: Java
---
先看代码：

Father类
```
package com.zhenghuiyan.testorder;

public class Father {
    public String fStr1 = "father1";
    protected String fStr2 = "father2";
    private String fStr3 = "father3";
    
    {
        System.out.println("Father common block be called");
    }
    
    static {
        System.out.println("Father static block be called");
    }
    
    public Father() {
        System.out.println("Father constructor be called");
    }

}
```
首先是Father类，该类有一个默认构造函数，有一个static的代码块，为了方便查看结果，还有一个普通代码块。
<!--more-->
Son类
```
package com.zhenghuiyan.testorder;

public class Son extends Father{
    public String SStr1 = "Son1";
    protected String SStr2 = "Son2";
    private String SStr3 = "Son3";
    
    {
        System.out.println("Son common block be called");
    }
    
    static {
        System.out.println("Son static block be called");
    }
    
    public Son() {
        System.out.println("Son constructor be called");
    }
    
    public static void main(String[] args) {
        new Son();
        System.out.println();
        new Son();
    }

}
```
Son类的内容与Father类基本一致，不同在于Son继承自Father。该类有一个main函数，仅为了测试用，不影响结果。

在main函数中实例化Son。

结果为：
```
Father static block be called
Son static block be called
Father common block be called
Father constructor be called
Son common block be called
Son constructor be called

Father common block be called
Father constructor be called
Son common block be called
Son constructor be called
```
观察结果，并结合书籍中的知识，我们可以了解到：

1，在类加载的时候执行父类的static代码块，并且只执行一次（因为类只加载一次）；

2，执行子类的static代码块，并且只执行一次（因为类只加载一次）；

3，执行父类的类成员初始化，并且是从上往下按出现顺序执行（在debug时可以看出）。

4，执行父类的构造函数；

5，执行子类的类成员初始化，并且是从上往下按出现顺序执行。

6，执行子类的构造函数。
