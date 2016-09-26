原文地址 : [http://android.xsoftlab.net/training/basics/supporting-devices/screens.html#create-bitmaps](http://android.xsoftlab.net/training/basics/supporting-devices/screens.html#create-bitmaps)

Android设备屏幕分为两个通用的属性：尺寸和密度。你应该期待应用将会被安装在屏幕的密度和尺寸都在范围内的设备上。正因为这样，你应该包含一些可替换的资源，以便应用在不同尺寸的屏幕和不同密度的屏幕效果最优。

 - 有4种普遍屏幕尺寸：small, normal, large, xlarge。、
 - 还有4种普遍的屏幕密度：low (ldpi), medium (mdpi), high (hdpi), extra high (xhdpi)。

为了对不同的屏幕声明使用不同的布局和图像，你必须将这些备选资源分开放置，和不同的语言字符串很类似。

这里也应该意识到要考虑屏幕的方向，所以很多应用应该通过布局为不同的方向提供良好的用户体验。

##创建不同的布局
为了在不同尺寸的屏幕上提升用户体验，你应该为想要支持的屏幕尺寸创建唯一的XML布局文件。
每一个布局文件应该保持在合适的资源目录下，以-< screen_size>为后缀，唯一的大屏幕布局应该被保存在目录res/layout-large下。

> **Note:** Android为了适配屏幕会拉伸你的布局。所以，不需要关心每一种尺寸的布局元素的绝对尺寸，而应该关心布局之间的结构关系，否则会影响用户体验。

下面这个工程为大屏幕提供了一个合适的布局。
```
MyProject/
    res/
        layout/
            main.xml
        layout-large/
            main.xml
```

文件名称要尽可能的准确，但是其中的内容为了不同尺寸的屏幕可以不一样。
一般在代码中简单的引用下布局文件：
```java
@Override
 protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.main);
}
```

系统会在应用运行的时候基于设备的屏幕尺寸加载合适的布局文件。更多关于Android如何选择合适的可用资源请参见：[Providing Resources](http://android.xsoftlab.net/guide/topics/resources/providing-resources.html#BestMatch)。

像其它工程一样，这个工程为水平方向提供了适当的布局：
```
MyProject/
    res/
        layout/
            main.xml
        layout-land/
            main.xml
```

默认情况下，layout/main.xml被用作于默认方向。

如果需要为大屏幕的水平方向屏幕提供布局，那么你需要同时使用large和land标识符：
```
MyProject/
    res/
        layout/              # default (portrait)
            main.xml
        layout-land/         # landscape
            main.xml
        layout-large/        # large (portrait)
            main.xml
        layout-large-land/   # large landscape
            main.xml
```

> **Note:** Android 3.2及更高的版本对于支持规定的屏幕尺寸有更为先进的方法。它允许你为一定范围内的屏幕尺寸提供资源，一定范围包括自小的宽度，高度和密度。这节课不覆盖这些新知识点，有关更多信息，请参见：[Designing for Multiple Screens](http://android.xsoftlab.net/training/multiscreen/index.html)。

##创建不同的位图
你应该提供合适的位图资源给每个通用的密度区域:low, medium, high and extra-high density,这可以帮助你在所有的密度下达到良好的图像效果和性能。

为了产生这些图像，你应该根据矢量格式的真实资源来为每一种密度提供扩展尺寸：

    xhdpi: 2.0
    hdpi: 1.5
    mdpi: 1.0 (baseline)
    ldpi: 0.75

这里的意思是说，如果你为xhdpi的设备生成了一张200x200的图像，那么你应该为hdpi生成150x150的图像，以此推类。

然后，将这些文件放入到合适的图像资源目录下：
```
MyProject/
    res/
        drawable-xhdpi/
            awesomeimage.png
        drawable-hdpi/
            awesomeimage.png
        drawable-mdpi/
            awesomeimage.png
        drawable-ldpi/
            awesomeimage.png
```

任何时候通过@drawable/awesomeimage引用图像的时候，系统会根据密度选择合适的位图图像。
> **Note:** 低密度ldpi并不总是必须的。当你提供了hdpi的资源，系统会将hdpi资源缩小一半来适应ldpi的屏幕。

更多有关为APP创建icon资源的提示和指南，请参见：[Iconography design guide](http://android.xsoftlab.net/design/style/iconography.html).