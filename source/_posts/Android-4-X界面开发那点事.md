title: "Android 4.X界面开发那点事"
date: 2014-05-27 10:18:53
tags: Android, UI, 适配
categories: Android
---
###前言：
 
为什么要使用dp/dip?导航栏的宽度/高度应该设为多少dp合适？一般情况下应该使每个控件的高度显示为多少？本文就是要回答这些问题。
 
 
##一，Android中的各种单位

 
在android开发过程中，我们无法忽略的一个问题就是设备适配问题。从我们刚开始学android的时候，各类教程上就不厌其烦的说，“android碎片化严重，适配很麻烦”、“控件的长度用dip作单位，字体用sp”等等。我们有必要去详细了解一下这些东西，在开发过程中才能游刃有余。
 
 
下面是android中的常见单位：  
 
- px（像素）可以理解为一个小颜色块，是设备显示时的单位；
- dip是设备独立像素，不同设备有不同的显示效果，和具体硬件有关；备注：dip == dp
- dpi是屏幕像素密度，每英寸像素数；
- sp是像素缩放。google建议用于字体显示。

dpi等于对角线的像素值（=$\sqrt{长^2+宽^2}$）除于英寸数。比如，分辨率480 x 800，屏幕尺寸4.3英寸的DPI为：216.
 
为了方便，google将dpi分为多种模式*LDPI*, *MDPI*, *HDPI*, *XHDPI*, *XXHDPI*, and *XXXHDPI*，常见四个如下，分别对应资源文件中的四个后缀：
 
![](http://ww2.sinaimg.cn/large/8d608af3gw1egsz5tfcf2j20h602oaa1.jpg)
 
也就是说，216会被划分进hdpi。
<!--more-->
这里有一个基准：160。因为第一款Android设备（HTC的T-Mobile G1）是属于（约等于）160dpi的，以160为基准，乘以0.5、1、2都可以比较好的对应一个dpi模式。因此有了dip转px的公式：
 
$$px = dp * (dpi / 160)$$
 
简而言之，单位dp从表现上和px类似，但是它的长度是不固定的，会根据当前硬件的参数而变化，越宽，每dp就越长（相对于固定的px而言），越窄，每dp就越短。
 
因此，为了有更好的显示效果，按照google的推荐，字体使用sp为单位，其他元素使用dp为单位，需要缩放的图使用.9图。
 
##二，Android中的尺寸

Android系统现在已经百花齐放，手机、电视、平板、手表等等都可以刷进Android系统。它的开源性质造就了它远超其他系统的市场份额。就手机和平板而言，google官方将600dp以下为手机（handset）,600dp以上的为平板（tablet）.   
 
 
###48dp
我们经常会有这样一个需求，就是显示一个小控件让用户输入或者单纯展示信息，比如一个EditText。
 
![](http://ww4.sinaimg.cn/large/8d608af3gw1egsz7ybfnmj20le02ujri.jpg) 

      
那么，这个小控件设置为多少合适呢？答案是48dp.
 
因为要使一个可以被点击的控件能比较好的被点击，它的尺寸应该在7到10毫米之间。而根据各种设备上的状况平均来说，48dp被转化为物理尺寸后都会在9毫米左右。
 
因此，使用48dp可以保证你的控件能控制为7-10mm这个区间内。随之而来的好处是，既可以比较好的控制合适的信息密度，又可以兼顾控件的点击效果。
 
 ![](http://ww2.sinaimg.cn/large/8d608af3gw1egsz8z6i1kj20d5042dg1.jpg)

 如下图，ActionBar、每一行的EditText和NavigationBar的高度都设置为了48dp,整体达到一个不错的展示效果。
 
![](http://ww4.sinaimg.cn/large/8d608af3gw1egsz9n827bj20md0fhq46.jpg)

###后记：
 
参考资料为：http://developer.android.com/design/style/metrics-grids.html
 
后半段的内容大部分翻译自android文档，一方面感叹Google把文档写的如此之好，另一方面自己尝试着翻译之后也十分佩服那些翻译书籍的作者。
 
 有不妥之处，还请斧正。