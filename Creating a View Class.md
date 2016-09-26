原文地址：[http://android.xsoftlab.net/training/custom-views/index.html](http://android.xsoftlab.net/training/custom-views/index.html)

#引言
Android框架中拥有大量的View类，这些类用来展示各式各样的数据，并可以直接与用户交互。但是某些时候，APP有一项很特殊的需求，而框架中的View还不能满足这样的需求，这时就需要根据需求自己创建一个全新的View类了。这节课程将会学习如何创建健壮的可复用的View。

#创建View类
一个设计良好的自定义View类与其它任何设计良好的类都很相似。它封装了一系列特殊的功能，并将简单易用的接口暴露了出来。它高效、合理的使用了CPU及内存等资源。除了要上面这些特点之外，自定义View还需要：

- 遵循Android标准。
- 提供可以工作于Android XML布局中的自定义样式属性。
- 发送访问事件。
- Android平台的兼容性。

Android框架提供了一系列的基础类及XML标签来辅助你创建符合以上标准的View。这节课将会学习如何使用Android框架来创建View类的核心功能。

##创建View的子类
Android中的所有View类都继承于[View](http://android.xsoftlab.net/reference/android/view/View.html)。自定义View可以直接继承View，或者通过继承View子类(比如[Button](http://android.xsoftlab.net/reference/android/widget/Button.html))的方式来节省时间。

为了使 [Android Developer Tools](http://android.xsoftlab.net/guide/developing/tools/adt.html) 可以与你的View产生交互，应当至少提供一个含有Context与AttributeSet作为属性的构造方法。这个构造方法可以使布局编辑器创建或编辑View的实例。
```java
class PieChart extends View {
    public PieChart(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

##定义自定义属性
如果要往UI中添加View，你需要以XML元素的形式指定该View，并通过元素属性控制View的行为与外貌。书写良好的自定义View还可以通过XML添加、设计这些样式。为了使自定义View拥有以下这些行为，你必须：

- 在<declare-styleable>资源元素中定义自定义View的属性。
- 在XML布局中指定属性的值。
- 在运行时接收属性值。
- 将接收到的属性值应用到View中。

这部分将会学习如何定义属性、如何指定它们的值。下部分会学习在运行时处理接收并使用这些值。

要定义自定义属性，需要在工程中添加<declare-styleable>资源。通常会将这些资源放在res/values/attrs.xml文件中。下面是一个attrs.xml文件的示例：
```java
<resources>
   <declare-styleable name="PieChart">
       <attr name="showText" format="boolean" />
       <attr name="labelPosition" format="enum">
           <enum name="left" value="0"/>
           <enum name="right" value="1"/>
       </attr>
   </declare-styleable>
</resources>
```

上面这段代码声明了两个自定义属性，showText 和 labelPosition，它们都属于一个名为PieChart的风格实体。风格实体的名字按照惯例与对应的自定义View的类名相一致。尽管不是必须要遵循这项惯例，但很多受欢迎的代码编辑者都依照这项命名惯例来提供实现声明。

一旦定义了自定义属性，你就可以像内置属性那样在XML文件中使用它们。唯一的不同就是，自定义属性属于不同的命名空间。它们不属于标准的http://schemas.android.com/apk/res/android命名空间，而属于http://schemas.android.com/apk/res/[your package name]。举个例子，下面是如何使用自定义属性的示例：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
 <com.example.customviews.charting.PieChart
     custom:showText="true"
     custom:labelPosition="left" />
</LinearLayout>
```

为了避免重复定义命名空间，示例中使用了xmls指令。该指定使http://schemas.android.com/apk/res/com.example.customviews. 命名空间与custom 别名产生了关联。你可以为命名空间使用任何你想使用的名称。

注意，这里在布局中引用的自定义View采用的是自定义View的权限定名。如果View类是个内部类，还必须通过外部类的类名进一步限定。举个例子， PieChart 含有一个名叫PieView的内部类。如果要为这个类使用自定义属性，你应该使用标签：com.example.customviews.charting.PieChart$PieView.

##应用自定义属性
当一个View从XML布局中被创建后，XML标签中的所有属性会被读取，并通过View的构造方法以[AttributeSet](http://android.xsoftlab.net/reference/android/util/AttributeSet.html)的形式传递到View中。尽管它可能是直接从[AttributeSet](http://android.xsoftlab.net/reference/android/util/AttributeSet.html)中读取数据的，但是它还是有一些缺点：

- 资源所引用的属性值不能够被解析
- 不支持风格

相反的，可以将[AttributeSet](http://android.xsoftlab.net/reference/android/util/AttributeSet.html)传给[obtainStyledAttributes()](http://android.xsoftlab.net/reference/android/content/res/Resources.Theme.html#obtainStyledAttributes(android.util.AttributeSet,%20int[],%20int,%20int))方法。该方法会返回一个[TypedArray](http://android.xsoftlab.net/reference/android/content/res/TypedArray.html)对象，它内部包含了被间接引用到的数组值。

Android资源编译器为了使使用obtainStyledAttributes()方法更加简便做了大量的工作。在资源目录中的每一个<declare-styleable>资源都会在R.java中定义相应的属性id。你可以使用这预定义的常量去[TypedArray](http://android.xsoftlab.net/reference/android/content/res/TypedArray.html)中读取属性。下面是PieChart类如何读取属性的示例：
```java
public PieChart(Context context, AttributeSet attrs) {
   super(context, attrs);
   TypedArray a = context.getTheme().obtainStyledAttributes(
        attrs,
        R.styleable.PieChart,
        0, 0);
   try {
       mShowText = a.getBoolean(R.styleable.PieChart_showText, false);
       mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);
   } finally {
       a.recycle();
   }
}
```

注意[TypedArray](http://android.xsoftlab.net/reference/android/content/res/TypedArray.html)是一个共享资源，必须在使用完之后对其进行回收。

##添加属性及事件
Attributes是控制View的外观与行为的一种强大的方式，但是它只有在View初始化的时候才会被读取。如果要提供动态的行为，需要暴露相应的get，set方法。下面的代码展示了PieChart是如何暴露名为showText的属性方法的：
```java
public boolean isShowText() {
   return mShowText;
}
public void setShowText(boolean showText) {
   mShowText = showText;
   invalidate();
   requestLayout();
}
```

注意在setShowText中调用了[invalidate()](http://android.xsoftlab.net/reference/android/view/View.html#invalidate())方法与[requestLayout()](http://android.xsoftlab.net/reference/android/view/View.html#requestLayout())方法。这些方法可以确保View的行为更改生效。在更改完成属性之后不得不重新绘制该View，这样才能使View的外观生效。这样系统才会知道该View需要重新绘制。同样的，如果新属性值可能会引起尺寸或者形状的更改还需要请求新的布局。忘记调用这些方法会引起很难发现的Bug.

自定义View还应当支持事件监听器，以便与重要的事件交互。比如，PieChart暴露一个名为OnCurrentItemChanged的自定义事件，用来通知监听器用户旋转了饼图。

很容易忘记暴露属性和事件，尤其你是唯一一个自定义View的用户。在定义View接口时多花点心思可以在将来维护时减少不必要的开销。一个良好的习惯就是总是暴露任何属性的外观与行为的属性方法。

##可访问性
自定义View应当支持更宽广的用户。这其中包括视力有缺陷的残疾人。为了支持这部分用户的使用，应当：

- 使用android:contentDescription属性标识你的输入字段
- 在适当的时候通过sendAccessibilityEvent()方法发送可访问事件
- 支持更多的控制器，比如D-pad及轨迹球

有关更多信息请参见 [Making Applications Accessible](http://android.xsoftlab.net/guide/topics/ui/accessibility/apps.html#custom-views)。