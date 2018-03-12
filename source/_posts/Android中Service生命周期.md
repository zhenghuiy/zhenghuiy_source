title: Android中Service生命周期
tags: 'Android, Service, 生命周期'
categories: Android
date: 2014-05-18 09:57:36
---
这几天面试的时候，反复被问到一个关于Service的问题。

在大二暑假实习的时候，做了一个APP。有一个应用场景是，需要开机启动一个Service，在Service中另开一个线程，去对比用户配置中的时间，作出及时提醒。

然后面试的时候在描述该做法时就被问到一个问题，如果Service被系统或者其他应用kill了怎么办？我当时的回答是，在onDestroy中去处理。面试官说，onDestroy并不会被调用。

面试的详情暂且不表，在后期会专门写面经。现在讨论这个问题，Service被kill后生命周期是怎样的。


OK，用代码说话。

### 1,新建一个项目，项目中有一个Activity,一个Service。在Activity的button的监听处理中去开启这个Service
```Java
package com.zhenghuiy.killedservicelifecycletest;

import android.os.Bundle;
import android.app.Activity;
import android.content.Intent;
import android.view.Menu;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity implements OnClickListener{
    private Button startServiceBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initViews();
    }
    
    private void initViews() {
        startServiceBtn = (Button)findViewById(R.id.startService);
        startServiceBtn.setOnClickListener(this);
    }
    
    @Override
    public void onClick(View view) {
        if(view.getId() == R.id.startService){
        Intent intent = new Intent();
        intent.setClass(this, MyService.class);
        this.startService(intent);
    }
    
    }

}
```
### 2,重写Service的大部分函数，具体看注释
<!--more-->
```Java
package com.zhenghuiy.killedservicelifecycletest;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.util.Log;

public class MyService extends Service implements Runnable{
    /*
    * Service当以bindService的形式调用时，会调用onBind
    * 当以startService,则调用onStartCommand
    * 另外，onBind是一个抽象函数，必须重写
    * */
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        showLog("onStartCommand is called");
        
        //Service运行在UI主线程，为了避免因堵塞而被关闭，另开一个线程
        new Thread(this).start();
        
        return super.onStartCommand(intent, flags, startId);
    }
    
    @Override
    public IBinder onBind(Intent itent) {
        showLog("onBind is called");
        return null;
    }
    
    @Override
    public void onCreate() {
        super.onCreate();
        showLog("onCreate is called");
    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        showLog("onDestroy is called");
    }
    
    /*
    * onStart方法已经过时
    * 在2.0之后的版本使用onStartCommand
    * */
    @Override
    @Deprecated
    public void onStart(Intent intent, int startId) {
        super.onStart(intent, startId);
        showLog("onStart is called,the Intent action is"+intent.getAction());
    }
    
    @Override
    public void onTaskRemoved(Intent rootIntent) {
        super.onTaskRemoved(rootIntent);
        showLog("onTaskRemoved is called,the Intent action is"+rootIntent.getAction());
    }
    
    @Override
    public void onTrimMemory(int level) {
        super.onTrimMemory(level);
        showLog("onTrimMemory is called,the level is"+level);
    }
    
    private void showLog(String text){
        Log.v(this.getClass().getName(),text);
    }
    
    @Override
    public void run() {
        while(true){}
    
    }
}
```
### 3,用真机测试

运行后点击button，启动service，此时以下函数被调用：

![](http://zhenghuiy-blog.qiniudn.com/1.jpg)


点击home回到手机桌面，此时该Service仍然在后台运行。onTrimMemory被调用

![](http://zhenghuiy-blog.qiniudn.com/2.jpg)

我使用的手机是华为3C。进入系统的setting后，可以看到显示该应用有一个进程和一个服务在运行中。

在设置里的应用管理那点击“停止”。onDestroy被调用

![](http://zhenghuiy-blog.qiniudn.com/3.jpg)

说明，当服务被系统自动或手动（人为的在设置里停止）停止时，仍然会正常走完其生命周期。

### 4，测试使用其他应用，比如“腾讯手机管家”去停止Service.

前面同样的过程就不赘述，当Service在后台运行的时候，使用手机管家去“一键加速”。

可以在设置——》应用管理 里看到，原来该测试应用的item显示“有1个进程和1个服务在运行”变成“有0个进程和1个服务在运行”。再刷新一遍，就发现，该应用已经不在运行中的列表里了。

并且，logcat里始终没有打印“onDestroy is called”.

结论是，其他“管家式”应用“清理”的方法是，直接kill该进程。此时，Service不会走正常的生命周期，也就是onDestroy未被调用。

### 5，回到问题本身

当时面试官问出这个问题，我的回答是：

一种方法是，使用服务器进行推送。如果客户端有响应，说明Service存活。如果没有响应，就启动Service.

另一种方法是，将该Service独立出来，运行在另一个进程中。（但是仔细想想，这个方法并不能避免Service被kill，因此不算正确答案）。

不知道其他方法还有什么？