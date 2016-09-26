原文地址 : [http://android.xsoftlab.net/training/basics/actionbar/overlaying.html](http://android.xsoftlab.net/training/basics/actionbar/overlaying.html)

##浮层效果的ActionBar
默认情况下，ActionBar总是会出现在Activity窗口的顶部，这样会稍微的减少Activity布局的剩余空间。如果需要在用户使用的时候隐藏和显示ActionBar,可以通过调用ActionBar的hide()方法和show()方法。然而，这会让Activity重新计算并且重新绘制。

![](http://android.xsoftlab.net/images/training/basics/actionbar-overlay@2x.png)

为了避免ActionBar显示隐藏的时候重新计算Activity的大小，你可以使用ActionBar的浮层模式。在浮层模式下，Activity的布局将会使用所有的可用空间，就好像ActionBar不在那里，然后系统会将ActionBar绘制在布局的顶层。这会遮住布局顶部的一些空间，但是当ActionBar显示隐藏的时候不需要重新计算布局的尺寸。它变换的时候是无缝连接的，非常平滑。
> **Tips:** 如果想使在ActionBar后面那部分布局可见，创建一个自定义的风格，并指定一个半透明的背景，就像上面途中显示的效果一样。关于如何自定义ActionBar的背景，请看上一章的课程。

##启动浮层模式
如果要打开ActionBar的浮层模式，需要床架一个自定义的主题，并且继承已存在的ActionBar主题，然后设置android:windowActionBarOverlay的属性为true。

###为Android 3.0及高版本提供支持
如果设置的minSdkVersion是11或大于11，自定义主题的话应该继承Theme.Holo或者是它的子类：

```xml
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@android:style/Theme.Holo">
        <item name="android:windowActionBarOverlay">true</item>
    </style>
</resources>
```

###为Android 2.1及高版本提供支持
如果APP为了运行在Android 3.0以下的版本而使用了支持库的话，自定义的主题应该继承Theme.AppCompat主题或者它的子类:

```xml
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@android:style/Theme.AppCompat">
        <item name="android:windowActionBarOverlay">true</item>
        <!-- Support library compatibility -->
        <item name="windowActionBarOverlay">true</item>
    </style>
</resources>
```

这里也应该注意到这里定义了两个windowActionBarOverlay属性样式：一个是android:前缀开头的，另一个没有。有android:前缀的是用于包含有风格的Android平台版本，而没有带前缀的则是为了从支持库读取风格的老版本。

##指定布局的上外边距
当ActionBar处于浮层模式的时候，它会遮挡本来应该处于可见状态的部分布局。为了确保这部分布局一直可见，在View的顶部使用添加外边距或者内边距，并设置值为ActionBar的高度actionBarSize：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingTop="?android:attr/actionBarSize">
    ...
</RelativeLayout>
```

如果使用了支持库的话，则需要将android:前缀删除：

```xml
<!-- Support library compatibility -->
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingTop="?attr/actionBarSize">
    ...
</RelativeLayout>
```

在这里，没有前缀的?attr/actionBarSize可工作于所有的版本上，包括Android 3.0及更高的版本。