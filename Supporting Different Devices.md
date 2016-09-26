原文地址 :　[http://android.xsoftlab.net/training/basics/supporting-devices/index.html](http://android.xsoftlab.net/training/basics/supporting-devices/index.html)

##支持不同的设备
在世界上，Android设备以很多种形状和尺寸呈现。正因为有这么多种设备，你的应用有机会被巨多的用户所使用。为了在Android上尽可能的成功，你的应用需要适配花样繁多的设备配置。一些重要的点就是你应该考虑包含不同的语言、适配各种屏幕尺寸密度、各种各样的Andorid平台版本。

这节课将会教你如何利用可替代资源和其它特性等最基本的平台特性，这样你的APP才可以只用一个APK就可以在花样繁多的Android设备上提供极佳的用户体验。

#支持不同的语言
这对于在APP的代码中使用R.string.xx这种额外的字符串资源并将它们放入一个额外的文件中来说是最好的练习。Android使得每一个Android工程管理这种资源很轻松。

*如果你使用的是Android SDK Tools创建的工程，那么在工程的res/目录下有很多种类的资源类型。这里有一些默认的文件比如res/values/string.xml便是存放字符串资源的地方。*

##创建本地语言目录和字符串文件
为了支持更多的语言，在res/目录下创建一个包含values，连接符，国际标准化组织语言编码作为名称的目录，举个例子，values-ex/是一个包含了以es编码的本地语言的简单资源目录。Android会在设备运行的时候读取本地语言设置从而加载合适的本地语言。更多信息请参见：[Providing Alternative Resources](http://android.xsoftlab.net/guide/topics/resources/providing-resources.html#AlternativeResources).

如果决定了将要支持哪种语言，只需要创建一个资源目录和一个字符串资源文件：
```
MyProject/
    res/
       values/
           strings.xml
       values-es/
           strings.xml
       values-fr/
           strings.xml
```
在合适的文件中为每一个本地语言添加字符串值。

在运行时，Android系统会基于用户的设备设置来选择适合的字符串资源。

下面是一些不同语言所对应的字符串资源文件：
English (default locale), /values/strings.xml:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">My Application</string>
    <string name="hello_world">Hello World!</string>
</resources>
```
Spanish, /values-es/strings.xml:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">Mi Aplicación</string>
    <string name="hello_world">Hola Mundo!</string>
</resources>
```

French, /values-fr/strings.xml:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">Mon Application</string>
    <string name="hello_world">Bonjour le monde !</string>
</resources>
```

>**Note:**你可以对任何资源类型使用这种本地限定词，比如如果你只想将位图图像提供给局部的版本就可以这么做。更多信息请参见：[Localization](http://android.xsoftlab.net/guide/topics/resources/localization.html).

##使用字符串资源
你可以在源码或者XML文件中引用这些字符串资源，字符串资源名称通过在文件中定义的< string>元素的name属性定义。

在代码中，可以通过语句R.string.< string_name>引用字符串资源。下面是可接受字符串资源的一些方法：
```java
// Get a string resource from your app's Resources
String hello = getResources().getString(R.string.hello_world);
// Or supply a string resource to a method that requires a string
TextView textView = new TextView(this);
textView.setText(R.string.hello_world);
```

在其它的XML文件中，可以通过语句@string/< string_name>引用字符串资源，下面是如何在XML使用字符串值：
```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />
```