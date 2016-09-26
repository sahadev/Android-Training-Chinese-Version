原文地址：[http://android.xsoftlab.net/training/system-ui/index.html](http://android.xsoftlab.net/training/system-ui/index.html)

#引言
[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)是屏幕上的一块显示区域，专门用来显示通知，设备的通讯状态以及设备的导向。典型的[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)与APP同时显示在屏幕上。APP展示了沉浸式的内容，比如电影或者照片，可以临时性的将[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)的图标变暗，以便减少不必要的干扰，或者临时性的隐藏[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)，以便进入一种身临其境的状态。

如果你对Android Design Guide很熟悉，应该知道将APP设置为符合标准的Android UI指南及使用模型是很重要的一点。你应当仔细考虑你的用户所需要的及所期望的，在修改[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)之前，因为他们会给用户一种标准的方式来导航设备以及查看它们的状态。

这节课将会讨论如何在不同的Android版本中变暗或隐藏[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)来创造身临其境的用户体验，然而还依旧保留快速访问[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)的方法。

#使System Bars变暗
这节课将会描述如何使Android 4.0以上的[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)变暗。Android对早期的版本并没有提供内置的可以使[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)变暗的方法。

当你使用这项方法时，内容并不会重新调整尺寸，但是[System Bars](https://developer.android.com/design/get-started/ui-overview.html#system-bars)上图标在视觉上确实是收回去了。无论用户是点击了状态条区域还是导航条区域，这两个条都会完全显示出来。这种方法的优势在于Bar还在，但是它们的详细信息被模糊了，因此利用Bar可以很轻松的创建一个没有任何牺牲的身临其境的体验。

##使状态导航条变暗
你可以在Android 4.0及以上的版本中通过SYSTEM_UI_FLAG_LOW_PROFILE标志来使状态条及通知条变暗：
```java
// This example uses decor view, but you can use any visible view.
View decorView = getActivity().getWindow().getDecorView();
int uiOptions = View.SYSTEM_UI_FLAG_LOW_PROFILE;
decorView.setSystemUiVisibility(uiOptions);
```

在用户触碰到状态导航条时，这个标志会被清除，这会使状态导航条变亮。一旦这个标志被清除，如果你想再次使导航条变暗的话，还需要重新设置这个标志。

下图展示了导航条变暗后的展示效果(注意这里只是将状态条隐藏了，并不是使它变暗了)。注意导航条(图像的右边)在这里呈浅白色的点：
![](http://android.xsoftlab.net/images/training/low_profile_hide2x.png)

下图展示了相同的图像，只是系统条这时完全被展示了出来：
![](http://android.xsoftlab.net/images/training/low_profile_show2x.png)

##使状态导航条变亮
如果你想清除这个标志，你可以这么做：
```java
View decorView = getActivity().getWindow().getDecorView();
// Calling setSystemUiVisibility() with a value of 0 clears
// all flags.
decorView.setSystemUiVisibility(0);
```

