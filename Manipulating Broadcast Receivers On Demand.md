原文地址：[http://android.xsoftlab.net/training/monitoring-device-state/manifest-receivers.html](http://android.xsoftlab.net/training/monitoring-device-state/manifest-receivers.html)

监测设备状态变化最简单的实现方式就是为每种状态都创建一个[广播接收器](http://android.xsoftlab.net/reference/android/content/BroadcastReceiver.html)，然后只需在相应的广播接收器内依据当前的设备状态重新执行各自的任务。

这种方式的负面影响就是在每次广播接收器被触发后，APP都会唤醒设备，这可能会比想象的更加频繁。

一种较好的解决方法就是在运行时关闭或开启广播接收器。这样可以使在清单文件中声明过的广播接收器也可以按需触发。

##动态开启广播接收器
我们可以通过[PackageManager](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html)可以将清单文件中声明的任何组件都切换到开启状态，其中也包括你将要开启或者关闭的广播接收器：
```java
ComponentName receiver = new ComponentName(context, myReceiver.class);
PackageManager pm = context.getPackageManager();
pm.setComponentEnabledSetting(receiver,
        PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
        PackageManager.DONT_KILL_APP)
```

通过使用这项手段，如果你发现网络连接已经断开，那么就可以关闭所有的相关广播接收器，除了监听连接变化的广播接收器。反之，一旦连接到网络，那么则应当停止网络变化的监听。在执行网络任务之前，只需要检查一下是否在线即可。

你也可以使用这种方式来推迟那种需要超宽带宽的网络任务。只需要监听一下网络连接的变化即可，一旦连接到Wi-Fi，那则可以开始进行网络下载。