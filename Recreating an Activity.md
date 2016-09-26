原文地址：[http://android.xsoftlab.net/training/basics/activity-lifecycle/recreating.html#RestoreState](http://android.xsoftlab.net/training/basics/activity-lifecycle/recreating.html#RestoreState)

有这么几个关于activity通过正常渠道销毁的场景，比如用户按下了返回按钮，又或者是在activity中调用了终止信号finish。系统可能也会在activity在停止状态时销毁它，也可能会在长时间不使用的时候销毁它，也可能会当前台activity需要更多资源时，系统必须关掉后台进行来恢复内存时销毁它。

当activity因为用户按下了返回按钮或者是自己关闭而被销毁的话，系统会认为activity的实例是永远消失了，因为习惯上会认为activity已经不再需要了。然而，如果activity是因为系统资源枯竭而被销毁的话，尽管activity的实例已经被销毁了，系统会记得它存在过，以便用户在返回的时候系统可以创建一个新的activity实例，并且通过上一个activity销毁时保存的一系列数据来恢复当时的状态。系统恢复原先的状态时通过一个名为instance state的对象存储的数据，它是一个Bundle对象，并以键值对的方式存储数据的集合。

> **警告：**在每次用户旋转屏幕的时候，activity会被销毁并重新创建。当屏幕改变的方向，系统会销毁并重新创建当前的activity，因为屏幕的配置发生了改变，activity可能需要加载更改后的资源(比如说布局)。

默认情况下，系统使用Bundle对象存储activity不居中每一个View对象的信息(比如说在EditText中输入的内容)。所以如果activity对象被销毁然后重新创建了的话，可以不必再写多少代码就可以恢复布局的状态到原来的状态。然而，activity可能有更多的状态信息需要恢复，比如activity中记录用户进度的成员变量。

> **Note:**为了使Android系统可以保存activity中view的状态，**每一个VIEW必须拥有唯一的ID**，支持android:id属性。

为了保存activity状态的附加数据，你必须重写onSaveInstanceState方法。系统会在用户离开的时候调用这个方法，并且会传回一个Bundle对象，这个对象可以用来在activity意外被销毁的事件中保存下来。如果系统稍后必须创建activity实例，系统会通过onRestoreInstanceState方法和onCreate方法传递相同的Bundle对象。

![](http://android.xsoftlab.net/images/training/basics/basic-lifecycle-savestate.png)

随着系统开始停止activity，它会调用onSaveInstanceState(1)，所以你可以指定一些在稍后恢复的时候所需要的附加状态数据。如果activity销毁了，然后一个相同的实例必须重新创建，那么系统会通过onCreate方法(2)和onRestoreInstanceState(3)方法将默认的状态数据传递回来。

##存储Activity的状态
随着activity开始进入停止状态，系统会调用onSaveInstanceState方法，所以activity可以存储一系列的状态信息。这个方法的默认实现是保存了一些与activity的view层级相关额信息，比如在EditText空间中的内容，又或者是ListView的滑动位置。

为了保存activity的附加状态信息，必须实现onSaveInstanceState方法然后添加键值对到Bundle对象中：
```java
static final String STATE_SCORE = "playerScore";
static final String STATE_LEVEL = "playerLevel";
...
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    // Save the user's current game state
    savedInstanceState.putInt(STATE_SCORE, mCurrentScore);
    savedInstanceState.putInt(STATE_LEVEL, mCurrentLevel);
    
    // Always call the superclass so it can save the view hierarchy state
    super.onSaveInstanceState(savedInstanceState);
}
```

> **警告:**因为总是调用了onSaveInstanceState的父类实现，所以默认实现可以保存view层级的状态。

##恢复Activity的状态
如果之前的activity销毁了之后又重新创建了，可以通过Bundle对象恢复保存的状态。onCreate方法和onSaveInstanceState方法都会将相同的Bundle对象通过参数回调回来。

因为onCreate方法在系统创建新实例的时候会调用，你必须在尝试访问它之前检查Bundle对象是否为null，如果是null，那说明系统创建了一个新的activity对象，否则就是原来的那个对象被销毁了这里又重新创建了一个。

以下是如何在onCreate方法中恢复一些状态数据的例子：
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState); // Always call the superclass first
   
    // Check whether we're recreating a previously destroyed instance
    if (savedInstanceState != null) {
        // Restore value of members from saved state
        mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
        mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
    } else {
        // Probably initialize members with default values for a new instance
    }
    ...
}
```

除了onCreate方法可以恢复状态之外，onRestoreInstanceState()方法同样也可以实现该功能，该方法会在onStart方法之后调用，不过仅仅是在保存了数据之后才会调用，所以你不需要去做Bundle对象是否为null的检查：
```java
public void onRestoreInstanceState(Bundle savedInstanceState) {
    // Always call the superclass so it can restore the view hierarchy
    super.onRestoreInstanceState(savedInstanceState);
   
    // Restore state members from saved instance
    mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
    mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
}
```

>**警告:**因为总是调用了onRestoreInstanceState()的父类实现，所以默认实现可以恢复view层级的状态。

有关更多因为在运行时的重启事件而造成的activity重新创建(比如屏幕旋转)的信息，请参见[Handling Runtime Changes](http://android.xsoftlab.net/guide/topics/resources/runtime-changes.html)。