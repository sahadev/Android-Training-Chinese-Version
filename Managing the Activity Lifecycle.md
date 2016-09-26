原文地址 : [http://android.xsoftlab.net/training/basics/activity-lifecycle/index.html](http://android.xsoftlab.net/training/basics/activity-lifecycle/index.html)

#导言
用户通过导航退出或者返回应用的时候，应用中Activity的生命周期会在不同的状态之间变换。举个例子，当Activity初次启动的时候，它会来到系统的前面，然后得到用户焦点。在这个过程中，Android系统会调用Activity的一系列生命周期函数。如果用户执行了启动另一个Activity的动作或是转换到其它APP上了，Android则会调用另一部分的生命周期函数，看起来就像是进入了后台(虽然现在这个Activity是完全不可见状态的，但是它的状态还是会被完整的保存着)。

在生命周期回调函数中，你可以在用户离开或者重新进入Activity时定义Activity的行为。举个例子，如果你正在构建一个流媒体播放器，当用户转换到其它APP时你可能希望暂停视频播放并暂停网络连接，当用户又返回当前的Activity时，你希望重新建立网络连接并在原来的位置继续播放视频。

这节课介绍了每个Activity实例的重要生命周期回调函数的回调及如何使用它们。所以当Activity不再使用它们的时候，Activity需要释放那些不再会重新使用的资源，就像用户所期待的那样。

#启动一个Activity
不像其他程序一样，App并不是从main()函数启动的。Android系统在Activity的实例中通过调用指定的回调函数进行启动。这样可以确保它们的生命周期状态保持一致。Activity启动的时候有一列按顺序执行的回调函数，在终结的时候也有一列的按顺序执行的回调函数。

这节课提供了大多数的生命周期函数概要，以及展示在创建一个新的Activity实例时如何处理第一个生命周期函数。

##了解生命周期函数
在Activity的生命中，系统会按顺序调用核心的生命周期函数集，类似一个阶梯状的金字塔。正因为这样，Activity的每一个生命周期状态在这个过程都是单独分开的。当系统创建了一个新的Activity实例，每一个回调函数都会移动Activity的状态到前一个的顶部。那么在最顶部的状态便是Activity运行在前台的状态，并且用户可以直接与它交互。

如果用户开始离开这个Activity，系统会调用其它函数来移除刚才的每一个状态，以便逐渐销毁这个Activity。在一些例子中，Activity的状态会移除一部分然后等待(比如用户将其它APP切换到了前面)，等待到activity的状态再一次回到顶部(用户回到了这个Activity)，恢复用户离开时的状态。下图是简单的生命周期说明示意图：
![](http://android.xsoftlab.net/images/training/basics/basic-lifecycle.png)

因为Activity非常复杂，你可能没有必要实现所有的生命周期函数。然而，懂得每一个生命周期函数是很重要的，确保可以通过它们的实现用户所期待的行为方式。可能有几种方式可以通过实现生命周期函数保证APP有良好的行为习惯，包括以下：

- 当在使用APP时用户接到电话或者转到别的APP上时确保不要崩溃。
- 当用户不再使用部分资源的时候，请及时释放它。
- 当用户离开APP然后又返回APP时不要丢失用户的进度，比如输入的文字和看到的视频位置。
- 当屏幕在水平方向或垂直方向旋转时不要丢失用户进度更不要崩溃。


你将会学习接下来的课程，这里有一些上图所展示的状态，然而，这里只列了3种状态，也就是说，在一个时期内，Activity只是在这3个状态中的一个内。

**Resumed**
在这个状态下，activity位于前台，并且用户可以直接与它交互。

**Paused**
在这个状态下，activity被其它Activity掩盖了一部分，其它Activity位于前台，并且是半透明状或者是只掩盖了一部分。在这个状态下，activity不再接收用户输入，不能执行任何代码。

**Stopped**
在这个状态下，activity完全被隐藏，对用户完全不可见。这里考虑到它位于后台。当停止时，activity的所有状态信息比如成员变量都会被保留，但是不能再执行任何代码。

其它状态(onCreate和onStart)是非常短暂的，系统通过调用下一个生命周期函数会非常顺序的从这个状态移动到下一个状态。所以，在在系统调用了onCreate()方法只会很快的调用onStart()，然后紧接着就是onResume().

这就是最基本的生命周期。现在你将会学习一些特殊的生命周期行为。

##指定App的启动Activity
当用户在主屏幕上点击了你应用的图标，系统会启动应用内声明了"launcher"的Activity，并调用该Activity的onCreate方法。这个Activity就是APP呈现给用户的主入口界面。

你可以在Android的清单文件中声明哪一个Activity是APP的主入口界面，AndroidManifest.xml位于工程的根目录内。

APP的主Activity必须在清单文件中以标签< intent-filter>声明，并且需要包含MAIN行为以及LAUNCHER类别：
```xml
<activity android:name=".MainActivity" android:label="@string/app_name">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

> **Note:**当你使用Android SDK工具创建了一个新的Android工程时，这个默认的工程会在清单文件中包含一个以这种过滤声明的Activity。

如果MAIN行为或者LAUNCHER类别没有在应用的其中一个Activity声明的话，那么APP的图标将不会出现在设备的主屏幕应用列表上。

##创建一个新的实例
很多APP会包含好几个不同的Activity以便让用户执行不同的行动。当用户点击了APP的启动图标创建了一个主Activity或者是在APP内点击响应按钮启动了别的Activity，系统会通过调用onCreate方法创建每一个Activity的实例。

你必须实现onCreate方法来执行一个最基本的应用启动逻辑，并且应该在Activity的整个生命周期中只执行一次。举个例子，你实现的onCreate()方法应该定义用户界面以及可能要初始化一些类相关变量。

举个例子，下面的这个onCreate的例子展示了一些代码来执行一些基础的设置，比如声明用户界面，定义一些成员变量，然后配置一些UI。
```java
TextView mTextView; // Member variable for text view in the layout
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Set the user interface layout for this Activity
    // The layout file is defined in the project res/layout/main_activity.xml file
    setContentView(R.layout.main_activity);
    
    // Initialize member TextView so we can manipulate it later
    mTextView = (TextView) findViewById(R.id.text_message);
    
    // Make sure we're running on Honeycomb or higher to use ActionBar APIs
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        // For the main activity, make sure the app icon in the action bar
        // does not behave as a button
        ActionBar actionBar = getActionBar();
        actionBar.setHomeButtonEnabled(false);
    }
}
```
> **Caution:**使用SDK_INT预防一些旧的系统运行新的API，这样的话，老系统会发生运行时错误。

onCreate()方法一旦执行完毕，系统会很快的调用onStart和onResume方法。你的系统绝不会驻留在Created状态和Started状态。从技术上将，当onStart方法被调用后activity就变得可见，但是onResume方法会很快的跟着调用，并保持在Resumed状态，直到一些事件将它改变，比如当接到电话，用户转换到了别的APP上，又或者设备的屏幕关闭了。

在下面的其它课程中，你将会见到如何启动其它方法，当activity从Paused或者Stopped状态恢复到Resumed状态时，onStart方法和onResume方法在Activity的生命周期中是非常有用的。

> **Note:**onCreate方法会包含一个名叫savedInstanceState的参数，我们稍后会在有关[Recreating an Activity](http://android.xsoftlab.net/training/basics/activity-lifecycle/recreating.html)的课程中讨论。

![](http://android.xsoftlab.net/images/training/basics/basic-lifecycle-create.png)

##销毁Activity
activity的第一个生命周期函数是onCreate，那么它的最后一个生命周期便是onDestory了。当系统接收到终止信号的时候，将会调用这个方法，然后将activity完成的从系统内存中移除。

大多数的APP不需要实现这个方法，因为本地类引用会随着Activity一起总结，不过Activity的清理工作应该放在onPause下或者onStop下。然而，不过当你在onCreate中启动了一个后台线程或者其它长时间运行的资源，这时如果不适当的清理内存的话，可能会造成内存泄露，你应该在onDestory中杀死或者关闭它们。

```java
@Override
public void onDestroy() {
    super.onDestroy();  // Always call the superclass
    
    // Stop method tracing that the activity started during onCreate()
    android.os.Debug.stopMethodTracing();
}
```


> **Note:**在所有的情况下系统调用onDestory方法之后已经调用过onPause方法与onStop方法，不过有一个例外情况：你在onCreate方法中调用了finish方法。在一些例子中，当你的Activity临时决定要启动另一个Activity，你可能要在onCreate方法内调用finish方法来销毁这个Activity，在这种情况下，系统会立即调用onDestory方法，而不会调用其它任何生命周期方法。