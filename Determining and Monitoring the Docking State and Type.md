原文地址：[http://android.xsoftlab.net/training/monitoring-device-state/docking-monitoring.html](http://android.xsoftlab.net/training/monitoring-device-state/docking-monitoring.html)

Android设备可以被放置在若干种不同的扩展坞中。这些扩展坞包括汽车坞与家庭坞以及数字与模拟坞。其中坞的状态与充电状态非常相近，因为这些坞也提供了充电功能。

APP在何种坞中的运行频率取决于APP自身。你可以在设备处于APP坞时提高运动类APP的更新频率，或者设备处于汽车坞时完全关闭更新，或者也可以在APP在更新交通信息时将更新频率提高至最大。

这些坞的状态也同样通过粘性Intent广播，它可以用来查询是否被放置在了某个坞中，如果被放置了，那么可以查询是何种类型的坞。

##检查当前的坞状态
当前坞的状态被放置在粘性Intent中。因为它是粘性的，所以不需要注册广播接收器。你可以直接通过[registerReceiver()](http://android.xsoftlab.net/reference/android/content/Context.html#registerReceiver(android.content.BroadcastReceiver, android.content.IntentFilter))方法来获得这个Intent。

```java
IntentFilter ifilter = new IntentFilter(Intent.ACTION_DOCK_EVENT);
Intent dockStatus = context.registerReceiver(null, ifilter);
```

接下来则通过该Intent获取当前坞的状态：
```java
int dockState = battery.getIntExtra(EXTRA_DOCK_STATE, -1);
boolean isDocked = dockState != Intent.EXTRA_DOCK_STATE_UNDOCKED;
```

##检查当前坞的类型
如果设备被放置在坞中，那么它可能处于以下类型中：
- Car
- Desk
- 低端桌面坞(模拟)
- 高端桌面坞(数字)

注意后面这两种类型只在Android 11中介绍到，所以只需要统一检查后面这三种类型就可以：
```java
boolean isCar = dockState == EXTRA_DOCK_STATE_CAR;
boolean isDesk = dockState == EXTRA_DOCK_STATE_DESK || 
                 dockState == EXTRA_DOCK_STATE_LE_DESK ||
                 dockState == EXTRA_DOCK_STATE_HE_DESK;
```

##监测坞状态及类型的变化
当设备被放置或移除坞时，系统会广播一个[ACTION_DOCK_EVENT](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_DOCK_EVENT)。为了可以监测坞状态的变化，只需要在清单文件中注册一个广播接收器就可以：
```xml
<action android:name="android.intent.action.ACTION_DOCK_EVENT"/>
```
你可以在对应的广播接收器内获取坞的类型以及状态。