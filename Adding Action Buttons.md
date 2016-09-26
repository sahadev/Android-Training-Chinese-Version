##添加Action按钮
ActionBar按钮允许在当前的APP上下文内添加很多重要的功能按钮。这样便会通过图标或者文字作为功能按钮直接出现在ActionBar上。功能按钮如果没有空间或者是不足够重要的按钮都会隐藏在隐藏按钮下。

![](http://android.xsoftlab.net/images/training/basics/actionbar-actions.png)

##在XML指定功能
所有的功能按钮和其它在隐藏按钮下的功能按钮都可以通过XML[菜单资源](http://android.xsoftlab.net/guide/topics/resources/menu-resource.html)来定义。为了添加功能到ActionBar，需要在res/menu/目录下创建一个新的xml文件。

为每一个需要添加到ActionBar的按钮添加一个< item>标签：

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <!-- Search, should appear as action button -->
    <item android:id="@+id/action_search"
          android:icon="@drawable/ic_action_search"
          android:title="@string/action_search"
          android:showAsAction="ifRoom" />
    <!-- Settings, should always be in the overflow -->
    <item android:id="@+id/action_settings"
          android:title="@string/action_settings"
          android:showAsAction="never" />
</menu>
```

这里声明了当ActionBar有可用空间的时候Search功能应该作为一个按钮放在ActionBar上。但是设置按钮会总是出现在下拉列表中(默认情况下，所有的功能都会出现在下拉列表中，这对于每一个功能显示的声明你的设计意图是最好的练习)。

icon属性这里要求一个图片的资源ID，这里跟的是@drawable/name,这里的name必须是保存在工程中res/drawable/目录下保存的位图图像名称。比如"@drawable/ic_action_search"就是引用了一个名为ic_action_search.png的图片资源。很相似的，title属性也是使用一个在XML文件中定义的字符串资源。

> 当为应用创建图标或者其它的位图图像时，这对于提供不同版本的屏幕密度资源来说是很重要的一点。

如果为了兼容像Android 2.1这种第版本而使用了支持库，showAsAction属性对于命名空间android:是不可用的。如果要使用支持库必须要在XML中定义自定义的XML命名空间，然后使用这个命名空间的标识符(自定义的XML命名空间应该基于应用的名称，如果只是在这个文件下操作的话，可以命名为你想命名的任何名称)：

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:yourapp="http://schemas.android.com/apk/res-auto" >
    <!-- Search, should appear as action button -->
    <item android:id="@+id/action_search"
          android:icon="@drawable/ic_action_search"
          android:title="@string/action_search"
          yourapp:showAsAction="ifRoom"  />
    ...
</menu>
```
##添加功能按钮到ActionBar上
为了将菜单按钮直接放置到ActionBar上，需要重写实现activity中的onCreateOptionsMenu()方法，然后加载菜单资源到回调方法的参数menu对象中：
```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate the menu items for use in the action bar
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.main_activity_actions, menu);
    return super.onCreateOptionsMenu(menu);
}
```
##响应功能按钮
当用户点击了其中一个功能按钮或者其它下拉列表按钮之后，然后系统会调用activity中的
onOptionsItemSelected()方法。在实现的这个方法中，调用回调参数MenuItem的getItemId()方法返回的值与ID进行匹配，去决定是哪个功能按钮按下的。这里的ID便是在XML中声明的< item>标签中的android:id属性中定义的。
```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    // Handle presses on the action bar items
    switch (item.getItemId()) {
        case R.id.action_search:
            openSearch();
            return true;
        case R.id.action_settings:
            openSettings();
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```
##为低等级的Activity添加返回按钮
![Gmail的返回按钮](http://android.xsoftlab.net/images/ui/actionbar-up.png)

**Note**：Gmail的返回按钮

应用中的所有Activity不都是应用的主入口，所以应该在ActionBar上提供一个返回按钮，以便让用户可以通过点击返回按钮返回到上一个屏幕页面。

当运行在Android 4.1或者更高版本上，又或者是使用了ActionBarActivity的支持库的应用，可以在清单文件中简单的申明ActionBar的返回按钮所要返回的界面：
```xml
<application ... >
    ...
    <!-- The main/home activity (it has no parent activity) -->
    <activity
        android:name="com.example.myfirstapp.MainActivity" ...>
        ...
    </activity>
    <!-- A child of the main activity -->
    <activity
        android:name="com.example.myfirstapp.DisplayMessageActivity"
        android:label="@string/title_activity_display_message"
        android:parentActivityName="com.example.myfirstapp.MainActivity" >
        <!-- Parent activity meta-data to support 4.0 and lower -->
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
</application>
```
然后通过setDisplayHomeAsUpEnabled()设置返回按钮为可用状态：
```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_displaymessage);
    getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    // If your minSdkVersion is 11 or higher, instead use:
    // getActionBar().setDisplayHomeAsUpEnabled(true);
}
```
因为现在系统知道了DisplayMessageActivity的父界面为MainActivity，当用户按下了返回按钮，然后系统会适当的导航界面到MainActivity，所以你不需要自己处理返回按钮的点击事件。

关于导航按钮的更多信息，请参见：[Providing Up Navigation](http://android.xsoftlab.net/training/implementing-navigation/ancestral.html "Providing Up Navigation")



