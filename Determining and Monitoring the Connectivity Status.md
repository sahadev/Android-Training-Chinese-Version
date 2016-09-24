原文地址：[http://android.xsoftlab.net/training/monitoring-device-state/connectivity-monitoring.html](http://android.xsoftlab.net/training/monitoring-device-state/connectivity-monitoring.html)

通常会有一些后台服务需要连接到网络来更新数据。但是如果没有连接到互联网，或者由于网络太慢而不能完成更新，那么为什么不在连接到网络并且状况良好时再做这些工作呢？

你可以使用[ConnectivityManager](http://android.xsoftlab.net/reference/android/net/ConnectivityManager.html)来检查是否已经连接到互联网，如果连接上了，还可以查询当前连接的类型。

##检测是否联网
如果没有连接到网络，那么就没必要做基于网络的更新了。下面的代码演示了如何通过[ConnectivityManager](http://android.xsoftlab.net/reference/android/net/ConnectivityManager.html)来检查当前的设备是否连接到了网络。
```java
ConnectivityManager cm =
        (ConnectivityManager)context.getSystemService(Context.CONNECTIVITY_SERVICE);
NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
boolean isConnected = activeNetwork != null &&
                      activeNetwork.isConnectedOrConnecting();
```

##检测当前的网络连接类型
有时可能需要检查当前的网络连接类型。

移动设备的网络可能是由蜂窝数据、WiMAX、Wi-Fi及以太网络提供。可以通过查询得知的网络连接类型，基于当前的可用带宽来更改应用的刷新频率。
```java
boolean isWiFi = activeNetwork.getType() == ConnectivityManager.TYPE_WIFI;
```

移动数据所花费的成本要明显的高于Wi-Fi，所以在大多数情况下，当处于移动数据连接时，应当降低更新频率。类似的，较大文件的下载也应当暂停，知道连接到Wi-Fi网络后再继续下载。

因为会中断某些网络任务，所以监听网络状况变化这一点就变得尤为重要了：以便可以在良好的网络状况下恢复任务。

##监听网络连接的变化
当网络状况发生变化时，[ConnectivityManager](http://android.xsoftlab.net/reference/android/net/ConnectivityManager.html)会广播一个[CONNECTIVITY_ACTION](http://android.xsoftlab.net/reference/android/net/ConnectivityManager.html#CONNECTIVITY_ACTION) ("android.net.conn.CONNECTIVITY_CHANGE")的消息。你可以在清单文件中注册一个专门用于监听此消息的广播接收器，以便恢复或暂停后台网络任务。
```xml
<action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
```

由于设备的网络连接时常会发生变化，所以该广播会在每次切换到移动数据或者Wi-Fi情况下都会被触发。因此，最好是为了恢复更新或者下载才用此种方法。通常的做法是，在开始任务之前检查一下网络的连接状况，如果网络不允许，那么使用该方法以便恢复。

这项方法需要动态开启广播接收器，具体的讲解会在下节课描述。