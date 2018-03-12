title: Android中Serializable和Parcelable的对比
tags:
  - Android
categories:
  - Android性能
date: 2016-03-09 22:36:00
---
## 前言
对于Android开发者来说，序列化总是一个不能避免的问题。前有“使用`enum`实现单例模式可以自动序列化”的观点，后有“在Intent传输过程中使用`Parcelable`进行序列化可以减少性能损耗”的思考。

由此，我们很有必要找到应对各种场景的最佳实践。

本文中我们谈谈`Serializable` 和 `Parcelable`。

## 结论
二话不说先说结论（就是这么直接干脆）：序列化的数据仅需在内存中传输的，使用`Parcelable`；反之，如果需要持久化或者网络传输等等的，请使用`Serializable`。

## 论述
`Serializable`相信大家不会陌生，从Java中就有这个接口存在。`Serializable`接口是一种标识接口，它的迷人之处在于，你只要让需要序列化的类实现该接口，Java便会对该类进行高效的序列化。
<!--more-->
是不是觉得很方便很简洁很爽？然而它的缺点也在于此，由Java进行序列化工作便需要进行各种反射，势必带来性能问题。特别是对于Android这种大量使用Intent进行通信的系统来说，大量使用`Serializable`容易引起垃圾回收。

所以Google工程师们就创建出了`Parcelable`。因为`Parcelable`的序列化工程都是已知的（由实现接口的人去写），所以不需要反射，性能自然就上来了。

相应得，这种机制就要求实现接口的人去写大量的模版代码。正所谓有得必有失嘛。

但是作为讲究性能与代码可读性平衡的开发者，我认为这些代码是可以接受的。

## 最佳实践
1，intent传递数据时，使用`Parcelable`序列化数据；
2，数据持久化时使用`Serializable`序列化数据。

以上两点是频度最高的两个应用场景，请记在脑中，最好形成反射。触发了前面的条件就biubiubiu，无脑用上去就行。——看看我为你们节省了多少脑细胞啊。

## 特别注意
**千万不要用`Parcelable`去进行内存以外的序列化。**
你如果为了偷懒，无脑使用`Serializable`在所有场景，问题都不大，顶多是一点性能问题。但是，`Parcelable`的实现不是一种通用的接口，很有可能你在这个SDK版本中实现序列化的数据，在另一个版本中就直接乱码了。这点需要特别注意。

## 小思考
既然`Parcelable`有版本问题，那`Serializable`有么？我觉得有可能也有，因为该接口的实现也是由JDK版本决定的嘛。但是既然目前都没人提出，个人感觉应该是这个类的实现在常用的7、8版本中都没有变更过。所以可以放心安全的用。这个问题以后再验证了。

## 参考资料
[Android系统中Parcelable和Serializable的区别](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0204/2410.html)