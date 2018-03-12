title: 记一件需要反省的事——如何实现webView内部跳转
tags:
  - Android
  - webView
categories:
  - Android
date: 2016-04-02 23:31:00
---
# 起因
今天在做一个“WebView内部跳转”的小需求时，发现了一件比较诡异的事：项目中没有在 `shouldOverrideUrlLoading`中主动去用`view.loadUrl`逻辑处理，为何能够实现WebView内部跳转呢？

带着这个小疑惑，我去问了一个厉害的同事，他说道，只要`shouldOverrideUrlLoading`返回值为`false`，即可实现内部跳转。

听到这个我有些困扰，因为记忆中一直是需要手动去load新的url才能实现，否则就跳浏览器的。然后google了下，就搜到以下两个：

[[shouldOverrideUrlLoading return method](http://stackoverflow.com/questions/16693507/shouldoverrideurlloading-return-method)]

[[WebViewClient shouldOverrideUrlLoading 常见错误用法](http://www.cnblogs.com/shaobin0604/p/3313680.html)]

我的内心是崩溃的。。

# 分析原因
我之所以会记错技术点，小原因有三：
1，我之前使用WebView都比较简单，没有去设置WebViewClient，所以会有链接跳转都交由系统处理的惯性思维；
2，在之前的项目中，接触到了系统对webView中的以http协议开头的处理，更加加深了“系统会主动去处理链接”的想法；
3，然后就是之前包括刚刚搜索了几次“webView 内部跳转”，都是说需要手动去调用`view.loadUrl`处理的博客。
<!--more-->
最后，一个最大的原因就是，**求知之心不够坚定，对技术细节不求甚解。**

比如，虽然搜索“webView 内部跳转”不管google还是百度都被错误用法的那篇博客占领，但是如果我有对`shouldOverrideUrlLoading`多搜索多一步了解，那势必可以避免这个问题。

由这点联想到平时的学习习惯，深感自己应该好好反省。

在这里定下以后的学习习惯：**任何安卓相关的技术点，必然要有官网的资料作为佐证，否则得自己去写个小程序验证，才能真正“相信”。**

也许各位看客可能会觉得我有点小题大做，但是对于经历过这样尴尬时刻的我而言，这绝不是一件小事。我对自己最低的要求就是做事认真。

嗯，反正以此为戒吧。

# 如何实现webView内部跳转

现在回到原题，“如何实现webView内部跳转”的结论是什么呢？

> 1, 若没有设置 WebViewClient 则在点击链接之后由系统处理该 url，通常是使用浏览器打开或弹出浏览器选择对话框。
2, 若设置 WebViewClient 且该方法返回 *true* ，则说明由应用的代码处理该 url，WebView 不处理。
3, 若设置 WebViewClient 且该方法返回 *false*，则说明由 WebView
 处理该 url，即用 WebView 加载该 url。