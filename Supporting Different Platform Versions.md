原味地址 : [http://android.xsoftlab.net/training/basics/supporting-devices/platforms.html](http://android.xsoftlab.net/training/basics/supporting-devices/platforms.html)

每当APP使用了Android所提供的最新版的API时，应用应该继续对老版本提供支持，直到所有的设备都更新到最新版。这一节将会展示如何采用最先进的新版本API时还能继续良好的支持老版本。

[Platform Versions](http://developer.android.com/intl/zh-cn/about/dashboards/index.html) 的信息图表会基于访问Google Play Store(谷歌应用商店)的许多设备从而有规律的统计更新Android每一个版本的活跃设备分布图。这对于更新APP编译环境到最新Android版本而且还可以支持90%以上的活跃设备来说是最好的实践。

> **Tips:**为了可以在若干个Android版本上还可以提供最佳的特性与功能，你应该在APP中使用 [Android Support Library](http://android.xsoftlab.net/tools/support-library/index.html)(Android支持库)，它可以使得你可以在旧版本上使用若干个较新的平台API。

##指定最低API等级与目标API等级
文件AndroidManifest.xml详细描述了APP相关的信息以及支持的Android版本。特别的，< uses-sdk标签中的minSdkVersion，targetSdkVersion属性分别指明了APP兼容的最低版本以及最高版本:
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" ... >
    <uses-sdk android:minSdkVersion="4" android:targetSdkVersion="15" />
    ...
</manifest>
```

每当Android的新版本发布，一些风格与习惯可能会被改变。为了允许APP采用这些更为优秀的变化，以确保APP对每一台用户设备进行风格匹配，你应该设置targetSdkVersion的值为最新的安卓可用版本。

##在运行时检查系统版本
Android在Build常量类中提供了每一个平台版本的唯一编码。在APP中使用这些编码以确保这些API在当前的系统上是可用的。

```java
private void setUpActionBar() {
    // Make sure we're running on Honeycomb or higher to use ActionBar APIs
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        ActionBar actionBar = getActionBar();
        actionBar.setDisplayHomeAsUpEnabled(true);
    }
}
```

> **Note:**当解析XML资源时，Android会自动忽略当前设备不支持的xml属性。所以在使用XML属性的时候可以完全放心。举个例子，如果你设置了targetSdkVersion="11"，然后包含了ActionBar的APP默认是跑在Android 3.0及以上的。然后为了向ActionBar上添加菜单按钮，你需要在菜单资源XML文件中添加android:showAsAction="ifRoom"。在交叉版本的XML文件中这样做是安全的，因为老版本的Android平台会自动忽略showAsAction 属性(所以，你就不用专门再在res/menu-v11/中做区分)。

##使用平台风格和主题
Android为APP提供了与底层系统感官上相一致的用户体验主题。这些主题可以通过清单文件应用到APP中。通过使用这些内嵌的风格和主题，你的APP很自然的可以和最新的Android版本在感官上保持一致。

如果想使Activity看起来像对话框：
```xml
<activity android:theme="@android:style/Theme.Dialog">
```

如果想使Activity有一个透明的背景：
```xml
<activity android:theme="@android:style/Theme.Translucent">
```

如果要使用在/res/values/styles.xml下定义的自定义主题：
```xml
<activity android:theme="@style/CustomTheme">
```

如果要将自定义主题应用到整个APP中，在< application>标签中添加 android:theme 属性：
```xml
<application android:theme="@style/CustomTheme">
```

更多有关创建和使用主题的相关信息，请参见指南： [Styles and Themes](http://android.xsoftlab.net/guide/topics/ui/themes.html) 