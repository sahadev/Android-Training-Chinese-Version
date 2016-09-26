原文地址 : [http://android.xsoftlab.net/training/basics/actionbar/styling.html](http://android.xsoftlab.net/training/basics/actionbar/styling.html)

##ActionBar的样式
ActionBar提供了为用户提供了常见的习惯性的用户界面以及按钮功能。但是这并不意味着必须要和其它APP看起来一模一样。如果需要设计更符合产品品牌样式风格的话，ActionBar也可以做到，你可以通过Android的[style and theme](http://android.xsoftlab.net/guide/topics/ui/themes.html "style and theme")很容易做到这一点。

Android已经包含了一少部分的内置Activity主题，这些主题包含了"dark"或者"light"的ActionBar风格。你也可以通过继承这些主题来进一步的自定义自己ActionBar。
> **Note：**如果使用的是ActionBar的支持库，那么你必须使用[Theme.AppCompat ](http://android.xsoftlab.net/reference/android/support/v7/appcompat/R.style.html#Theme_AppCompat)家族的风格。在这样的情况下，每一个样式属性必须声明两次：一次是在平台样式属性中，一次是在支持库样式属性中。相关情况请看下面的样例。

##使用一个Android主题
![](http://android.xsoftlab.net/images/training/basics/actionbar-theme-dark@2x.png)

![](http://android.xsoftlab.net/images/training/basics/actionbar-theme-light-solid@2x.png)

Android内置了两种最基本的Activity主题，它们区别在于ActionBar的颜色：

- Theme.Holo 是"dark"主题。


- Theme.Holo.Light 是"light"主题。

你可以将这些主题应用到整个APP中或者是单个Activity中，通过在清单文件的< application>元素或是< activity>元素内部使用android:theme属性来指定：
```xml
<application android:theme="@android:style/Theme.Holo.Light" ... />
```

![](http://android.xsoftlab.net/images/training/basics/actionbar-theme-light-darkactionbar@2x.png)

你也可以通过使用Theme.Holo.Light.DarkActionBar主题来达到ActionBar是"dark"样式而Activity的其余部分则是"light"样式的效果。

如果是使用了支持库的话，必须使用Theme.AppCompat下的主题：

- Theme.AppCompat 对应的是"dark"主题.
- Theme.AppCompat.Light 对应的是"light"主题.
- Theme.AppCompat.Light.DarkActionBar 对应的是带着黑色ActionBar的亮色主题.

请确保在ActionBar上图标颜色与ActionBar的颜色有适当的对比度。为了辅助达到这一点， [Action Bar Icon Pack](http://android.xsoftlab.net/design/downloads/index.html#action-bar-icon-pack)包含了一些用在ActionBar上的标准的功能按钮。

##自定义ActionBar的背景色
![](http://android.xsoftlab.net/images/training/basics/actionbar-theme-custom@2x.png)

为了改变ActionBar的背景色，需要创建一个自定义的主题，然后重写actionBarStyle属性。这个属性指向了其它的背景样式,您可以在其它背景样式中重写background属性来给ActionBar指定一个图像资源。

如果APP使用了navigation tabs或者 split action bar，你也可以分别通过backgroundStacked和backgroundSplit属性指定他们的背景色。

> **注意:** 选择适合的父类主题对于自定义主题来说很重要，当继承主题之后，所有的样式都会被继承下来。如果没有父类主题，除非很明确的声明了每一项样式，否则ActionBar会丢失很多样式属性。

###为Android 3.0及以上版本定义
当仅仅支持了Android 3.0及更高的版本，你可以像这样定义ActionBar的背景：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@android:style/Theme.Holo.Light.DarkActionBar">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
    </style>
    <!-- ActionBar styles -->
    <style name="MyActionBar"
           parent="@android:style/Widget.Holo.Light.ActionBar.Solid.Inverse">
        <item name="android:background">@drawable/actionbar_background</item>
    </style>
</resources>
```

然后应用这个主题到整个APP中或者是单个Activity中：

```xml
<application android:theme="@style/CustomActionBarTheme" ... />
```

###为Android 2.1及以上版本定义
当使用了支持库，它的修改方式和上面很相似：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@style/Theme.AppCompat.Light.DarkActionBar">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
        <!-- Support library compatibility -->
        <item name="actionBarStyle">@style/MyActionBar</item>
    </style>
    <!-- ActionBar styles -->
    <style name="MyActionBar"
           parent="@style/Widget.AppCompat.Light.ActionBar.Solid.Inverse">
        <item name="android:background">@drawable/actionbar_background</item>
        <!-- Support library compatibility -->
        <item name="background">@drawable/actionbar_background</item>
    </style>
</resources>
```

然后应用这个主题到整个APP中或者是单个Activity中：

```xml
<application android:theme="@style/CustomActionBarTheme" ... />
```

##自定义字体颜色
为了改变ActionBar上的字体颜色，你需要单独重写text元素的每一个属性：

- ActionBar标题：创建自定义的样式，然后指定textColor的值，然后将这个样式应用到你自己定义的主题actionBarStyle中的titleTextStyle属性里。

> **注意:**如果要使用应用到titleTextStyle中的自定义样式，那么应该使这个样式继承TextAppearance.Holo.Widget.ActionBar.Title。


- ActionBar的标签：重写主题中的actionBarTabTextStyle。
- ActionBar中的按钮：重写主题中的actionMenuTextColor。

###为Android 3.0及以上版本定义
当仅仅支持了Android 3.0及更高的版本，你的XML风格文件应该使这样的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@style/Theme.Holo">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
        <item name="android:actionBarTabTextStyle">@style/MyActionBarTabText</item>
        <item name="android:actionMenuTextColor">@color/actionbar_text</item>
    </style>
    <!-- ActionBar styles -->
    <style name="MyActionBar"
           parent="@style/Widget.Holo.ActionBar">
        <item name="android:titleTextStyle">@style/MyActionBarTitleText</item>
    </style>
    <!-- ActionBar title text -->
    <style name="MyActionBarTitleText"
           parent="@style/TextAppearance.Holo.Widget.ActionBar.Title">
        <item name="android:textColor">@color/actionbar_text</item>
    </style>
    <!-- ActionBar tabs text styles -->
    <style name="MyActionBarTabText"
           parent="@style/Widget.Holo.ActionBar.TabText">
        <item name="android:textColor">@color/actionbar_text</item>
    </style>
</resources>
```

###为Android 2.1及以上版本定义
当使用了支持库，你的XML风格文件应该使这样的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@style/Theme.AppCompat">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
        <item name="android:actionBarTabTextStyle">@style/MyActionBarTabText</item>
        <item name="android:actionMenuTextColor">@color/actionbar_text</item>
        <!-- Support library compatibility -->
        <item name="actionBarStyle">@style/MyActionBar</item>
        <item name="actionBarTabTextStyle">@style/MyActionBarTabText</item>
        <item name="actionMenuTextColor">@color/actionbar_text</item>
    </style>
    <!-- ActionBar styles -->
    <style name="MyActionBar"
           parent="@style/Widget.AppCompat.ActionBar">
        <item name="android:titleTextStyle">@style/MyActionBarTitleText</item>
        <!-- Support library compatibility -->
        <item name="titleTextStyle">@style/MyActionBarTitleText</item>
    </style>
    <!-- ActionBar title text -->
    <style name="MyActionBarTitleText"
           parent="@style/TextAppearance.AppCompat.Widget.ActionBar.Title">
        <item name="android:textColor">@color/actionbar_text</item>
        <!-- The textColor property is backward compatible with the Support Library -->
    </style>
    <!-- ActionBar tabs text -->
    <style name="MyActionBarTabText"
           parent="@style/Widget.AppCompat.ActionBar.TabText">
        <item name="android:textColor">@color/actionbar_text</item>
        <!-- The textColor property is backward compatible with the Support Library -->
    </style>
</resources>
```
##自定义标签指示器

![](http://android.xsoftlab.net/images/training/basics/actionbar-theme-custom-tabs@2x.png)

如果要改变导航标签navigation tabs的指示器的话，创建一个activity主题，然后重写actionBarTabStyle属性。这个属性会指向其它样式资源。这个样式资源需要重写background属性，然后并设置它的值为一个状态列表图像资源。
> **Note:**状态列表图像是很重要的。它可以在标签选中的情况下使用户区别出选中的标签与未选中便签的不同，从而判断是哪个便签被选中。有关如何创建一个作用于展示多种按钮状态的图像资源，请参见[State List](http://android.xsoftlab.net/guide/topics/resources/drawable-resource.html#StateList)文档。

举个例子，以下是一个状态列表图像资源文件的内容。这里声明了用作于ActionBar标签的若干个状态，每个状态都指定了一张背景图：
```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
<!-- STATES WHEN BUTTON IS NOT PRESSED -->
    <!-- Non focused states -->
    <item android:state_focused="false" android:state_selected="false"
          android:state_pressed="false"
          android:drawable="@drawable/tab_unselected" />
    <item android:state_focused="false" android:state_selected="true"
          android:state_pressed="false"
          android:drawable="@drawable/tab_selected" />
    <!-- Focused states (such as when focused with a d-pad or mouse hover) -->
    <item android:state_focused="true" android:state_selected="false"
          android:state_pressed="false"
          android:drawable="@drawable/tab_unselected_focused" />
    <item android:state_focused="true" android:state_selected="true"
          android:state_pressed="false"
          android:drawable="@drawable/tab_selected_focused" />
<!-- STATES WHEN BUTTON IS PRESSED -->
    <!-- Non focused states -->
    <item android:state_focused="false" android:state_selected="false"
          android:state_pressed="true"
          android:drawable="@drawable/tab_unselected_pressed" />
    <item android:state_focused="false" android:state_selected="true"
        android:state_pressed="true"
        android:drawable="@drawable/tab_selected_pressed" />
    <!-- Focused states (such as when focused with a d-pad or mouse hover) -->
    <item android:state_focused="true" android:state_selected="false"
          android:state_pressed="true"
          android:drawable="@drawable/tab_unselected_pressed" />
    <item android:state_focused="true" android:state_selected="true"
          android:state_pressed="true"
          android:drawable="@drawable/tab_selected_pressed" />
</selector>
```

###为Android 3.0及以上版本定义
当仅仅支持了Android 3.0及更高的版本，你的XML风格文件应该使这样的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@style/Theme.Holo">
        <item name="android:actionBarTabStyle">@style/MyActionBarTabs</item>
    </style>
    <!-- ActionBar tabs styles -->
    <style name="MyActionBarTabs"
           parent="@style/Widget.Holo.ActionBar.TabView">
        <!-- tab indicator -->
        <item name="android:background">@drawable/actionbar_tab_indicator</item>
    </style>
</resources>
```

###为Android 2.1及以上版本定义
当使用了支持库，你的XML风格文件应该使这样的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@style/Theme.AppCompat">
        <item name="android:actionBarTabStyle">@style/MyActionBarTabs</item>
        <!-- Support library compatibility -->
        <item name="actionBarTabStyle">@style/MyActionBarTabs</item>
    </style>
    <!-- ActionBar tabs styles -->
    <style name="MyActionBarTabs"
           parent="@style/Widget.AppCompat.ActionBar.TabView">
        <!-- tab indicator -->
        <item name="android:background">@drawable/actionbar_tab_indicator</item>
        <!-- Support library compatibility -->
        <item name="background">@drawable/actionbar_tab_indicator</item>
    </style>
</resources>
```
> **更多资源**
> 
> - [ActionBar](http://android.xsoftlab.net/guide/topics/ui/actionbar.html#Style)指南列出了更多ActionBar的风格属性。
> - [Styles and Themes](http://android.xsoftlab.net/guide/topics/ui/themes.html)指南可以学习到主题样式的工作原理。
> - 移步[Android Action Bar Style Generator](http://www.actionbarstylegenerator.com/)查看更多的ActionBar完整样式。