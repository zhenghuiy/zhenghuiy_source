title: '[转]QUIC和TCP'
tags:
  - quic
  - TCP
  - ''
categories:
  - 计算机网络
date: 2016-08-19 12:18:00
---
## 前言
这几天在研究部门内部的自定义协议，因此去了解Google的QUIC协议的特性，下面这篇文章将QUIC协议和TCP协议做了比较，个人比较喜欢作者的阐述，特地转到博客中来。

原文作者：henrystark henrystark@126.com
原文地址：http://blog.chinaunix.net/uid-28387257-id-4335291.html

## 正文
### 0.写作目的
QUIC由Google提出，基于UDP，用于加快网络速率。常用来和基于TCP的SPDY比较。Google在传输层、应用层或其他方面做出的提升网络质量的贡献令人佩服。本篇blog将论述QUIC的起源、优缺点，以及TCP存在的问题。

### 1.引言
Why QUIC is necessary? 每个接触QUIC的programmer总会这样问。答案也很简单：SPDY、TCP不够好！不过这样说太肤浅了，下面我来分析本质原因【引 1】。基于一条TCP连接的SPDY复用连接会面临这样的情况：当有丢包发生时，所有连接都将阻塞，这是由TCP的拥塞控制特性决定的【引 2 3】。丢包必须恢复，而恢复过程中，或早或晚，滑动窗口总有停等的时刻，耗费一个RTT。在广域网上，一个RTT相当于50-100ms。相比较而言，当x条并行HTTP连接中，有一条丢包，只会阻塞一条。

QUIC是和HTTP同一层的应用层协议，其核心是将丢包控制工作转移到应用层【注 1】。由于QUIC基于UDP，可以不理会丢包，快速投递，再用丢包恢复方法保证可靠性。除此之外，基于一条TCP连接的SPDY和多条并行HTTP连接相比，没有优势可言。多条连接中，每个连接都有一个拥塞窗口，不受彼此丢包影响。Google希望通过QUIC更好地处理多条连接下的拥塞状况。

### 2.TCP的症结
以上所述其实反映了TCP基于窗口的拥塞控制策略的问题。TCP的核心在于“丢包必须恢复”，正是这种丢包恢复导致传输速率降低。而除此之外，TCP拥塞控制也存在粒度不精细等问题。举例而言，往年有一道很好的面试题：早期网络中，为什么蚂蚁等下载器比网页下载快？答案是下载器使用多线程，多连接下载，而网页下载往往使用单连接。也许回答到这种程度，大部分人已经满意了。但往下问，还有更深刻的内涵：为什么多连接比单连接快？ISP给用户分配的带宽不是固定吗？
<!-- more -->
我想，到了这种程度，大多数人回答不出所以然来。可以从两方面解释：1.多连接下载中，每个连接负责下载不同的offset range 2.TCP基于窗口的拥塞控制不够好，多个连接更具有侵略性，能占据更多的带宽。关于第一点，学过网络编程的人多多少少知道。第二点，则需要详细解释了。TCP用congestion window（cwnd）来控制发送速率，发送初期，cwnd二次增长，丢包时减半，之后线性增长。TCP的初始窗口为2MSS，下限为1MSS，当多条流存在时，即便丢包，窗口减幅也有限。现有用户带宽并不高，带宽时延积BDP也不会很大，100MSS的cwnd足够大了。如果有10条TCP连接并发下载，那么最差的cwnd之和也是10MSS，当然比只有一条连接时，窗口为1MSS要好。

也许说到这种程度就可以了，但是我们还可以更深一步：为什么TCP基于窗口的拥塞控制行为会有这样的缺陷？怎么改进？首先需要明确两个概念：窗口、段，TCP中，窗口是以段（segment，MSS）为单位的，一个MSS通常为1460Bytes，cwnd就是x*MSS。再来看多流并发的问题：多连接下载中，假设流数为n，那么总窗口最低降为n个MSS。这对于并行下载来说是优势，因为最低窗口也很高。那么，从反方面看呢？你觉得并发下载好，是因为多流可以提升带宽利用率，换言之，多流窗口之和没有超过BDP。那是否存在这样的状况：多流最低窗口之和远远大于BDP？不幸的是，存在。这种状况诞生的场景是：并发量高而BDP小的网络，其中最典型的就是数据中心网络。数据中心网络具有的特征为：高带宽、低延时、高并发、BDP小，交换机缓存有限。实际上，当多流窗口之和远大于BDP时，会频繁引发网络拥塞，造成超时，也就是著名的TCP吞吐率崩塌问题，也称为Incast问题。

说这么多，意义何在呢？启发TCP下一步的改进方向，基于窗口的拥塞控制能力有限，或许已经到了改变的时候了【注 2】。TCP下一步的改变方向或许是“速率控制”，而非“窗口控制”。丢包时，“速率减半”。RCP等协议是基于速率的。表面看来，这两种控制方式并无差别。但是速率控制能精确控制发送速率，而不降低负载率。举例来说，如果窗口到了1MSS，还是不能避免丢包拥塞超时，那怎么办呢？很多人会说减小MSS，可是这会降低带宽利用率，举例来说，当MSS为500时，带宽利用率最大为40/540，因为包头的40字节是无用的。比40/1500低了很多。而速率控制方法可以在不降低带宽利用率的情况下控制发送速率。其根本区别在于：基于窗口的控制策略是burst投递，也就是突发投递【注 3】。而基于速率的控制可以渐缓投递。

讲到这里，是否有醍醐灌顶的感觉？传输控制的本质已经在你眼前揭开了，希望你得到点什么，不过那并不是属于你自己的东西，而是无数前辈的结晶，我只是按我的理解方式叙述而已。

### 3.QUIC的机制
QUIC具有的特性如下【引 4】：

0-RTT connections
Packet pacing that reduces packet loss
Forward error correction that reduces retransmission latency
Adaptive congestion control (friendly to TCP), reducing reconnections for mobile clients
Encryption equivalent to TLS
Chrome can talk QUIC to Google today 
其中，QUIC对packet loss和拥塞避免的处理最值得关注。下面我来介绍这两部分处理【引 5】。

packet loss：QUIC丢包恢复有两种办法，前向纠错（FEC）和重传。前向纠错可以减少重传，但需要在包中添加冗余信息，用XOR实现【注 4】。如果前向纠错不能恢复包，就启用重传，重传的不是旧包，而是重新构造的包。
congestion avoidance：TCP使用了窗口来进行拥塞控制，QUIC使用带宽探测器、监察delay变化并使用pacing来减少丢包【注 5】。当接收端判定丢包后，发送NACK给对端，通知丢包事件。此后进行的速率降低工作类似于TCP，保持对TCP的友好性。

（本篇只是开头，QUIC的源码我还没把握体系，关于QUIC的内容会逐步补充，待续。。。）

引用
1. QUIC FAQ for Geeks。
https://docs.google.com/document/d/1lmL9EF6qKrk7gbazY8bIdvq3Pno2Xj_l_YShP40GLQE/edit。参见“Why isn’t SPDY over TCP good enough?”。
2. Head-of-line blocking。http://en.wikipedia.org/wiki/Head-of-line_blocking。
3. HTTP pipeline。http://en.wikipedia.org/wiki/HTTP_pipelining。HTTP pipeling特性支持把多条HTTP连接封装到一条TCP连接中，但Head-of-line blocking会严重降低这种复用连接的性能。这本来是为了减少握手交互时间而提出的特性改进，却带来了上述的性能降低问题。
4. QUIC特性。http://www.infoq.com/news/2014/02/quic。
5. QUIC dealing with packet loss。https://docs.google.com/document/d/1RNHkx_VvKWyWg6Lr8SZ-saqsQx7rFV-ev2jRFUoVD34/edit#。

注解
1. 将拥塞控制分离的想法很早就有人提出，基于传输层的可靠协议UDT或TCP甚至也可以分离到内核之外，成为用户空间协议栈。
2. If you understand these words, congratulations! This thought is very meaningful, in other words, it may be epochal. You deserve it. But don't say the thought owe to you, because it's not my own, it's proposed by researchers. Thanks to my friend, "jedihy" (http://c2fun.cn 8th blogs of him), I got the idea from him.
3. 参阅TCP Pacing。
4. 前向纠错并不是一种新手段，基于XOR的包恢复策略屡见不鲜，参阅“链路层 network coding”。
5. pacing是一个重要策略，如果你理解了第一部分“TCP的症结”，就会知道它有多重要。