原文地址：[https://developer.android.com/training/run-background-service/report-status.html](https://developer.android.com/training/run-background-service/report-status.html)

这节课主要学习如何将IntentService中的执行结果返回给请求点。一种推荐的方式就是使用 [LocalBroadcastManager](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)来实现，它会将所广播的[Intent](https://developer.android.com/reference/android/content/Intent.html)限制在APP内部。

##发送IntentService的处理结果
为了可以将IntentService的处理结果发送给其它组件，首先需要创建一个Intent对象，并将执行结果放入该Intent对象内。

接下来要做的就是，将刚才创建好的Intent通过[LocalBroadcastManager.sendBroadcast()](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html#sendBroadcast(android.content.Intent))发送出去。但凡是对该Intent注册了的，那么发送该Intent都会收到结果。可以通过[getInstance()](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html#getInstance(android.content.Context))获取[LocalBroadcastManager](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)的实例。

例子如下：
```java
public final class Constants {
    ...
    // Defines a custom Intent action
    public static final String BROADCAST_ACTION =
        "com.example.android.threadsample.BROADCAST";
    ...
    // Defines the key for the status "extra" in an Intent
    public static final String EXTENDED_DATA_STATUS =
        "com.example.android.threadsample.STATUS";
    ...
}
public class RSSPullService extends IntentService {
...
    /*
     * Creates a new Intent containing a Uri object
     * BROADCAST_ACTION is a custom Intent action
     */
    Intent localIntent =
            new Intent(Constants.BROADCAST_ACTION)
            // Puts the status into the Intent
            .putExtra(Constants.EXTENDED_DATA_STATUS, status);
    // Broadcasts the Intent to receivers in this app.
    LocalBroadcastManager.getInstance(this).sendBroadcast(localIntent);
...
}
```

下一步就是如何处理接收到的Intent对象了。

##接收IntentService的处理结果
如果需要接收广播出来的Intent，那么就需要用到[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)了。在[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)的实现类中重写[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)的[onReceive()](https://developer.android.com/reference/android/content/BroadcastReceiver.html#onReceive(android.content.Context,%20android.content.Intent))回调方法。当[LocalBroadcastManager](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)将相应的Intent对象广播出来后，那么该方法就会被自动回调。

举个例子：
```java
// Broadcast receiver for receiving status updates from the IntentService
private class ResponseReceiver extends BroadcastReceiver
{
    // Prevents instantiation
    private DownloadStateReceiver() {
    }
    // Called when the BroadcastReceiver gets an Intent it's registered to receive
    @
    public void onReceive(Context context, Intent intent) {
...
        /*
         * Handle Intents here.
         */
...
    }
}
```

一旦定义好了[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)，那么就可以为其定义指定的意图过滤器了。要做到这些，需要创建一个IntentFilter。下面的代码演示了如何定义一个过滤器：
```java
// Class that displays photos
public class DisplayActivity extends FragmentActivity {
    ...
    public void onCreate(Bundle stateBundle) {
        ...
        super.onCreate(stateBundle);
        ...
        // The filter's action is BROADCAST_ACTION
        IntentFilter mStatusIntentFilter = new IntentFilter(
                Constants.BROADCAST_ACTION);

        // Adds a data filter for the HTTP scheme
        mStatusIntentFilter.addDataScheme("http");

```

为了将BroadcastReceiver以及IntentFilter注册到系统，需要先获取[LocalBroadcastManager](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)的实例，然后再调用它的[registerReceiver()](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html#registerReceiver(android.content.BroadcastReceiver,%20android.content.IntentFilter))方法。下面的代码演示了这个过程：
```java
        // Instantiates a new DownloadStateReceiver
        DownloadStateReceiver mDownloadStateReceiver =
                new DownloadStateReceiver();
        // Registers the DownloadStateReceiver and its intent filters
        LocalBroadcastManager.getInstance(this).registerReceiver(
                mDownloadStateReceiver,
                mStatusIntentFilter);
        ...
```

BroadcastReceiver可以同时处理多种类型的Intent对象，这项特性可以为每种Action定义不同的代码，而不需要专门去定义BroadcastReceiver。为同一个BroadcastReceiver定义另外的IntentFilter，只需再创建一个IntentFilter，然后再次注册一下就好：
```java
        /*
         * Instantiates a new action filter.
         * No data filter is needed.
         */
        statusIntentFilter = new IntentFilter(Constants.ACTION_ZOOM_IMAGE);
        ...
        // Registers the receiver with the new filter
        LocalBroadcastManager.getInstance(getActivity()).registerReceiver(
                mDownloadStateReceiver,
                mIntentFilter);
```

发送广播Intent并不会启动或者恢复Activity。就算是APP处于挂起状态(处于后台)也同样会接收到Intent。如果APP处于挂起状态时，有任务完成需要通知用户，那么可以使用[Notification](https://developer.android.com/reference/android/app/Notification.html)。绝不要启动Activity来响应接收到的广播Intent。