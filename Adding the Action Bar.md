## 导言- 添加ActionBar
原文地址：[http://android.xsoftlab.net/training/basics/actionbar/index.html](http://android.xsoftlab.net/training/basics/actionbar/index.html "http://android.xsoftlab.net/training/basics/actionbar/index.html")

ActionBar是很多重要的特性之一，你可以用它实现用户的自定义行为。它提供了若干的用户界面特性，以便你的应用可以很快的提供与其它应用很相似的用户界面。关键功能包括：

- 在应用内部有一块专门的空间用来展示应用的标志以及知识用户所在的当前位置。
- 以可预测的方式访问一些重要的行为(比如搜索)。
- 支持导航以及界面变换(通过tabs或者是下拉列表)

![](http://android.xsoftlab.net/images/training/basics/actionbar-actions.png)

这节训练课程提供了最基础的ActionBar入门指南，如果要查看更多的相关信息及特征，请移步：[http://android.xsoftlab.net/guide/topics/ui/actionbar.html](http://android.xsoftlab.net/guide/topics/ui/actionbar.html "http://android.xsoftlab.net/guide/topics/ui/actionbar.html")

#Setting Up the Action Bar
原文地址：[http://android.xsoftlab.net/training/basics/actionbar/setting-up.html](http://android.xsoftlab.net/training/basics/actionbar/setting-up.html "http://android.xsoftlab.net/training/basics/actionbar/setting-up.html")

在很多设计格式中，ActionBar用来展示Activity的标题，然后APP的图标会被放置在左边。甚至在这个简单的样式中，ActionBar对告知用户他们所在的当前位置来说是很有用的。而且它还为你的应用保持了一致的身份标示。
![这张图展示了含有图标和标题的ActionBar](http://android.xsoftlab.net/images/training/basics/actionbar-basic.png)

如果要设置基本的ActionBar，那需要你的应用使用含有并可用ActionBar的Activity主题。

## 支持Android 3.0及更高版本
从Android 3.0开始，ActionBar特性被包含在了所有使用了Theme.Holo主题的Activity中。当设置了targetSdkVersion 或 minSdkVersion为11或更高的版本中它是默认主题。

所以要在你的Activity中使用ActionBar的话，需要简单设置一下targetSdkVersion，minSdkVersion：
```xml
<manifest ... >
    <uses-sdk android:minSdkVersion="11" ... />
    ...
</manifest>
```
> 如果你创建了自定义主题，那么请确保它的父主题是Theme.Holo类主题之一。

就这样，现在Theme.Holo主题变应用到你的APP中了，然后所有的Activity都会显示ActionBar.

## 支持Android 2.1及以上版本
如果要在Android 2.1以上，3.0一下的APP版本中添加ActionBar,需要在APP的工程中添加Android支持库Android Support Library。

为了开始，请阅读文档[http://android.xsoftlab.net/tools/support-library/setup.html](http://android.xsoftlab.net/tools/support-library/setup.html "Support Library Setup")，然后设置v7 appcompat库。

添加支持库并集成到你的APP工程中之后：

1.更新Activity让它继承ActionBarActivity，比如：
```
public class MainActivity extends ActionBarActivity { ... }
```
2.在你的清单文件中，更新< application>或者单个的< activity>使用Theme.AppCompat主题：

```<activity android:theme="@style/Theme.AppCompat.Light" ... >
```
现在你的Activity便在运行Android 2.1或者更高的版本上有了ActionBar.

别忘了在你的清单文件中设置适当的API等级：
```
<manifest ... >
    <uses-sdk android:minSdkVersion="7"  android:targetSdkVersion="18" />
    ...
</manifest>
```
