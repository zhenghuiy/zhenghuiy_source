title: Android 性能典范：拯救计划
tags:
  - Android
  - Android性能
categories:
  - Android性能
date: 2016-04-05 19:38:00
---
# 前言
今天逛稀土时偶然看到[hanks](http://gold.xitu.io/#/user/567a559d60b25aa3dcb2a1ce)分享的一篇英文文章，粗略浏览便已觉得不错，因此翻译成中文，与君分享。

原文地址：[Android Performance Patterns: Rescue tips](https://medium.com/@laanayabdrzak/android-performance-patterns-rescue-tips-8c1e4c7cb1f0#.xzvcgcall)

# 正文
现在的app到处都充斥着华丽的动画、复杂的转化还有自定义View，然而用户体验必须尽可能直观且类似。以下这些范例将会帮助你做出一个流畅的、快速响应的、甚至可能减少电量损耗的app，这些范例由一些可以提升整体应用表现的微优化组成。
<!--more-->
## 避免“坏”表现
- 避免堵塞主线程
- 避免可能引发大范围重绘的不必要的重绘
- 用 `RelativeLayout` 来减少布局层级
- 避免在 `LinearLayout` 中使用嵌套的 weight 属性（因为weight属性会使每个子View进行两次measure）
- 避免使用没有恰当处理的自定义View
- 避免创建没必要的对象
- 将常量声明为 static final（static比普通变量快 15% - 20%）
- 使用基本数据类型（Integer、Float 比基本类型慢两倍）
- 避免内部的 `getter` 和 `setter`(直接访问属性可以快3倍)
- 使用改进的循环语法【译者注：这里应该是指for each循环】
- 对私有的内部类考虑使用包访问级别代替私有访问级别
- 谨慎使用native方法

## 自定义View
- 遵循KISS原则
- 在布局中使用`merge`标签来作为根标签（避免额外的`ViewGroup`）
- 使用`include`标签（便于布局的复用）
- 避免不必要的布局
- 不要在`onDraw`中申请内存或者做复杂逻辑
- 去除不必要的`invalidate()`调用
- 考虑创建自己的`ViewGroup`
- 用`RecyclerView`替代`ListView`和`GridView`

## 避免内存抖动
- 不要申请大量不必要的对象内存：
  1, 不可变对象：String
  2, 自动装箱：Integer, Boolean...
- 考虑使用对象池并缓存来减少内存抖动
- 留心`enum`类型的开销（一个指向枚举类型的引用就要占据4个字节）

## 避免内存泄漏
- 不要在内部类里泄漏context实例
- 不要在`activity`里泄漏view实例
- 使用内部静态类优于非静态的
- 除非键都是`WeakReference`，否则不要使用`WeakHashmap`作为缓存

## CPU
- 不要嵌套多通路布局
- 当需要时才去进行复杂的计算【译者注：类似懒加载】
- 缓存复杂计算的结果以复用
- 考虑 `RenderScript` 的性能
- 尽可能减少主线程的工作

## 避免过度绘制
- 精简`drawable`
- 在透明部分使用.9图
- 设置view的透明度时多注意
- 去除view中无用的背景

## bitmap
- 将bitmap解码为需要的尺寸：`BitmapFactory.Options（
inSampleSize, inDensity, inTargetDensity）`
- 加载bitmap到内存时，设置尺寸为显示尺寸
- 如无必要不要进行缩放
- 使用`LRU`缓存

## Service
- 除非Service在处理事务否则不要让其保持运行。同时也要小心`stopService`当Service工作完成时
- 系统倾向保留有Service运行的进程，那么被service占用的内存将无法被其他进程使用或者被内存置换
- 限制service生命周期的最佳实践是使用`IntentService`，它会在工作完成后结束自身
- 让没必要存活的Service继续运行是Android app内存管理最差的举动之一

## 线程
- 在线程的`run()`方法中使用   `Process.setThreadPriority(THREAD_PRIORITY_BACKGROUND)`可以减少该线程及UI线程的计算性能损耗
- 如果你没有通过这种方式为线程设置低优先级，那么该线程仍会拖慢你的app，因为默认情况下它的优先级与UI线程的优先级相同
- 维护住当前线程的引用，以便你之后可能先打断该线程。例如：当网络连接失败你可以取消该线程

## 避免ANR
- UI线程中做的事越少越好
- 如果应用正在后台响应用户的输入，最好显示进度给用户（例如显示一个进度条）
- 使用**Systrace**或**Traceview**等性能工具来检测应用响应能力的瓶颈
- 如果你的应用有一个非常耗时的初始化过程，考虑使用启动页或者尽快渲染主要的view，表明正在加载中并且正在显示异步的信息