原文地址 : [http://android.xsoftlab.net/training/basics/activity-lifecycle/pausing.html](http://android.xsoftlab.net/training/basics/activity-lifecycle/pausing.html)

在APP的正常使用过程中，在前台工作的Activity有时可能会被其他的可视化组件挡住，而引起Activity进入Paused状态。举个例子，当一个半透明的Activity打开后(类似于Dialog那种风格)，那么原先的那个Activity便会进入Paused状态。只要Activity仍然只是部分可见，并且它没有获得焦点，那么它就一直保持在Paused状态。

然而，只要activity一旦被全部挡住，并且不可见，那么就会进入Stopped状态。

在系统调用Activity的onPause方法时，activity随之就进入了paused状态，这期间允许你停止一些不应该继续进行的活动(比如视频)，还应该对用户的任何信息做持久化存储，万一用户退出了APP。如果用户从Paused状态返回了Activity，系统会调用onResumed方法并回到Resumed状态。

> **Note:**当Activity的onPause方法被调用，这意味着Activity可能会在Paused状态待一会，并且稍后用户可能会再次返回到这个Activity中。无论如何，这通常是用户离开Activity的第一个信号。

![](http://android.xsoftlab.net/images/training/basics/basic-lifecycle-paused.png)
**上图：**当一个半透明的Activity挡住了原先的Activity，系统会调用onPause方法，然后Activity会等在Paused状态(1)，如果在Paused状态返回了Activity，那么系统会调用onResume方法(2)。

##暂停Activity
当系统调用了onPause方法，这从技术上说activity当前是部分可见状态，但是大多数情况下，这表示用户离开了Activity，并且稍后会进入Stopped状态。你应该一般使用onPause方法做这些事情：

- 停止动画或者运行中的活动等这类消耗CPU资源的行为。
- 保存没有存储的改变，但这仅限于用户希望保存的东西(比如电子邮件的草稿)。
- 释放系统资源，比如广播接收器，正在处理中的传感器(比如GPS)，任何用户不再需要的，可能会影响到电量的任何资源。 

举个例子，如果应用使用了Camera，在onPause方法中最适合去释放它。
```java
@Override
public void onPause() {
    super.onPause();  // Always call the superclass method first
    // Release the Camera because we don't need it when paused
    // and other activities might need to use it.
    if (mCamera != null) {
        mCamera.release()
        mCamera = null;
    }
}
```

通常情况下，并不应该使用onPause方法来持久化存储用户的改变(比如输入表格的用户信息)。唯一的一点就是用户希望这些数据可以自动的存储(比如起草的email)。然而，应该避免在onPause方法中执行高强度的CPU工作，比如写入数据库，因为它会减慢切换到下一个Activity的速度(你应该在onStop方法中做这些重量级操作)。

你应该在onPause方法中保持相对简单的完成操作，为了可以快速过渡到下个Activity。

> **Note:**如果activity在Paused状态，那么activity会常驻在内存中，它会在activity恢复的时候重新被调用。你不需要重新初始化这些在任何回调函数中被重新创建的组件。

##恢复Activity
如果用户从Paused状态恢复到了Resumed状态，系统会调用onResume方法。

应该意识到系统每次调用这个方法activity就进入了前台，包括在第一次创建的时候。因此，你应该在onResume中实例化组件，然后在onPause中释放这些组件，每次在activity进入resumed状态的时候执行其必须的初始化操作(比如启动动画和activity获取到焦点之后只实例化要使用的组件)。

下面这个onPause的例子是上面onResume例子的副本，所以应该在activity暂停的时候释放初始化过的camera对象。
```java
@Override
public void onResume() {
    super.onResume();  // Always call the superclass method first
    // Get the Camera instance as the activity achieves full user focus
    if (mCamera == null) {
        initializeCamera(); // Local method to handle camera init
    }
}
```