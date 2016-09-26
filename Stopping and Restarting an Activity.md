原文地址 : [http://android.xsoftlab.net/training/basics/activity-lifecycle/stopping.html#Start](http://android.xsoftlab.net/training/basics/activity-lifecycle/stopping.html#Start)

在activity的生命周期内，适当的停止和重新启动activity是一个非常重要的过程，它可以确保用户能感觉到APP一直是存活状态，并且不会丢失他们的进度。这里有几项关键的场景适用于activity停止与重启：

- 用户打开了最新使用APP的窗口然后从你的APP切换到了别的APP上。现在APP内的activity当前位于后台，并且处于停止状态。如果用户从设备屏幕的主界面通过点击图标或者从最近使用APP的窗口返回了了App，那么原来的那个activity则会重新启动。
- 用户在APP内执行了一个启动新activity的动作。那么当前的这个activity会在第二个activity创建的时候停止。如果用户按下了返回按钮，则第一个activity会被重启。
- 用户在使用APP的时候接到了一个电话。

Activity类提供了两个生命周期函数，onStop和onRestart，它们可以使activity单独的处理停止状态和重启状态。和局部UI被遮挡的暂停状态不同，停止状态保证UI一定是不可见的，并且用户的焦点转移到了另一个Activity上。

>**Note:**因为系统会在activity停止的时候驻留在内存中，所以你可能不需要实现onStop方法和onResume方法，甚至是onStart方法。为了大多数活动相对简单，activity只需要停止和重启就好了，你可能只需要使用onPause方法来暂停一些正在进行中的活动并且释放系统资源。

![](http://android.xsoftlab.net/images/training/basics/basic-lifecycle-stopped.png)
**上图:**当用户离开了activity，系统会调用onStop方法停止activity(1)。如果用户从activity的停止状态返回了，系统会调用onRestart方法(2)，然后会很快的接着调用onStart方法(3)和onResume方法(4)。注意不用关心activity在停止的时候的状况，因为系统总是会在onStop之前调用onResume。

##停止Activity
当activity的onStop方法被调用了，这时，activity不再可见，并且需要释放用户不需要使用的所有资源。activity一旦停止，如果系统需要恢复系统内存的话，系统会销毁这个activity。在极端情况下，系统会将整个APP杀死，并且不会调用activity的onDestory方法，所以使用onStop方法释放资源可能会引起内存泄露，这是很严重的。

尽管onPause方法在onStop之前被调用了，你应该使用onStop执行更大、更多的CPU高负荷操作,比如将信息写入数据库。

这个例子实现了onStop方法，方法内对草稿内容进行了持久化存储：
```java
@Override
protected void onStop() {
    super.onStop();  // Always call the superclass method first
    // Save the note's current draft, because the activity is stopping
    // and we want to be sure the current note progress isn't lost.
    ContentValues values = new ContentValues();
    values.put(NotePad.Notes.COLUMN_NAME_NOTE, getCurrentNoteText());
    values.put(NotePad.Notes.COLUMN_NAME_TITLE, getCurrentNoteTitle());
    getContentResolver().update(
            mUri,    // The URI for the note to update.
            values,  // The map of column names and new values to apply to them.
            null,    // No SELECT criteria are used.
            null     // No WHERE columns are used.
            );
}
```

当activity处于停止状态时，Activity对象会驻留在系统中，然后当activity恢复的时候会被重新调用。你不需要重新实例化那些在其它回调方法内创建的对象。系统会保持布局中的每一个View的状态，所以如果用户在EditText中输入了文字，那么这些内容会保持在内存中，所以不需要专门去保存和恢复它。

> **Note:**甚至是activity在停止的状态被系统销毁了，仍然可以在一个Bundle中保存View对象的状态(比如EditText中的文本)，并且可以在用户返回了这类Activity的实例(原来的对象已被销毁，这个对象是又重新创建的)时恢复它们(下节课会讨论有关在Activity销毁和重新创建的时候使用Bundle存储或读取其它状态的数据)。

##启动/重新启动Activity
当Activity从停止状态返回了前台，系统会调用onRestart方法和onStart方法，这样的调用会每次出现在activity可见的时候。然而onRestart方法只是被在activity从停止状态恢复的时候被调用，所以可以使用这个方法执行一些特别的必须的恢复工作，只限于activity先前处于停止状态而不是销毁状态。

APP使用onRestart方法恢复activity的状态并不通用，所以这里没有该方法的可以适用于一般受欢迎的APP的使用样例。然而，因为onStop方法本来应该清理所有的activity资源，所以在activity重启的时候需要重新实例化它们。还有，你也应该在activity第一次创建的时候实例化它们(当这个activity的实例不存在的时候)。对于这个原因，你应该使用与onStop方法相对应的onStart方法，因为系统会在创建activity时候和在停止状态下重启activity的时候都会调用onStart方法。

举个例子，因为用户可能有多种方式将长时间不可见的activity重新带回来，所以onStart方法是一个验证系统特性是否可用的好地方：
```java
@Override
protected void onStart() {
    super.onStart();  // Always call the superclass method first
    
    // The activity is either being restarted or started for the first time
    // so this is where we should make sure that GPS is enabled
    LocationManager locationManager = 
            (LocationManager) getSystemService(Context.LOCATION_SERVICE);
    boolean gpsEnabled = locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER);
    
    if (!gpsEnabled) {
        // Create a dialog here that requests the user to enable GPS, and use an intent
        // with the android.provider.Settings.ACTION_LOCATION_SOURCE_SETTINGS action
        // to take the user to the Settings screen to enable GPS when they click "OK"
    }
}
@Override
protected void onRestart() {
    super.onRestart();  // Always call the superclass method first
    
    // Activity being restarted from stopped state    
}
```

当系统需要销毁Activity时，它会调用activity的onDestory方法。因为在通常情况下，你会在onStop方法中释放大部分资源，那么在onDestory方法内不需要再做什么工作。这个方法是清理可能会导致内存泄露的资源的最后机会，所以你应该确保其它附属线程以及其它类似方法跟踪的长时间运行的工作也被销毁。