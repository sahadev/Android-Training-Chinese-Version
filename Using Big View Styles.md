原文地址：[http://android.xsoftlab.net/training/notify-user/expanded.html#big-view](http://android.xsoftlab.net/training/notify-user/expanded.html#big-view)

通知在通知栏中以两种风格呈现：正常视图与大视图。只有在通知展开的时候才会展示大视图。这只有在通知处于通知栏顶部时或者用户点击了通知时才会出现。

大视图于Android 4.1开始出现，并且不支持老版本。这节课将会介绍如何使用大视图的通知。

这是正常视图的示例：

![](http://android.xsoftlab.net/images/training/notifications-normalview.png)

下面是大风格的示例：

[http://android.xsoftlab.net/images/training/notifications-bigview.png](http://android.xsoftlab.net/images/training/notifications-bigview.png)

这节课所展示的示例程序将会以正常视图和大视图两种方式为用户提供相同的功能：

- 可以延迟提醒或者取消通知。
- 以一种方式展示提醒文本给用户。

正常视图以Activity的形式提供了以上功能。要在设计通知的时候记住这一点：首先在正常视图中提供各种功能，因为这样可以有很多用户与通知产生交互。

##设置通知启动Activity
示例应用程序使用[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)的子类PingService来构造并发布通知。

在下面的代码段中，[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)的方法[onHandleIntent()](http://android.xsoftlab.net/reference/android/app/IntentService.html#onHandleIntent(android.content.Intent))指明了一个新的Activity会在用户点击通知的时候启动。[setContentIntent()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentIntent(android.app.PendingIntent))方法中设置了在用户点击通知时被发布的PendingIntent，因此可以启动Activity。
```java
Intent resultIntent = new Intent(this, ResultActivity.class);
resultIntent.putExtra(CommonConstants.EXTRA_MESSAGE, msg);
resultIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | 
        Intent.FLAG_ACTIVITY_CLEAR_TASK);
     
// Because clicking the notification launches a new ("special") activity, 
// there's no need to create an artificial back stack.
PendingIntent resultPendingIntent =
         PendingIntent.getActivity(
         this,
         0,
         resultIntent,
         PendingIntent.FLAG_UPDATE_CURRENT
);
// This sets the pending intent that should be fired when the user clicks the
// notification. Clicking the notification launches a new activity.
builder.setContentIntent(resultPendingIntent);
```

##构造大视图通知
下面代码中展示了如何在大视图通知中设置一个按钮：
```java
// Sets up the Snooze and Dismiss action buttons that will appear in the
// big view of the notification.
Intent dismissIntent = new Intent(this, PingService.class);
dismissIntent.setAction(CommonConstants.ACTION_DISMISS);
PendingIntent piDismiss = PendingIntent.getService(this, 0, dismissIntent, 0);
Intent snoozeIntent = new Intent(this, PingService.class);
snoozeIntent.setAction(CommonConstants.ACTION_SNOOZE);
PendingIntent piSnooze = PendingIntent.getService(this, 0, snoozeIntent, 0);
```

下面的片段展示了如何构造[Builder](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html)对象。它设置了大视图的风格为"big text"，并设置了提醒消息的文本。它还使用了[addAction()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#addAction(android.support.v4.app.NotificationCompat.Action))方法来添加Snooze按钮及Dismiss按钮，这两个按钮将会出现在大视图通知上：
```java
// Constructs the Builder object.
NotificationCompat.Builder builder =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.ic_stat_notification)
        .setContentTitle(getString(R.string.notification))
        .setContentText(getString(R.string.ping))
        .setDefaults(Notification.DEFAULT_ALL) // requires VIBRATE permission
        /*
         * Sets the big view "big text" style and supplies the
         * text (the user's reminder message) that will be displayed
         * in the detail area of the expanded notification.
         * These calls are ignored by the support library for
         * pre-4.1 devices.
         */
        .setStyle(new NotificationCompat.BigTextStyle()
                .bigText(msg))
        .addAction (R.drawable.ic_stat_dismiss,
                getString(R.string.dismiss), piDismiss)
        .addAction (R.drawable.ic_stat_snooze,
                getString(R.string.snooze), piSnooze);
```