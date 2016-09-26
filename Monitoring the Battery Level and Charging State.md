原文地址：[http://android.xsoftlab.net/training/monitoring-device-state/index.html](http://android.xsoftlab.net/training/monitoring-device-state/index.html)

#引言
作为一款优秀的APP应用，应该想方设法的降低电量的消耗。通过这节课的学习，你将有能力使APP可以基于设备的状态来调整它的功能以及行为。

我们可以通过比如在断开连接时关闭后台服务，或者在电量低的时候降低更新的频率等等手段来将耗电量降到最低。

#监测电池电量及充电状态
当你在改变后台更新频次时，检查当前的电池电量以及充电状态是我们先要做的。

应用程序的更新频率取决于电池的电量以及充电状态。由于设备处于充电状态时应用的耗电量几乎可以忽略，所以，在设备连接到充电器时，你可以将应用的刷新频率开到最大，如果设备没有在充电，那么降低更新频率可以延长电池的使命时间。

##检查当前的充电状态
首先我们需要检查当前的充电状态。[BatteryManager](http://android.xsoftlab.net/reference/android/os/BatteryManager.html)会将电池信息以及充电信息通过粘性Intent将其广播。

因为是粘性Intent，所以不需要注册BroadcastReceiver，只需要在调用registerReceiver()时传一个null就可以，当前的电池状态由该方法直接返回。你也可以在这里传递一个BroadcastReceiver对象，但是我们接下来的处理方式并不是在其中做的，所以这并不是必须的。

```java
IntentFilter ifilter = new IntentFilter(Intent.ACTION_BATTERY_CHANGED);
Intent batteryStatus = context.registerReceiver(null, ifilter);
```

如果设备当前处于充电状态，那么可以获得当前的充电状态，无论它是通过USB还是通过AC适配器充电的。
```java
// Are we charging / charged?
int status = batteryStatus.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                     status == BatteryManager.BATTERY_STATUS_FULL;
// How are we charging?
int chargePlug = batteryStatus.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);
boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;
boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;
```

通常的做法是：应当是在连接到AC电源适配器时，将后台的更新频率加到最大，如果当前处于USB状态，这个频率应当适当降低，如果断开充电，则应当进一步降低。

##监测充电状态的变化
设备的充电状态很容易随着充电器的插入、拔出而发生变化。所以监测充电状态的变化可以相应的更新应用的刷新频率。

当设备插上充电器或是拔出充电器时，[BatteryManager](http://android.xsoftlab.net/reference/android/os/BatteryManager.html)都会广播一个Action，所以应当注册一个BroadcastReceiver用来监听这些事件。在清单文件中需要定义[ACTION_POWER_CONNECTED](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_POWER_CONNECTED)及[ACTION_POWER_DISCONNECTED](ACTION_POWER_DISCONNECTED)的意图过滤器。
```xml
<receiver android:name=".PowerConnectionReceiver">
  <intent-filter>
    <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
    <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED"/>
  </intent-filter>
</receiver>
```

在该BroadcastReceiver内，你可以获取当前的充电状态：
```java
public class PowerConnectionReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) { 
        int status = intent.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
        boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                            status == BatteryManager.BATTERY_STATUS_FULL;
    
        int chargePlug = intent.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);
        boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;
        boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;
    }
}
```

##检查电池的剩余电量
在一些情况下还需要检查设备的剩余电量。当电量较低时可能需要降低应用的后台服务频率。

你可以通过以下方式获得设备的剩余电量：
```java
int level = batteryStatus.getIntExtra(BatteryManager.EXTRA_LEVEL, -1);
int scale = batteryStatus.getIntExtra(BatteryManager.EXTRA_SCALE, -1);
float batteryPct = level / (float)scale;
```

##监测电量的重要变化
应用不能一直连续不断的监听电池的状态。

通常来说，一直不断的监听电池电量会使监听电池的意义大于应用的实际意义，所以最好是只监听一些比较重要的变更事件。

下面的清单文件摘自一段广播接收器内。该广播接收器会在电池的电量很低时或者是在电量恢复到安全水平时被触发。它监听了两个事件：[ACTION_BATTERY_LOW](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_BATTERY_LOW)及[ACTION_BATTERY_OKAY](http://android.xsoftlab.net/reference/android/content/Intent.html#ACTION_BATTERY_OKAY).

```xml
<receiver android:name=".BatteryLevelReceiver">
<intent-filter>
  <action android:name="android.intent.action.ACTION_BATTERY_LOW"/>
  <action android:name="android.intent.action.ACTION_BATTERY_OKAY"/>
  </intent-filter>
</receiver>
```

通常情况下，在电量很低时要关闭所有的后台更新。在你使用应用之前，手机关闭了，那么应用的数据是否是最新的就没那么重要了。

在很多情况下，手机充电时是被放在一个固定的位置上的。下节课我们将会学习如何检查设备的放置环境以及如何监测设备的放置状态。