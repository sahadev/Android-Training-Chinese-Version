原文地址：[http://android.xsoftlab.net/training/keyboard-input/visibility.html](http://android.xsoftlab.net/training/keyboard-input/visibility.html)

当输入的焦点进入或者离开文本框时，Android会适当的显示或隐藏输入法。系统还会决定UI及文本框如何出现在输入法的上方。比如，当垂直方向上的可用空间非常紧张时，那么文本框可能就会填充输入法上方的整个区域。对于大多数的APP来说，这样的默认行为是它们所需要的。

在一些情况中，你想直接控制输入法的可见性，以及希望当输入法可见的时候如何布局你的UI。那么这节课主要就是介绍如何控制以及响应输入法的可见性。

##当Activity启动的时候显示输入法
尽管在Activity启动的时候Android将焦点给了第一个文本框，但是它是不会引起输入法显示的。这样的行为是符合正常的习惯的，因为进入文本框可能不会Activity启动后的首要任务。不管怎样，如果进入文本框是Activity的首要任务的话(比如登录界面)，那么你可能希望默认情况下进入Activity后就会弹出输入法。

为了在Activity启动后可以显示输入法，需要在清单文件中对应的Activity的元素中添加属性[android:windowSoftInputMode](http://android.xsoftlab.net/guide/topics/manifest/activity-element.html#wsoft)。如下：
```xml
<application ... >
    <activity
        android:windowSoftInputMode="stateVisible" ... >
        ...
    </activity>
    ...
</application>
```

> **Note:**如果用户的设备含有实体按键，那么软键盘是不会弹出的。

##按需求弹出输入法
如果在Activity的生命周期内有这么一个方法，你希望确保在该方法调用后输入法是可见的，那么你可以使用[InputMethodManager](http://android.xsoftlab.net/reference/android/view/inputmethod/InputMethodManager.html)来将它弹出。

举个例子，下面的方法持有了一个View对象，用户会在这个View内部输入点什么，所以调用[requestFocus()](http://android.xsoftlab.net/reference/android/view/View.html#requestFocus())方法可以将焦点赋给它，然后[showSoftInput()](http://android.xsoftlab.net/reference/android/view/inputmethod/InputMethodManager.html#showSoftInput(android.view.View, int))就会将输入法打开：
```java
public void showSoftKeyboard(View view) {
    if (view.requestFocus()) {
        InputMethodManager imm = (InputMethodManager)
                getSystemService(Context.INPUT_METHOD_SERVICE);
        imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT);
    }
}
```

> **Note:** 一旦输入法被弹出，那么最好不要使用代码隐藏它。系统会在用户终止文本框中的任务后将它隐藏或者用户通过系统隐藏了它(比如使用了返回按钮)。

##指明你的UI应该如何响应
当输入法出现在屏幕上时，它会减少APP在屏幕上的使用空间。那么系统会决定如何调整UI的部分区域，但是它可能不是最准确的。为了确保APP拥有最佳的用户体验，应该指明系统如何调整UI。

为了声明Activity的首选方式，应当在项目的清单文件中对应的Activity下添加[android:windowSoftInputMode](http://android.xsoftlab.net/guide/topics/manifest/activity-element.html#wsoft)属性，并使用其中一个含有"adjust"的值。

举个例子，为了确保系统可将UI调整到可用区域，应当使用"adjustResize"：
```xml
<application ... >
    <activity
        android:windowSoftInputMode="adjustResize" ... >
        ...
    </activity>
    ...
</application>
```

除了以上的方法，你还可以使用组合的的方式来声明UI调整规则与输入法的可见性规则：
```xml
    <activity
        android:windowSoftInputMode="stateVisible|adjustResize" ... >
        ...
    </activity>
```

指明"adjustResize"是很重要的：如果UI中含有一些用户可能需要迅速访问的按键或者需要操作的文本框的话。举个例子，如果你使用相对布局将一个按钮放置到了屏幕的底部，那么使用"adjustResize"调整布局可以使该按钮出现在输入法的顶部，这样在输入完成之后可以直接点击该按钮。