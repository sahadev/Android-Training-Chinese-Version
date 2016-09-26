原文地址：[http://android.xsoftlab.net/training/notify-user/navigation.html](http://android.xsoftlab.net/training/notify-user/navigation.html)

设计通知时要考虑到用户所预想的导航体验。通常有以下两种情况：

常规的Activity(Regular activity)

- 这里所启动的Activity是作为应用程序的正常流程部分出现的。

指定的Activity(Special activity)

- 用户只会看到这个Activity，如果这个Activity是从通知启动的话。在直觉上，这个Activity是用来展示通知上的详细信息的。

##设置常规的Activity

设置启动常规的Activity需要执行以下步骤：

- 1.在清单文件中定义Activity的层级，最终的清单文件应该是这样的：
```xml
<activity
    android:name=".MainActivity"
    android:label="@string/app_name" >
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity
    android:name=".ResultActivity"
    android:parentActivityName=".MainActivity">
    <meta-data
        android:name="android.support.PARENT_ACTIVITY"
        android:value=".MainActivity"/>
</activity>
```

- 2.创建一个基于回退栈的Intent，它用来启动父Activity(下面的代码可能有误，请自行验证。)：
```java
int id = 1;
...
Intent resultIntent = new Intent(this, ResultActivity.class);
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
// Adds the back stack
stackBuilder.addParentStack(ResultActivity.class);
// Adds the Intent to the top of the stack
stackBuilder.addNextIntent(resultIntent);
// Gets a PendingIntent containing the entire back stack
PendingIntent resultPendingIntent =
        stackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);
...
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
builder.setContentIntent(resultPendingIntent);
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mNotificationManager.notify(id, builder.build());
```

##设置启动指定的Activity
启动指定的Activity不需要回退栈，所以不需要在清单文件中定义Activity的层级，也不需要在代码中使用[addParentStack()](http://android.xsoftlab.net/reference/android/support/v4/app/TaskStackBuilder.html#addParentStack(android.app.Activity))构建回退栈。相反的，使用清单文件来设置Activity的任务模式，并通过[getActivity()](http://android.xsoftlab.net/reference/android/app/PendingIntent.html#getActivity(android.content.Context,%20int,%20android.content.Intent,%20int))创建[PendingIntent](http://android.xsoftlab.net/reference/android/app/PendingIntent.html)就可以完成创建。

- 1.在清单文件中，为Activity添加如下属性：

 - [android:name](http://android.xsoftlab.net/guide/topics/manifest/activity-element.html#nm)="activityclass" 指定该Activity的全限定名称.
 - [android:taskAffinity](http://android.xsoftlab.net/guide/topics/manifest/activity-element.html#aff)="" 与在代码中设置的[FLAG_ACTIVITY_NEW_TASK](http://android.xsoftlab.net/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK)标志一同使用。这可以确保这个Activity不会进入应用程序的默认任务栈中。任何已有的任务栈皆不会受到这个属性的影响。
 - [android:excludeFromRecents](http://android.xsoftlab.net/guide/topics/manifest/activity-element.html#exclude)="true" 将该Activity从***Recents***中排除，所以用户不会意外的再通过返回键启动这个Activity.

下面的代码段展示了这个属性的设置：
```xml
<activity
    android:name=".ResultActivity"
...
    android:launchMode="singleTask"
    android:taskAffinity=""
    android:excludeFromRecents="true">
</activity>
...
```

- 2.构建及发射通知
 - a.创建一个用于启动Activity的Intent.
 - b.通过调用[setFlags()](http://android.xsoftlab.net/reference/android/content/Intent.html#setFlags(int))方法并设置[FLAG_ACTIVITY_NEW_TASK](http://android.xsoftlab.net/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK)和[FLAG_ACTIVITY_CLEAR_TASK](http://android.xsoftlab.net/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TASK)标志来设置Activity将要启动一个新的、空的任务栈.
 - c.为Intent设置你所需要的选项.
 - d.通过调用[getActivity()](http://android.xsoftlab.net/reference/android/app/PendingIntent.html#getActivity(android.content.Context,%20int,%20android.content.Intent,%20int))由Intent创建一个[PendingIntent](http://android.xsoftlab.net/reference/android/app/PendingIntent.html)。你可以将这个[PendingIntent](http://android.xsoftlab.net/reference/android/app/PendingIntent.html)作为[setContentIntent()](http://android.xsoftlab.net/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentIntent(android.app.PendingIntent))方法的参数。

下面的代码段演示了这个实现过程：
```java
// Instantiate a Builder object.
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
// Creates an Intent for the Activity
Intent notifyIntent =
        new Intent(new ComponentName(this, ResultActivity.class));
// Sets the Activity to start in a new, empty task
notifyIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | 
        Intent.FLAG_ACTIVITY_CLEAR_TASK);
// Creates the PendingIntent
PendingIntent notifyIntent =
        PendingIntent.getActivity(
        this,
        0,
        notifyIntent,
        PendingIntent.FLAG_UPDATE_CURRENT
);
// Puts the PendingIntent into the notification builder
builder.setContentIntent(notifyIntent);
// Notifications are issued by sending them to the
// NotificationManager system service.
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// Builds an anonymous Notification object from the builder, and
// passes it to the NotificationManager
mNotificationManager.notify(id, builder.build());
```

