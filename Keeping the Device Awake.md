原文地址：[http://android.xsoftlab.net/training/scheduling/index.html](http://android.xsoftlab.net/training/scheduling/index.html)

#引言
当Android设备处于闲置状态时，它首先屏幕会变暗，接着会关闭屏幕，最后会将CPU关闭。这些措施可以防止设备的电量迅速被耗尽。但是当APP需要的话，还是会有例外的情况：

- 比如游戏类APP或者视频类APP需要保持屏幕常亮。
- 有一部分APP或许不需要屏幕保持常亮，但是它们会要求CPU保持运转，直到它们的任务执行完毕。

这节课主要学习如何在需要的时候还能保持设备的唤醒状态而又不至于非常耗电。

#保持设备的唤醒状态
为了避免迅速将电量耗光，Android设备会在进入闲置状态后迅速的进入睡眠状态。不过，还是有一些情况需要保持设备的唤醒状态或者是保持CPU的唤醒状态以便保证一些任务完成。

什么样的方式取决于APP的需求。不过，有一条常规就是尽量采取最轻量级的方法，以便尽可能小的消耗系统资源。下面的部分将会学习如何处理APP与系统默认睡着行为的矛盾所在。

##保持常亮
某些APP比如游戏类APP或者视频类APP需要保持屏幕常亮。要做到这一点只需要在Activity中使用[FLAG_KEEP_SCREEN_ON](http://android.xsoftlab.net/reference/android/view/WindowManager.LayoutParams.html#FLAG_KEEP_SCREEN_ON)就可以，不过千万不要在服务或者其它组件中使用该标志：
```java
public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
  }
  ...
}
```

这种方法的优势在于：它不要指定特殊权限，系统会将APP之前的切换状态处理好，也不需要担心有关释放无用资源的问题。

另一个实现方式就是在应用的布局文件中使用[android:keepScreenOn](http://android.xsoftlab.net/reference/android/R.attr.html#keepScreenOn)属性：
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:keepScreenOn="true">
    ...
</RelativeLayout>
```

[android:keepScreenOn="true"](http://android.xsoftlab.net/reference/android/R.attr.html#keepScreenOn)的作用效果与使用[FLAG_KEEP_SCREEN_ON](http://android.xsoftlab.net/reference/android/view/WindowManager.LayoutParams.html#FLAG_KEEP_SCREEN_ON)的效果等同。你可以选择最合适的方式。使用标志的优势在于可以动态的清除该标志的状态，以便于屏幕可以转入关闭状态。

> **Note:** 除非确认屏幕不再需要保持常亮，否则不需要我们自己专门去清除该标志。WindowManager会严格把关这些事情：APP转入后台，APP转入前台。但是如果你明确要清除该标志以便屏幕可以转入关闭状态，那么可以使用[clearFlags()](http://android.xsoftlab.net/reference/android/view/Window.html#clearFlags(int))：getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)

##保持CPU的运行
如果APP希望在系统转入睡眠状态之前完成一些事情，那么可以使用一个名为WakeLock的PowerManager系统服务特性。WakeLock可以使APP控制设备的电源状态。

创建并持有WakeLock可以与电源直接交互。这样的话只应当在必要的时候再使用WakeLock。绝不要在Activity中使用WakeLock。就像上面说的那样，如果希望保持屏幕常亮，只需要使用[FLAG_KEEP_SCREEN_ON](http://android.xsoftlab.net/reference/android/view/WindowManager.LayoutParams.html#FLAG_KEEP_SCREEN_ON)就可以。

一个使用WakeLock的合理情况就是后台服务。再强调一次，在使用时应当最小限度的使用该标志，因为它会直接影响到电池的电量。

如果要使用WakeLock，首先需要在清单文件中添加WakeLock的权限：
```xml
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

如果APP包括了一个与服务做相同工作的广播接收器，那么可以通过[WakefulBroadcastReceiver](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html)来管理WakeLock。这是最合适的方案了。如果APP没有那样的情况，那么就再使用下面的方法：
```java
PowerManager powerManager = (PowerManager) getSystemService(POWER_SERVICE);
Wakelock wakeLock = powerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
        "MyWakelockTag");
wakeLock.acquire();
```

如果要释放WakeLock，调用[wakelock.release()](http://android.xsoftlab.net/reference/android/os/PowerManager.WakeLock.html#release())就好。它会释放你所持有的CPU资源。在任务完成后做这项工作是很重要的，因为这可以放置电池电量被迅速耗光。

##使用[WakefulBroadcastReceiver](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html)

广播接收器与服务的结合使用可以使你管理后台任务的生命周期。

[WakefulBroadcastReceiver](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html)是一种特殊的广播接收器：它可以创建、管理APP的[PARTIAL_WAKE_LOCK](http://android.xsoftlab.net/reference/android/os/PowerManager.html#PARTIAL_WAKE_LOCK)。[WakefulBroadcastReceiver](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html)会将关闭工作发送给服务(通常是IntentService)，这时设备将要进入睡眠状态。如果你没有持有WakeLock，那么在工作完成之前设备可能就会转入睡眠状态。这就会导致任务不能及时完成，这并不是我们想要的。

使用[WakefulBroadcastReceiver](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html)的第一步就是将其添加到清单文件中，与其它广播接收器的添加方式一样：
```xml
<receiver android:name=".MyWakefulReceiver"></receiver>
```

第二步就是使用[startWakefulService()](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html#startWakefulService(android.content.Context, android.content.Intent))方法来启动MyIntentService。这个方法与[startService()](http://android.xsoftlab.net/reference/android/content/Context.html#startService(android.content.Intent))很相似，除了在启动时[WakefulBroadcastReceiver](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html)持有了一个WakeLock。这里在[startWakefulService()](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html#startWakefulService(android.content.Context, android.content.Intent))中所使用的Intent被携带了一个WakeLock。
```java
public class MyWakefulReceiver extends WakefulBroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // Start the service, keeping the device awake while the service is
        // launching. This is the Intent to deliver to the service.
        Intent service = new Intent(context, MyIntentService.class);
        startWakefulService(context, service);
    }
}
```

当服务结束时，使用[MyWakefulReceiver.completeWakefulIntent()](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html#completeWakefulIntent(android.content.Intent))将WakeLock释放。[completeWakefulIntent()](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html#completeWakefulIntent(android.content.Intent))方法使用了被[WakefulBroadcastReceiver](http://android.xsoftlab.net/reference/android/support/v4/content/WakefulBroadcastReceiver.html)传递过来的Intent对象：
```java
public class MyIntentService extends IntentService {
    public static final int NOTIFICATION_ID = 1;
    private NotificationManager mNotificationManager;
    NotificationCompat.Builder builder;
    public MyIntentService() {
        super("MyIntentService");
    }
    @Override
    protected void onHandleIntent(Intent intent) {
        Bundle extras = intent.getExtras();
        // Do the work that requires your app to keep the CPU running.
        // ...
        // Release the wake lock provided by the WakefulBroadcastReceiver.
        MyWakefulReceiver.completeWakefulIntent(intent);
    }
}
```

