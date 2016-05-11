title: "[转]Java中单例模式的实现"
date: 2014-08-03 15:05:35
tags: Java, 单例
categories: Java
---
##前言：

近日在看Effective JAVA这本书，看到这一条的时候去网上查找下资料，发现了这篇写的不错的博客，因此觉得没有重复造轮子的必要，特此转载。

原文：

Inspired by Effective Java.

Singleton模式是在编程实践中应用最广泛的几种设计模式之一。以前知道的，实现单例的方法有两种(下面的A、B)。刚刚在读《Effective Java的时候》学到一种新的更好的方法(E)：单元素的枚举类型。同时通过网上资料也知道了其他两种方法(C、D)。最后一种在Java中从1.5版本开始支持，其他语言在验证后说明。

##A.饿汉式(类加载的时候就创建实例)。

代码如下：
```
public class MaYun {

    public static final Mayun instance = new Mayun(); //静态的final的MaYun
    
    private MaYun() {
        //MaYun诞生要做的事情
    
    }
    
    public void splitAlipay() {
    
        System.out.println(“Alipay是我的啦！看你丫Yahoo绿眉绿眼的望着。。。”);
    
    }

}

Call：MaYun.instance.splitAlipay();
```
**Feature**：可以通过反射机制攻击；线程安全[多个类加载器除外]。
<!--more-->

##A+.饿汉变种[推荐]
```
public class MaYun {

    private static Mayun instance = new Mayun();
    
    private static getInstance() {
    
        return instance;
    
    }
    
    private MaYun() {
    
        //MaYun诞生要做的事情
    
    }
    
    public void splitAlipay() {
    
        System.out.println(“Alipay是我的啦！看你丫Yahoo绿眉绿眼的望着。。。”);
    
    }

}
```
##A++.饿汉变种(类初始化的时候实例化instance)：
```
public class MaYun {
    
    private MaYun instance = null;
    
    static {
    
        instance = new MaYun();
    
    }
    
    private MaYun() {
    
        //MaYun诞生要做的事情
        
        }
        
        public static MaYun getInstance() {
        
            return this.instance;
        
        }
        
        public void splitAlipay() {
        
            System.out.println(“Alipay是我的啦！看你丫Yahoo绿眉绿眼的望着。。。”);
        
        }

}
```
##B.懒汉式。

代码如下：
```
public class MaYun {
    
    private static MaYun instance = null;
    
    private MaYun() {
    
        //MaYun诞生要做的事情
    
    }
    
    public static MaYun getInstance() {
        
        if (instance == null) {
        
            instance = new MaYun();
        
        }
        
        return instance;
    
    }
    
    public void splitAlipay() {
    
        System.out.println(“Alipay是我的啦！看你丫Yahoo绿眉绿眼的望着。。。”);
    
    }

}

Call：MaYun.getInstance().splitAlipay();
```
**Feature**:延时加载；线程不安全，多线程下不能正常工作；需要额外的工作(Serializable、transient、readResolve())来实现序列化。

##B+.懒汉式变种。
```
public class MaYun {
    
    private static MaYun instance = null;
    
    private MaYun() {
    
        //MaYun诞生要做的事情
    
    }
    
    public static synchronized MaYun getInstance() {
        
        if (instance == null) {
        
            instance = new MaYun();
        
        }
        
        return instance;
    
    }
    
    public void splitAlipay() {
    
        System.out.println(“Alipay是我的啦！看你丫Yahoo绿眉绿眼的望着。。。”);
    
    }

}
```
**Feature**:线程安全；效率比较低，因为需要线程同步的时候比较少。

##C.静态内部类[推荐]。

代码如下：
```
public class MaYun {
    
    private static class SigletonHolder {
    
        private static final instance = new MaYun();
    
    }
    
    public static final getInstance() {
    
        return SigletonHolder.instance;
    
    }
    
    private MaYun() {
    
        //MaYun诞生要做的事情
    
    }
    
    public void splitAlipay() {
    
        System.out.println(“Alipay是我的啦！看你丫Yahoo绿眉绿眼的望着。。。”);
    
    }
}
Call：MaYun.getInstance().splitAlipay();
```
**Feature**:线程安全；延迟加载。

##D.双重校验锁[不推荐]。

代码如下：
```
public class MaYun {
    
    private volatile static MaYun instance;
    
    private MaYun (){}
    
    public static MaYun getInstance() {
        
        if (instance == null) {
            
            synchronized (MaYun.class) {
                
                if (instance == null) {
                
                    instance = new MaYun();
                
                }
            
            }
        
        }
        
        return instance;
    
    }

}
```
**Feature**：jdk1.5之后才能正常达到单例效果。

##E.编写一个包含单个元素的枚举类型[极推荐]。

代码如下：
```
public enum MaYun {
    
    himself; //定义一个枚举的元素，就代表MaYun的一个实例
    
    private String anotherField;
    
    MaYun() {
        
        //MaYun诞生要做的事情
        
        //这个方法也可以去掉。将构造时候需要做的事情放在instance赋值的时候：
        
        /** himself = MaYun() {
        
        * //MaYun诞生要做的事情
        
        * }
        
        **/
        
    }
    
    public void splitAlipay() {
    
        System.out.println(“Alipay是我的啦！看你丫Yahoo绿眉绿眼的望着。。。”);
    
    }

}

Call：MaYun.himself.splitAlipay();
```
**Feature**:从Java1.5开始支持；无偿提供序列化机制，绝对防止多次实例化，即使在面对复杂的序列化或者反射攻击的时候。

总之，五类：懒汉，恶汉，双重校验锁，静态内部类，枚举。

- 恶汉：因为加载类的时候就创建实例，所以线程安全(多个ClassLoader存在时例外)。缺点是不能延时加载。

- 懒汉：需要加锁才能实现多线程同步，但是效率会降低。优点是延时加载。

- 双重校验锁：麻烦，在当前Java内存模型中不一定都管用，某些平台和编译器甚至是错误的，因为instance = new MaYun()这种代码在不同编译器上的行为和实现方式不可预知。

- 静态内部类：延迟加载，减少内存开销。因为用到的时候才加载，避免了静态field在单例类加载时即进入到堆内存的permanent代而永远得不到回收的缺点(大多数垃圾回收算法是这样)。

- 枚举：很好，不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。但是失去了类的一些特性，没有延迟加载，用的人也太少了～～

以后多推广推广单元素枚举这种更好的单例实现方式。在项目中的代码开始修改实施。

转载自：http://callmegod.iteye.com/blog/1474441


