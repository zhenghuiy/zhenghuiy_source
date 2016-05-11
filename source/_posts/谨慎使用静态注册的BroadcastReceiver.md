title: "谨慎使用静态注册的BroadcastReceiver"
date: 2014-08-07 15:26:33
tags: BroadcastReceiver, Android
categories: Android
---
在Android中，BroadcastReceiver有两种注册方式。

第一种是静态注册，在``Androidmanifest.xml``文件中以类似activity注册的方式进行注册。

第二种是动态注册，在``onResume()``中注册，在``onPause()``中注销。

一般而言，我们需要根据具体的使用场景来判断使用何种注册方式。

静态注册的特点是接收设备全局的相应广播，尽管静态注册看起来比较方便，但实际上是十分耗费资源的。

现在使用以下代码进行测试。
```
public class WifiReceiver extends BroadcastReceiver{
public static final String FLAG = "WifiReceiver";

@Override
public void onReceive(Context arg0, Intent arg1) {
if (arg1.getAction().equals(WifiManager.WIFI_STATE_CHANGED_ACTION)) {
int state = arg1.getIntExtra(WifiManager.EXTRA_WIFI_STATE, WifiManager.WIFI_STATE_DISABLED);

if (state == WifiManager.WIFI_STATE_DISABLED) {
Log.v(FLAG,"wifi is disabled!");
} else if (state == WifiManager.WIFI_STATE_ENABLED) {
Log.v(FLAG,"wifi is enabled!");
}
}

}

}
```
<!--more-->
使用一个BroadcastReceiver去接收WIFI状态改变的广播。

AndroidManifest.xml
```
package="com.zhenghuiyan.testawaken"
android:versionCode="1"
android:versionName="1.0" >

android:minSdkVersion="14"
android:targetSdkVersion="14" />

android:allowBackup="true"
android:icon="@drawable/ic_launcher"
android:label="@string/app_name"
android:name=".MyApplication"
android:theme="@style/AppTheme" >
android:name="com.zhenghuiyan.testawaken.MainActivity"
android:label="@string/app_name" >
android:name="com.zhenghuiyan.testawaken.receiver.WifiReceiver">
```


并在配置文件中进行静态注册。

MyApplication.java
```
public class MyApplication extends Application{

    @Override
    public void onCreate() {
        super.onCreate();
        Log.v(this.getClass().getName(),"app is onCreated!");
    }

}
```
监听app启动。

先将应用退出，然后开启/关闭WIFI。运行后的结果是，每次WIFI状态的改变都使APP的onCreate函数被调用，也就是说，不管你需不需要，静态注册的广播被触发时总是会使你的应用被启动。

而应用的启动，无疑是会消耗手机资源的。这里只是监听wifi的状态改变，如果是换成监听手机电量的，那么你的应用将无法真正停止，将十分耗费手机的资源。

应用的开发是为用户服务，创造更好的体验，应该尽量减少资源的消耗，因此，我们应该谨慎使用静态注册的BroadcastReceiver。
