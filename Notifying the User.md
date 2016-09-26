原文地址：[http://android.xsoftlab.net/training/notify-user/index.html](http://android.xsoftlab.net/training/notify-user/index.html)

#引言
通知用于在有事件发生时，将事情以更便捷的方式展示给用户。用户可以在他们方便的时候直接与通知交互。

[Notifications design guide](http://android.xsoftlab.net/design/patterns/notifications.html)课程讲述了如何设计有效的通知以及何时去使用它们。这节课将会学习如何实现通用的通知设计。

#构建通知
这节课的实现主要基于[NotificationCompat.Builder](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html)类，[NotificationCompat.Builder](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html)类属于支持库。开发者应该使用NotificationCompat及其子类，特别是[NotificationCompat.Builder](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html)，以便支持更宽泛的平台。

##创建通知构建器
当创建通知时，需要指定通知的UI内容以及它的点击行为。一个[Builder](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html)对象至少要包含以下条件：

- 一个小图标，通过[setSmallIcon()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setSmallIcon(int))方法设置。
- 通知标题，通过[setContentTitle()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentTitle(java.lang.CharSequence))方法设置。
- 详细文本，通过[setContentText()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentText(java.lang.CharSequence))方法设置。

比如：
```java
NotificationCompat.Builder mBuilder =
    new NotificationCompat.Builder(this)
    .setSmallIcon(R.drawable.notification_icon)
    .setContentTitle("My notification")
    .setContentText("Hello World!");
```

##定义通知的行为
创建通知时，应当至少为通知添加一个行为。这个行为会将用户带到Activity中，这个Activity中详细的展示了发生了什么事情，或者可以使用户采取进一步的行动。在通知内部，行为由[PendingIntent](http://android.xsoftlab.net/reference/android/app/PendingIntent.html)所包含的[Intent](http://android.xsoftlab.net/reference/android/content/Intent.html)指定，它可以用来启动Activity.

如何构造[PendingIntent](http://android.xsoftlab.net/reference/android/app/PendingIntent.html)取决于要启动的Activity的类型。当由通知启动Activity时，开发者必须考虑用户所期待的导航体验。在下面的代码中，点击通知会启动一个新的Activity，这个Activity继承了通知所产生的行为习惯。在这种情况下不需要创建人为的回退栈。
```java
Intent resultIntent = new Intent(this, ResultActivity.class);
...
// Because clicking the notification opens a new ("special") activity, there's
// no need to create an artificial back stack.
PendingIntent resultPendingIntent =
    PendingIntent.getActivity(
    this,
    0,
    resultIntent,
    PendingIntent.FLAG_UPDATE_CURRENT
);
```

##设置通知的点击行为
为了使[PendingIntent](http://android.xsoftlab.net/reference/android/app/PendingIntent.html)与手势产生关联，需要调用NotificationCompat.Builder的对应方法。比如要启动一个Activity，则调用[setContentIntent()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentIntent(android.app.PendingIntent))方法添加[PendingIntent](http://android.xsoftlab.net/reference/android/app/PendingIntent.html)即可。

##发布通知
发布通知需要执行以下步骤：

- 获得[NotificationManager](http://android.xsoftlab.net/reference/android/app/NotificationManager.html)的实例。
- 使用[notify()](http://android.xsoftlab.net/reference/java/lang/Object.html#notify())方法发布通知。在调用[notify()](http://android.xsoftlab.net/reference/java/lang/Object.html#notify())方法时需要指定通知的ID，这个ID用于通知的稍后更新。
- 调用[build()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#build())方法，它会返回一个[Notification](http://android.xsoftlab.net/reference/android/app/Notification.html)对象。

```java
NotificationCompat.Builder mBuilder;
...
// Sets an ID for the notification
int mNotificationId = 001;
// Gets an instance of the NotificationManager service
NotificationManager mNotifyMgr = 
        (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
// Builds the notification and issues it.
mNotifyMgr.notify(mNotificationId, mBuilder.build());
```

