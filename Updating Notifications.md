原文地址：[http://android.xsoftlab.net/training/notify-user/managing.html#Removing](http://android.xsoftlab.net/training/notify-user/managing.html#Removing)

当需要在不同时段发布同一事件类型的通知时，应该避免创建新的通知。相反的，应当考虑更新原有的通知，比如更改通知的某些值或者添加一些信息给通知。

下面的部分描述了如何更新通知以及如何移除通知。

##修改通知
为了设置通知是可以更新的，需要在发布通知时由[NotificationManager.notify(ID, notification)](http://android.xsoftlab.net/reference/android/app/NotificationManager.html#notify(int,%20android.app.Notification))方法指定该通知的ID。为了更新这条通知，需要更新或者创建一个NotificationCompat.Builder对象，并由这个对象构建一个Notification对象，然后将这个通知对象以相同的ID发布出去。

下面的代码段演示了在事件发生时，一条通知将会被用来更新该事件的数目：
```java
mNotificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// Sets an ID for the notification, so it can be updated
int notifyID = 1;
mNotifyBuilder = new NotificationCompat.Builder(this)
    .setContentTitle("New Message")
    .setContentText("You've received new messages.")
    .setSmallIcon(R.drawable.ic_notify_status)
numMessages = 0;
// Start of a loop that processes data and then notifies the user
...
    mNotifyBuilder.setContentText(currentText)
        .setNumber(++numMessages);
    // Because the ID remains unchanged, the existing notification is
    // updated.
    mNotificationManager.notify(
            notifyID,
            mNotifyBuilder.build());
...
```

##移除通知
在以下事件发生时，通知将会从通知栏中移除:

- 用户移除了该通知或者使用了"Clear All"功能(如果通知是可移除的话)。
- 用户点击了通知，这条通知在创建时使用了[setAutoCancel(false)](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setAutoCancel(boolean))方法(false是默认属性)。
- 通过调用[cancel()](http://android.xsoftlab.net/reference/android/app/NotificationManager.html#cancel(int))方法并指定该通知的ID。这个方法还可以移除进行中的通知。
- 通过调用[cancelAll()](http://android.xsoftlab.net/reference/android/app/NotificationManager.html#cancelAll())方法，将已经发布的所有通知移除。