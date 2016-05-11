title: "在github上参与开源项目"
date: 2014-07-22 11:36:39
tags: github, 参与开源项目
categories: 记事
---
近日用到一个不错的开源库[AndroidCharts](https://github.com/HackPlan/AndroidCharts)使用方便，易于扩展和自定制。在使用的过程中发现了一个小bug,于是果断发起了pull request。第一次使用GitHub参与开源项目还是十分手生的，需要各种查资料，学习Git的用法。

参考了许多资料，这里记录一下在GitHub上参与开源项目的过程。 

##1，寻找项目

可以是你使用到的项目，你觉得想参与的，或者在这里找：

- [GitHub Explore](https://github.com/explore)：受欢迎和热门的项目。 
- [GitHub Stars](https://github.com/stars?direction=desc&sort=created)：被其他人star过的项目（指的是你自己库的项目）。 
- [GitHub Showcases](https://github.com/showcases)：一个能搜索相关库的方法。 

##2，查看社区和文档

一般项目中都有的文件。

- Readme：几乎所有的Github项目都包含一个README.md文件。readme提供了该项目的一个概览及关于如何使用、构建甚至如何贡献于一个项目的相关细节。

- Contributing：项目和项目维护者不同，所以每个项目所期望的作贡献的最佳方法也会有所不同。一定要注意一个标注为CONTRIBUTING的文档，Contributing文档详细描述了一个项目的维护者希望看到贡献的补丁或功能应该符合怎样的规格。这可能包含要写什么测试，代码语法规范或补丁集中的区域。

- License：一个 LICENSE 文件当然就是该项目的许可证了。一个开源项目的 license 会告诉用户他们能做和不能做的（例如使用、修改、重新发布），及告诉贡献者他们允许其他人做的。有许多的办法对开源项目加上许可证，你可以在 [choosealicense.com](http://choosealicense.com/) 读到更多的关于每个许可证的含义。

- Documentation and Wikis：许多大型项目有的不只有一个readme来指导人么如何使用他们的项目。在这种情况下你通常能够发现一个指向库中名为“docs”的另一个文件或文件夹的链接。

##3，发起一个issue
<!--more-->
如果你发现了你正在使用的项目中的一个bug（但是你不知道怎么去修复它），或对文档有不解或对项目有疑问 — 那么创建一个话题吧！这非常容易且一般你不管创建什么话题，你都可能不是唯一一个出现该问题的人，所以其他人可能会发现你的话题很有帮助。关于更多的话题介绍，请查看我们的[Issues guide](https://guides.github.com/features/issues/index.html)。

话题专业提示

- 在建话题之前检查已有的话题：话题重复对双方都无利，所以搜索整个正开放和已关闭的话题以检查你遇到的问题是否已经有人解决了。 
- 务必对自己的问题有清晰的认识：期望的结果是什么？然而却发生了什么？ 详细描述其他人如何重现该问题。
- 在像 [JSFiddle](http://jsfiddle.net/) 或 [CodePen](http://codepen.io/) 类似的平台上重现该问题并给出问题 demo 的链接。 
- 包含一些系统相关的细节，比如用的什么浏览器、库或操作系统及版本号。 
- 在你的话题或在 [Gist](https://gist.github.com/) 里贴出你的错误输出或日志。如果在话题里贴出来，请用三个反引号\`\`\` 包围起来使得能够良好的呈现给大家。

##4，fork项目
点击页面右上角处的fork按钮，这样github就会在你的帐户下fork一个同样的备份。

![](http://img.blog.csdn.net/20130712101759718?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZml2ZTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##5,同步项目代码到本地

fork操作会将该项目添加到你帐户名下的项目主页面，有多种方式同步代码到本地。

- 直接下载源码的zip包(仅仅是代码下载不可同步)

- CloneinDesktop通过github的windows客户端同步到本地9windows下推荐此方式)

- 使用ssh、https、sbuversion等协议同步到本地

![](http://img.blog.csdn.net/20130712102013640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZml2ZTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##6，创建分支

不管你是为了修改bug还是扩展功能，都强烈建议你单独创建一个分支。

##7，修改内容并提交

可以执行变更并创建commit以追踪具体变更内容。
```
git commit −am "adding a smileyface to the documentation." 
```
##8,推送

把主题分支推送到自己的项目fork当中。
```
git push origin your_branch_name
```
##9,创建pull request

最后，需要创建一条pull request。首先，查看repo中的fork，我们可能会看到一条”您最近推送的分支”。如果结果确实如此，则可以选择”比较并pull request”。如果显示其它结果，则可以在下拉菜单中选择自己的分支，随后单击repo界面右上角的“pull request”或者“比较”按钮。

![](http://s7.51cto.com/wyfs01/M01/17/89/wKioJlIeBYqRDoIpAAC_Q5Ddmws348.jpg)

通过“比较并pull request（Compare and Pull Request）”按钮创建pull request

![](http://s5.51cto.com/wyfs01/M01/17/8A/wKioOVIeBa2CLzW7AADpQv-TBFQ827.jpg)

通过分支下拉菜单创建pull request

两种方式都会将我们引导至同一个页面，在这里大家可以创建pull request并在其中添加评论意见。该页面还以直观方式显示出我们所做出的各项变更。这将帮助项目管理员更轻松地查看自己已经完成的工作，同时简化决策过程、更快决定当前内容是否适合提交。如果变更存在问题，管理员可以在评论中提出质疑；管理员还可以要求我们清空pull request并重新提交，然后关闭pull request。

请注意，向项目管理员表达充分的尊重对于开源贡献而言非常重要；毕竟我们总是能够使用代码的分支版本，如果管理员不打算pull我们的变更，这往往与他们的角色定位有关。请记住，根据GitHub员工Zach Holman在《GitHub如何利用GitHub来创建GitHub》一文中所说，pull request实际属于对话的过程。最重要的是认同管理员的处理方式；相对于一味要求对方接受我们提交的内容，大家应当调整心态、只把提交过程视为与编写代码相关的对话通道。

##后记：

第一次参与开源项目的感觉还是蛮不错的，希望作者能将我的提交合并。后续如果有时间，我想基于该项目自己定制一些Android上的图表（现在github上的图表要么是不美观几乎是为了股票的显示而生的那种图、要么是美观但是不易用）或者参考该项目，自己新写一个开源控件。

参考：

[如何在GitHub上协作开发开源项目？](http://os.51cto.com/art/201308/408674.htm)

[在github上参与开源项目日常流程](http://blog.csdn.net/five3/article/details/9307041)

[Contributing to Open Source on GitHub](https://guides.github.com/activities/contributing-to-open-source/index.html)
