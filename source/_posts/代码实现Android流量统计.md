title: "代码实现Android流量统计"
date: 2014-07-23 12:16:15
tags: Android, Android流量统计， Android流量
categories: Android
---
##概述

尽管现在WIFI的覆盖范围越来越广，但是设备（在这里设备指手机或平板）流量的使用仍然是用户很关注的一个点。因此，在APP中加入流量统计模块对于提升用户体验（让用户可以知晓使用了多少流量，可以使用户使用APP时更加放心）有很大帮助。

Android使用的是Linux内核，在Linux系统里，所有信息都是以文件的形式存在的，因此在Android中，所有应用的流量信息都将被保存在操作系统的文件中。

对于Android2.2之前，流量信息存储在proc/net/dev（或者 proc/self/net/dev）文件下，我们解析文件即可得到流量信息。

在Android2.2（API Level 8）之后，系统提供了TrafficStats类，我们可以通过该类来获取流量信息。需要注意一点的是，有些设备不支持流量统计，具体表现是，当调用TrafficStats的方法时将返回TrafficStats.UNSUPPORTED，这种情况下，我们需要自己去解析/sys/class/net/  下的log文件。

##方法

以下是TrafficStats类的一些主要方法（摘录自[android 官方API文档](http://developer.android.com/reference/android/net/TrafficStats.html)）：

    static long getMobileRxBytes()
    Return number of bytes received across mobile networks since device boot.
    
    static long getMobileRxPackets()
    Return number of packets received across mobile networks since device boot.
    
    static long getMobileTxBytes()
    Return number of bytes transmitted across mobile networks since device boot.
    
    static long getMobileTxPackets()
    Return number of packets transmitted across mobile networks since device boot.
    
    static long getTotalRxBytes()
    Return number of bytes received since device boot.
    
    static long getTotalRxPackets()
    Return number of packets received since device boot.
    
    static long getTotalTxBytes()
    Return number of bytes transmitted since device boot.
    
    static long getTotalTxPackets()
    Return number of packets transmitted since device boot.
    
    static long getUidRxBytes(int uid)
    Return number of bytes received by the given UID since device boot.
    
    static long getUidRxPackets(int uid)
    Return number of packets received by the given UID since device boot.
    
    static long getUidTcpRxBytes(int uid)
    This method was deprecated in API level 18. Starting in JELLY_BEAN_MR2, transport layer statistics are no longer available, and will always return UNSUPPORTED.
    
    static long getUidTxPackets(int uid)
    Return number of packets transmitted by the given UID since device boot.

<!--more-->	
##流程

流量统计可以分为两个方面，一个是手机整体流量统计和单个app的流量统计。

下面总结一下统计的流程。

###手机整体流量：
1， 系统中存储的流量信息在关机之后会重置，因此我们应该使用一个``BroadcastReceiver``去接收关机广播，在每次关机的时候存储流量信息；

2， 在获取数据前判断该设备是否支持TrafficStats，即方法返回值是否等于``TrafficStats.UNSUPPORTED``，如果不支持，则将下面的各个方法通过解析``/sys/class/net/ `` 下的log文件自己去实现。

3， 当前通过2G/3G接收到总字节数为：``TrafficStats.getMobileRxBytes()``+之前的存储

4， 当前通过2G/3G发出总字节数为：``TrafficStats.getMobileTxBytes()``+之前的存储

5， 当前通过WIFI接收到总字节数为：
使用一个``BroadcastReceiver``监听WIFI状态变化，WIFI断开连接时``TrafficStats.getTotalRxBytes()`` – WIFI连接时``TrafficStats.getTotalRxBytes()``
= 本次WIFI接收流量。再加上之前的存储，等于总的流量。

6， 当前通过WIFI发出的总字节数为：
使用一个``BroadcastReceiver``监听WIFI状态变化，WIFI断开连接时``TrafficStats.getTotalTxBytes()`` – WIFI连接时``TrafficStats.getTotalTxBytes()``
= 本次WIFI接收流量。再加上之前的存储，等于总的流量。
 
###单个APP的流量：
1，系统中存储的流量信息在关机之后会重置，因此我们应该使用一个``BroadcastReceiver``去接收关机广播，在每次关机的时候存储流量信息；

2，在获取数据前判断该设备是否支持TrafficStats，即方法返回值是否等于``TrafficStats.UNSUPPORTED``，如果不支持，则将下面的各个方法通过解析``/sys/class/net/  ``下的log文件自己去实现。

3，获取APP的UID：
```
PackageManager pm = getPackageManager();
ApplicationInfo ai = pm.getApplicationInfo(PACKAGE_NAME, PackageManager.GET_ACTIVITIES);
Log.d("UID IS:" + ai.uid);
```
4, 该UID的APP通过WIFI接收到总字节数为：
使用一个``BroadcastReceiver``监听WIFI状态变化，WIFI断开连接时``TrafficStats.getUidRxBytes(int uid)`` – WIFI开始连接时``TrafficStats.getUidRxBytes(int uid) ``= 本次WIFI接收流量。再加上之前的存储，等于总的流量。

5, 该UID的APP通过WIFI发出总字节数为：
使用一个BroadcastReceiver监听WIFI状态变化，WIFI断开连接时``TrafficStats.getUidTxBytes(int uid)`` – WIFI开始连接时``TrafficStats.getUidTxBytes(int uid)`` = 本次WIFI接收流量。再加上之前的存储，等于总的流量。

6，该UID的APP通过2G/3G接收到总字节数为：``TrafficStats.getUidRxBytes(int uid)`` – WIFI的流量 + 之前的存储

7，该UID的APP通过2G/3G发送的总字节数为：``TrafficStats.getUidTxBytes(int uid)`` – WIFI的流量 + 之前的存储
 