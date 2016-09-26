原文地址：[http://android.xsoftlab.net/training/multiscreen/index.html](http://android.xsoftlab.net/training/multiscreen/index.html)

#引言
Android运行于数以百计不同尺寸的设备上。范围小到手持移动电话，大到电视设备。因此，在设计APP时应当兼顾到尽可能多的屏幕尺寸。这样才能照顾到较多的潜在用户。

但是仅仅考虑不同的设备类型还不够。每一种尺寸为用户提供了不同的可能性与挑战，所以为了使用户感到满意，应用程序需要做的不单单是支持多样的屏幕：它还必须对每种屏幕结构将用户体验优化到最佳。

这节课将会学习如何实现针对屏幕结构优化的用户界面。

> **Note:** 这节课与相关示例程序均使用的是[support library](http://android.xsoftlab.net/tools/support-library/index.html)。

#支持不同的屏幕尺寸
这节课将会学习通过以下方式来支持不同的屏幕尺寸：

- 确保布局可以灵活的调整尺寸。
- 对不同的屏幕结构提供适当的UI布局。
- 确保在正确的屏幕中使用了正确的布局。
- 提供可以正常缩放的位图。

##使用"wrap_content"及"match_parent"
为了使布局可以灵活的适配不同的屏幕尺寸，应当对某些View组件的width，height属性使用"wrap_content"或"match_parent"。如果使用了"wrap_content"，那么View的高宽会被设置为View内容所需的最小尺寸。然而"match_parent"会使View的高宽扩展到父布局的尺寸大小。

通过使用"wrap_content"或"match_parent"可以使View高宽扩展到View所需要的大小或者扩展到父布局的可用空间：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout android:layout_width="match_parent" 
                  android:id="@+id/linearLayout1"  
                  android:gravity="center"
                  android:layout_height="50dp">
        <ImageView android:id="@+id/imageView1" 
                   android:layout_height="wrap_content"
                   android:layout_width="wrap_content"
                   android:src="@drawable/logo"
                   android:paddingRight="30dp"
                   android:layout_gravity="left"
                   android:layout_weight="0" />
        <View android:layout_height="wrap_content" 
              android:id="@+id/view1"
              android:layout_width="wrap_content"
              android:layout_weight="1" />
        <Button android:id="@+id/categorybutton"
                android:background="@drawable/button_bg"
                android:layout_height="match_parent"
                android:layout_weight="0"
                android:layout_width="120dp"
                style="@style/CategoryButtonStyle"/>
    </LinearLayout>
    <fragment android:id="@+id/headlines" 
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="match_parent" />
</LinearLayout>
```
注意示例中是如何使用"wrap_content"及"match_parent"的。这可以使布局正确的适配不同的屏幕尺寸及方向。

下图是布局在垂直及水平方向的示例。注意View的尺寸会自动适配屏幕的高宽:
![](http://android.xsoftlab.net/images/training/layout-hvga.png)

##使用RelativeLayout
你可以使用[LinearLayout](http://android.xsoftlab.net/reference/android/widget/LinearLayout.html)结合"wrap_content"或"match_parent"构造相对复杂的布局。然而，[LinearLayout](http://android.xsoftlab.net/reference/android/widget/LinearLayout.html)不能够精确的控制子View的相对关系。在[LinearLayout](http://android.xsoftlab.net/reference/android/widget/LinearLayout.html)中View只能简单的被线性排列。如果需要调整View间的相对关系，一种较好的解决方式就是使用[RelativeLayout](http://android.xsoftlab.net/reference/android/widget/RelativeLayout.html)，它允许指定View间的相对关系。下面的示例中，你可以指定一个View靠一个View的左边布置，而另一个View靠屏幕的右边布置。
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/label"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Type here:"/>
    <EditText
        android:id="@+id/entry"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/label"/>
    <Button
        android:id="@+id/ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/entry"
        android:layout_alignParentRight="true"
        android:layout_marginLeft="10dp"
        android:text="OK" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_toLeftOf="@id/ok"
        android:layout_alignTop="@id/ok"
        android:text="Cancel" />
</RelativeLayout>
```

下图是该布局在QVGA屏幕中的显示效果：
![](http://android.xsoftlab.net/images/training/relativelayout1.png)

下图是该布局在大屏幕中的显示效果：
![](http://android.xsoftlab.net/images/training/relativelayout2.png)

要注意虽然这些View的尺寸发生了改变，但是其它之间的相对关系还是保留了下来。

##使用尺寸限定符
上面我们学习了如何利用灵活布局或者相对布局来匹配不同的屏幕，然而这对于匹配任何屏幕来说还不够好。因此，应用程序不单单只是实现灵活的布局，还应该对不同的屏幕配置提供相应的布局。可以通过[configuration qualifiers](http://developer.android.com/guide/practices/screens_support.html#qualifiers)中所描述的内容学习具体细节，它可以使程序在运行时根据当前的屏幕配置来自动选择对应的资源。

比如说，很多应用程序针对于大屏幕实现了"two pane"的模式。平板与电视大到足以同时显示两个面板，但是移动电话只能同时显示其中一个。所以，要实现这种布局，项目中应当含有以下文件：

- res/layout/main.xml，单面板布局(默认)：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="match_parent" />
</LinearLayout>
```

- res/layout-large/main.xml,双面板布局：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="400dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
```

要注意第二个布局的目录路径的large标识符。这个布局会在屏幕类型为large时被采用(比如，7英寸的平板或者更大的设备)。其它布局则会被小型设备所采用。

##使用最小宽度标识符
