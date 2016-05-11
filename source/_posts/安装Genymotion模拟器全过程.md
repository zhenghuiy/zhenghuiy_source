title: "安装Genymotion模拟器全过程"
date: 2014-05-07 21:32:50
tags: Genymotion, 模拟器
categories: 工具
---
今天在逛网站的时候，偶然看到一个号称“最火Android模拟神器”的Genymotion，它的特点简而言之就是秒开、方便、可与Eclipse相连——看介绍简直就是Android开发者的救星啊。
 
心动不如行动，马上动手搞起。在这里记录下安装与使用的全过程，介绍类的语言大家去网上找找，我这里只简单描述，力求以最短的时间上手使用。
 
##一，下载Genymotion
 
 
[进入官网](https://www.genymotion.com/)，简单的注册一下。
 
在注册成功后网站会自动引导你进入下载页面（为网站的设计点赞，一点不拖泥带水！），我已经注册过，所以进入官网后点击下图中的按钮。进入下载页面。
![](http://zhenghuiy-blog.qiniudn.com/1-1024x519.png)
    
 
    
 
进入下载页面后，有多个版本可供选择，大家选择自己对应的版本。注意一点是，Genymotion需要有Oracle VirtualBox的环境。因此，如果你恰好使用的是windows操作系统，那么恭喜你，选择图中标注的版本下载即可，该版本包含了Oracle VirtualBox。如果你使用的是其他操作系统，那么你应该选择其他版本，并且得先在电脑中安装Oracle VirtualBox（版本越高越好）。
![](http://zhenghuiy-blog.qiniudn.com/2-1024x541.png)   
 


 
##二，安装Genymotion
 
 
软件安装过程不多表述，与一般的软件安装过程一致（我个人习惯是将安装目录从C盘改为D盘，不知道大家是否一样）。
 
安装完毕后默认运行。
 
进去后第一个界面是这样的，自动弹出增加一个模拟器（再次为用户体验点赞！这只是小细节，没啥技术难度，但是这样设计的好处无疑是非常方便新用户上手。）
 
![](http://zhenghuiy-blog.qiniudn.com/3.png)
 
<!--more-->
 
点击确认后出现下图。红色凸显的那一块有个connect.毫无疑问，点击连接！
![](http://zhenghuiy-blog.qiniudn.com/4.png)
 

 
输入用户名和密码（现在知道为什么让你在官网上注册了吧）
![](http://zhenghuiy-blog.qiniudn.com/5.png)    
 
 
连接后Available virtual devices框中会出现一系列可供选择的列表，点击选择一个自己喜欢的，然后点击next。
![](http://zhenghuiy-blog.qiniudn.com/6.png) 
    
 
这里的名字可改可不改。建议不改，因为原有的名字中包含了android版本号、API版本和分辨率等等，有助于开发。
![](http://zhenghuiy-blog.qiniudn.com/7.png)    
 
Downloading….有事的去喝杯茶，上个厕所什么的。没事的可以点击左右按钮，看看介绍。最后，finish.
![](http://zhenghuiy-blog.qiniudn.com/8.png)    
 
模拟器后对应四个操作，分别是设置、拷贝、恢复出厂设置、删除。对于免费版，可以使用的只有第一、四项。点击设置，可以设置各项参数。OK，点击play，开始玩模拟器。
![](http://zhenghuiy-blog.qiniudn.com/9.png)    
 
果然是秒开！太快了！注意到右侧的一列按钮，都非常有用。灰色都是付费的高福帅才能用的。到此，大家就可以愉快的玩耍了。
![](http://zhenghuiy-blog.qiniudn.com/10.png)    
 
 
##三，安装eclipse插件
 
 
安装eclipse插件，有两种方法，这个和其他插件的安装是一样的。
 
第一种是更新网站，Name: [Genymobile Location](http://plugins.genymotion.com/eclipse)
 
第二种是手动方式，进入Genymotion插件官方下载页面，下载完毕后将该插件移动到Eclipse安装目录下的plugins子目录当中。
 
果断选择第一种方式。安装过程中会有一个warning（提醒您该插件没有签名）,无视它，点击OK。
![](http://zhenghuiy-blog.qiniudn.com/11.png)   
 
安装完毕后重启eclipse，注意到工具栏中多了一个图标。
![](http://zhenghuiy-blog.qiniudn.com/12.png)    
 
点击后出现一个对话框，点击OK，设置路径
![](http://zhenghuiy-blog.qiniudn.com/13.png)   
 
 
不多说，选择Genymotion的安装路径下的Genymotion,即默认是C:\Program
 
Files\Genymobile\Genymotion（我之前安装的时候改成了D盘，所以这里也改成D盘目录）。
![](http://zhenghuiy-blog.qiniudn.com/16.png)
    
 
倘若你路径设置错误，则点击图标后会出现以下错误
![](http://zhenghuiy-blog.qiniudn.com/17-1024x431.png)    


这时候只要打开preference，再重新设置路径就行。
![](http://zhenghuiy-blog.qiniudn.com/14.png)    


##四、开始享受新的模拟器吧！
 
    
 
体验了会儿，真的很快！不只体现在模拟器的开启，就是Activity的跳转也是堪比真机（甚至比真机还快），点击后响应速度杠杠的。我觉得可以抛弃原生的模拟器了。。
 
 
[参考资料](http://mobile.51cto.com/android-405002.htm)
