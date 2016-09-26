原文地址：[http://android.xsoftlab.net/training/improving-layouts/reusing-layouts.html](http://android.xsoftlab.net/training/improving-layouts/reusing-layouts.html)

尽管Android提供了种类繁多的可重用控件，但是有时你可能希望重用那些指定的布局。如果要重用这些布局，你可以使用< include/>标签与< merge/>标签，它们可将另外的布局嵌入进当前的布局中。

可重用布局这项功能特别强大，它可以使你创建那些复杂的可重用布局。比方说，一个含有yes和no按钮的容器或者一个含有progressBar及一个文本框的容器。它还意味着程序可以对这些布局进行单独控制。所以，虽然说你可以通过自定义View的方式来实现更为复杂的UI组件，但是使用重用布局的方法会更简便一些。

##创建一个可重用的布局
如果你已经知道哪一个布局需要重用，那么就创建一个新的xml文件用来定义这个布局。下面就定义了一个ActionBar的布局文件，众所周知，ActionBar是会在每个Activity中统一出现的：

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width=”match_parent”
    android:layout_height="wrap_content"
    android:background="@color/titlebar_bg">
    <ImageView android:layout_width="wrap_content"
               android:layout_height="wrap_content" 
               android:src="@drawable/gafricalogo" />
</FrameLayout>
```

##使用< include/>标签
在希望添加可重用布局的布局内，添加< include/>标签。下面的例子就是将上面的布局加入到了一个布局当中：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" 
    android:layout_width=”match_parent”
    android:layout_height=”match_parent”
    android:background="@color/app_bg"
    android:gravity="center_horizontal">
    <include layout="@layout/titlebar"/>
    <TextView android:layout_width=”match_parent”
              android:layout_height="wrap_content"
              android:text="@string/hello"
              android:padding="10dp" />
    ...
</LinearLayout>
```

