原文地址：[http://android.xsoftlab.net/training/system-ui/status.html](http://android.xsoftlab.net/training/system-ui/status.html)

这节课介绍了在不同的版本中隐藏状态条。隐藏状态条可以使内容展示区域更大，因此可以提供更强的身临其境的用户体验。

含有状态条的APP：

![](http://android.xsoftlab.net/images/training/status_bar_show.png)

隐藏状态条的APP，注意这里的ActionBar同样也隐藏了。绝不要在没有状态条的时候还显示ActionBar:

![](http://android.xsoftlab.net/images/training/status_bar_hide.png)

##在Android 4.0以下的版本中隐藏状态条
你可以通过设置[WindowManager](http://android.xsoftlab.net/reference/android/view/WindowManager.html)的标志来隐藏Android 4.0之前的状态条。你可以通过这项代码动态设置或者在清单文件中设置Activity的主题也可以实现。在清单文件中设置主题是一种首选方式，如果状态条一直保持隐藏状态。严格意义上说，你如果想的话，你可以动态的重写主题。
```xml
<application
    ...
    android:theme="@android:style/Theme.Holo.NoActionBar.Fullscreen" >
    ...
</application>
```

使用Activity主题的优势在于：

- 相对于动态设置来说更易于维护也减小了出错的风险。
- 这会使得UI转场更加平滑，因为系统在实例化Activity之前得到了它需要的UI渲染信息。

或者你可以动态设置[WindowManager](http://android.xsoftlab.net/reference/android/view/WindowManager.html)标志。这种方法适合在用户与APP交互的时候更加方便的隐藏或者显示状态条。
```java
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // If the Android version is lower than Jellybean, use this call to hide
        // the status bar.
        if (Build.VERSION.SDK_INT < 16) {
            getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                    WindowManager.LayoutParams.FLAG_FULLSCREEN);
        }
        setContentView(R.layout.activity_main);
    }
    ...
}
```

当你设置了[WindowManager](http://android.xsoftlab.net/reference/android/view/WindowManager.html)标志(不论是通过Activity的主题还是动态的设置)，该标志则会一直保留该效果，知道你将该标志移除。

你可以使用标志[FLAG_LAYOUT_IN_SCREEN](http://android.xsoftlab.net/reference/android/view/WindowManager.LayoutParams.html#FLAG_LAYOUT_IN_SCREEN)来使Activity的布局使用与[FLAG_FULLSCREEN](http://android.xsoftlab.net/reference/android/view/WindowManager.LayoutParams.html#FLAG_FULLSCREEN)标志相同的屏幕区域。这可以防止在状态条隐藏或者展示的时候重新调整布局的尺寸。

##在Android 4.1以上的版本中隐藏状态条
你可以通过使用[setSystemUiVisibility()](http://android.xsoftlab.net/reference/android/view/View.html#setSystemUiVisibility(int))方法隐藏Android 4.1以上系统的状态条。[setSystemUiVisibility()](http://android.xsoftlab.net/reference/android/view/View.html#setSystemUiVisibility(int))为独立的View层级设置了UI标志;这些设置被整合进Window的等级。使用[setSystemUiVisibility()](http://android.xsoftlab.net/reference/android/view/View.html#setSystemUiVisibility(int))比使用[WindowManager](http://android.xsoftlab.net/reference/android/view/WindowManager.html)的标志控制粒度更细。下面的代码用户隐藏状态条：
```java
View decorView = getWindow().getDecorView();
// Hide the status bar.
int uiOptions = View.SYSTEM_UI_FLAG_FULLSCREEN;
decorView.setSystemUiVisibility(uiOptions);
// Remember that you should never show the action bar if the
// status bar is hidden, so hide that too if necessary.
ActionBar actionBar = getActionBar();
actionBar.hide();
```

要注意以下几个方面：
- 一旦UI的标志被清除(比如，通过导航离开了Activity)，如果你想重新隐藏状态条的话则需要重新设置这些标志。
- 在不同的地方设置UI标志还有些差异。如果在Activity的[onCreate()](http://android.xsoftlab.net/reference/android/app/Activity.html#onCreate(android.os.Bundle))方法中隐藏了系统条，这时用户返回了桌面，那么系统条则会再次出现。当用户重新打开Activity时，[onCreate()](http://android.xsoftlab.net/reference/android/app/Activity.html#onCreate(android.os.Bundle))不会被再次调用，所以系统条会一直保持可见。如果你想使系统的UI还保留原来的状态，则可以在[onResume()](http://android.xsoftlab.net/reference/android/app/Activity.html#onResume())中或[onWindowFocusChanged()](http://android.xsoftlab.net/reference/android/view/Window.Callback.html#onWindowFocusChanged(boolean))中设置相应的标志。
- [setSystemUiVisibility()](http://android.xsoftlab.net/reference/android/view/View.html#setSystemUiVisibility(int))只有在View可见的时候设置才有效。
- 由导航的方式离开View会使由[setSystemUiVisibility()](http://android.xsoftlab.net/reference/android/view/View.html#setSystemUiVisibility(int))方法设置的标志被清除。

##使内容出现在状态条的后面
在Android 4.1以上的版本中，你可以将应用的内容区域显示在状态条的后面，所以内容区域的尺寸并不会随着状态条的隐藏显示而变化。通过使用[SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN](http://android.xsoftlab.net/reference/android/view/View.html#SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN)标志实现这一点。你可能还需要使用[SYSTEM_UI_FLAG_LAYOUT_STABLE](http://android.xsoftlab.net/reference/android/view/View.html#SYSTEM_UI_FLAG_LAYOUT_STABLE)标志来辅助APP维持一个稳定的布局状态。

当你使用了这项方法，你有责任确保APP UI的边界部分不会被系统条所遮盖。这会使PP不可用。在很多情况下，你可以通过在布局文件中添加android:fitsSystemWindows=true属性来处理这种情况。它会调整父ViewGroup的内边距来流出系统窗口的空间，这足以应付大多数的应用。

在另外一些情况中，你还需要修改默认的内边距来获得期望的布局。为了直接控制内容与系统条的相对关系，重写[fitSystemWindows(Rect insets)](http://android.xsoftlab.net/reference/android/view/View.html#fitSystemWindows(android.graphics.Rect))方法，该方法在内容区域所设置的窗口发生变化时会被调用，为了允许窗口可以调整内容区域。通过重写该方法你可以处理你想如何调整。

##随着ActionBar的变换同步状态条
在Android 4.1以上的版本中，为了避免重新调整布局的尺寸，当ActionBar显示或隐藏时，你可以开启ActionBar的Overlay Mode。当处于Overlay Mode下时，Activity的布局会使用所有的可用空间，就仿佛ActionBar不存在一样，系统会将ActionBar绘制在布局的上面一层。这会使布局顶部的部分变的模糊，不过当ActionBar显示或者隐藏时，系统并不会重写调整布局的尺寸，这会使转场无缝对接。

为了可以使ActionBar开启Overlay Mode，你需要创建一个自定义主题，并需要继承已有的携带ActionBar的主题，并需要将android:windowActionBarOverlay属性设置为true。