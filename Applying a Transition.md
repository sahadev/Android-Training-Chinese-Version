原文链接：[http://android.xsoftlab.net/training/transitions/transitions.html](http://android.xsoftlab.net/training/transitions/transitions.html)

在转场框架中，动画是由一帧帧的图像连续绘制形成的，这一帧帧的图像描述了启动场景到结束场景的整个过程。转场框架将这些动画作为一个转场对象。如果要启动动画，需要提供一个转场对象，并将结束场景提供给转场管理者。

这节课将会学习如何在场景转换中使用平移、缩放及淡入淡出动画。下节课将会学习如何定义自定义动画。

##创建转场
在前面的课程中，我们学习了如何创建场景的相关知识。一旦定义了启动场景与结束场景，那么还需要创建一个[Transition](http://android.xsoftlab.net/reference/android/transition/Transition.html)对象。转场框架允许使用内置的转场资源及动态创建的转场逻辑。

###由资源文件创建转场实例
这项技术的优点是，可以修改转场定义而不需要修改Java代码。这项技术还可以将复杂形式的转场定义分离成简单的形式，更多的信息请参见[Specify Multiple Transitions](http://android.xsoftlab.net/training/transitions/transitions.html#Multiple)。

如果要使用内置的转场资源，主要执行以下步骤：
1.在工程中添加目录res/transition/。
2.在上面的目录中创建一个新的XML资源文件。
3.添加一个内置的转场节点到该文件中。

下面的示例代码使用了[Fade](http://android.xsoftlab.net/reference/android/transition/Fade.html)转场动画：
res/transition/fade_transition.xml
```xml
<fade xmlns:android="http://schemas.android.com/apk/res/android" />
```

下面的代码段展示了如何加载转场资源到Activity中：
```java
Transition mFadeTransition =
        TransitionInflater.from(this).
        inflateTransition(R.transition.fade_transition);
```

###动态创建转场对象
这项技术可以在代码中动态的创建转场对象，使得可以在代码中修改UI界面。创建简单的内置转场实例只需要几个参数或者不需要参数。

如果要创建内置的转场实例，需要调用类[Transition](http://android.xsoftlab.net/reference/android/transition/Transition.html)子类其中一个构造方法即可。下面的例子就创建了一个[Fade](http://android.xsoftlab.net/reference/android/transition/Fade.html)转场的实例：
```java
Transition mFadeTransition = new Fade();
```

##使用转场动画
通常需要启动转场动画来响应某些事件，比如用户的输入行为。考虑有这么一种情景，当用户输入搜索的关键词，按下了搜索按钮，接下来APP会变换场景，这个场景显示了搜索之后的结果，并且输入框与搜索按钮会被隐藏。

为了使动画可以响应用户的某些输入，需要调用静态方法 [TransitionManager.go()](http://android.xsoftlab.net/reference/android/transition/TransitionManager.html#go(android.transition.Scene))，并传入结束场景及转场动画：
```java
TransitionManager.go(mEndingScene, mFadeTransition);
```

转场动画由转场实例所指定的结束场景开始，这里的启动场景是上一个转场的结束场景。如果没有上一个转场，那么启动场景会由当前的UI状体自动决定。

如果没有指定转场实例，那么转场管理者会采用一种自动话转场实例。有关更多信息，请参见[TransitionManager](http://android.xsoftlab.net/reference/android/transition/TransitionManager.html)类。

##选择指定的目标View
默认情况下，转场框架可以对所有的View执行转场。在某些情况下，可能需要将转场动画作用在View的子集上。比如，转场框架并不支持在ListView对象上使用动画，所以不要试图在ListView的子集上使用动画。不过转场框架允许选择指定的View来使用使用。

每一个可以作用转场动画的View都被称为target.不过只可以选择其中一个与场景关联的View部分.

如果要从target列表中要移除View，需要在启动转场之前调用[removeTarget()](http://android.xsoftlab.net/reference/android/transition/Transition.html#removeTarget(android.view.View))方法。如果要往target列表中添加View，调用[addTarget()](http://android.xsoftlab.net/reference/android/transition/Transition.html#addTarget(android.view.View))方法即可。有关更多信息，请参见[Transition](http://android.xsoftlab.net/reference/android/transition/Transition.html)类。

##多重转场
为了能使动画引起更多的关注，应该将动画的类型与所发生的事件相匹配。比如说，在移除某些View的时候，恰当的使用淡出效果会在潜意识中告诉用户这个View不再可用。再比如，在屏幕上移动View，最好的方式就是使用动画描述整个移动过程，以便可以让用户注意到View移动到了新的位置上。

你大可不必纠结选哪一种动画才好，转场框架提供了一种整合动画的能力，它可以将内置动画或者自定义动画整合到一起一同执行。

如果要在XML中定义转场集合，需要在res/transitions/目录下创建一个资源文件，然后在文件中列出各个转场动画。下面的代码段定义了与[AutoTransition](http://android.xsoftlab.net/reference/android/transition/AutoTransition.html)类相同的效果：
```xml
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
    android:transitionOrdering="sequential">
    <fade android:fadingMode="fade_out" />
    <changeBounds />
    <fade android:fadingMode="fade_in" />
</transitionSet>
```

如要加载上述的转场集合，需要调用[TransitionInflater.from()](http://android.xsoftlab.net/reference/android/transition/TransitionInflater.html#from(android.content.Context))方法。[TransitionSet](http://android.xsoftlab.net/reference/android/transition/TransitionSet.html)类继承了[Transition](http://android.xsoftlab.net/reference/android/transition/Transition.html)类，所以可以像普通的[Transition](http://android.xsoftlab.net/reference/android/transition/Transition.html)对象一样使用[Transition](http://android.xsoftlab.net/reference/android/transition/Transition.html)对象集合。

##在非场景中使用转场动画
更改View层级并不是修改UI界面的唯一方式。除此之外，还可以在单一层级中通过添加，修改，移除的方式更改用户界面。比如说，可以通过单一的布局来实现搜索界面。这个布局刚开始只显示了一个搜索的输入框与一个搜索图标。如果要显示搜索结果，需要在用户点击搜索按钮的时候通过调用ViewGroup.removeView()方法移除搜索按钮，然后通过ViewGroup.addView()方法添加搜索结果。

你可能在与上述情况相似的情况下采用类似的解决方式。比起采用两个单独的布局文件，可以采用动态修改单个布局文件的方式来完成相同的效果。

如果要实现上面的情景，那么则不需要创建场景。相反，你可以使用延迟转场来实现这种情况。转场框架的这个特性由当前View层级的状态开始，记录了该View层级的改变状态，最后在系统重绘UI界面时将转场应用到这个过程中。

如果要使用单一的View层级创建延迟转场，需要执行以下步骤：

1.如果要触发转场事件的发生，则需要调用[TransitionManager.beginDelayedTransition()](http://android.xsoftlab.net/reference/android/transition/TransitionManager.html#beginDelayedTransition(android.view.ViewGroup))方法，该方法暴露了两个参数：1.执行更改View的父View，2.要使用的转场动画。转场框架会存储更改View的当前状态及属性值。

2.要更改的View一定是跟使用情况相关的。转场框架会记录更改View的改变过程及属性。

3.在系统重绘用户界面的过程中，转场框架会在这个过程中执行动画。

下面的示例展示了如何使用延迟转场动画添加TextView。第一段定义了布局文件：
res/layout/activity_main.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/mainLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <EditText
        android:id="@+id/inputText"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    ...
</RelativeLayout>
```

第二段定义了动画添加TextView的代码：
```java
private TextView mLabelText;
private Fade mFade;
private ViewGroup mRootView;
...
// Load the layout
this.setContentView(R.layout.activity_main);
...
// Create a new TextView and set some View properties
mLabelText = new TextView();
mLabelText.setText("Label").setId("1");
// Get the root view and create a transition
mRootView = (ViewGroup) findViewById(R.id.mainLayout);
mFade = new Fade(IN);
// Start recording changes to the view hierarchy
TransitionManager.beginDelayedTransition(mRootView, mFade);
// Add the new TextView to the view hierarchy
mRootView.addView(mLabelText);
// When the system redraws the screen to show this update,
// the framework will animate the addition as a fade in
```

##定义转场的生命周期回调
转场的生命周期与Activity的生命周期非常类似。它代表了由[TransitionManager.go()](http://android.xsoftlab.net/reference/android/transition/TransitionManager.html#go(android.transition.Scene))方法开始到动画执行结束之间的完整的时间过程。在重要的生命过程中，转场框架会调用[TransitionListener](http://android.xsoftlab.net/reference/android/transition/Transition.TransitionListener.html)接口所定义的回调。

这些回调是极有用的，比如说，在场景转变的过程中复制View的属性值。你不能简单的将开始时View的值拷贝到结束时的View上，因为在转场完成之前结束中的View层级还没有完全没填充。相反的，你可以将值拷贝到一个变量中，然后在转场框架结束的时候将值拷贝到View层级中。为了可以在转场完成的时候获得通知，可以在Activity中实现[TransitionListener.onTransitionEnd()](http://android.xsoftlab.net/reference/android/transition/Transition.TransitionListener.html#onTransitionEnd(android.transition.Transition))方法。

有关更多信息，请参见[TransitionListener](http://android.xsoftlab.net/reference/android/transition/Transition.TransitionListener.html)类的相关说明。