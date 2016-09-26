原文地址：[http://android.xsoftlab.net/training/transitions/index.html](http://android.xsoftlab.net/training/transitions/index.html)

#引言
Activity所呈现的UI经常会由用户的输入或者其它事件而发生变化。比如，一个含有输入框的Activity，在用户输入要查找的关键字之后，这个输入框就会隐藏，并会在输入框的地方展示出搜索后的结果。

为了可以在这样的情况下呈现连贯的视觉效果，可以在不同View的展示隐藏过程中使用动画。这些的动画可以为用户提供一定的反馈，并会帮助他们学习程序是如何运转的。

Android提供了这种转场框架，它可以使开发者很容易的实现两个View之间动画转换效果。这个框架通过改变View的属性来实现动画效果。这个框架提供了一些常用的动画效果，并且还允许开发者创建自定义的动画效果及动画过程中的生命周期回调。

这节课将会学习如何使用内置的动画效果来作用两个View。这节课还囊括了如何创建自定义动画等知识。

> **Note:** 在Android 4.0之后，Android 4.4.2之前的版本中，使用animateLayoutChanges属性来使动画作用布局。有关更多知识，请参见[Property Animation](http://android.xsoftlab.net/guide/topics/graphics/prop-animation.html)及[Animating Layout Changes](http://android.xsoftlab.net/guide/topics/graphics/prop-animation.html).

#转场框架
动画所提供的不仅仅是视觉上的效果，它更多的作用是突出变化，以便可以提供一些行为来让用户在潜意识中学习程序是如何工作的。我们最常见的例子就是Activity在切换时候的动画，正常的切换动画可以让用户知道是进入了一个页面还是退出了一个页面。

为了帮助开发者可以学习View之间的动画，Android提供了转场框架。这个框架可以在View之间变化时一同作用一个或者一组动画。

转场框架拥有以下特性：

- Group-level animations：可以同时作用一组动画效果。
- Transition-based animation：动画的运行建立在View属性值从开始到结束之间数值变化的基础之上。
- Built-in animations：为常见的效果提供内置的动画，比如淡入、淡出或者平移。
- Resource file support：从布局资源文件中加载View层级及内置的动画。
- Lifecycle callbacks：定义回调为动画作用的过程提供更为精细的控制力。

##概述
转场框架可以作用于任何的View。这个View可以是单个的View对象，也可以是复合型的View容器，比如ViewGroup。转场框架通过改变View的属性来实现动画效果。

转场框架在View层级与动画的两条平行线之间工作。该框架的用途是存储View层级的状态，然后改变这些层级，再通过存储并应用动画定义来实现动画效果。

下图演示了View层级与框架对象和动画之间的关系：

![](http://android.xsoftlab.net/images/transitions/transitions_diagram.png)

转场框架提供了抽象的场景、转换及转换管理者。这些部分会在下面的部分详细介绍。如果要使用该框架，首先为View层级创建场景。接下来对View创建转换效果。为了能启动转场动画，需要使用一个 转换管理者来指明转换动画与结束场景。这个过程会在这节课的剩余课程中详细描述。

##场景
场景保存了View层级的状态，包括所有的View及其属性值。一个View层级可以是一个单纯的View对象，亦或者是一个复合型的ViewGroup对象。将View的状态存储于场景中可以使这些状态从一个场景转换到另一个场景。场景框架提供了[Scene](http://android.xsoftlab.net/reference/android/transition/Scene.html)类来表示一个场景。

转场框架可以从布局资源文件中创建场景或者从ViewGroup对象中创建场景。在代码中创建场景在两个地方会用到：一是动态生成View层级或者在运行时修改场景。

在很多情况下，并不需要专门去创建启动场景。如果已经采用了一种转换，那么转场框架会将上一个结束场景作为下一个转换的启动场景。如果还没有采用任何转换，那么框架会收集屏幕中当前状态下View的相关信息。

场景也可以定义自己的行为，这个行为会在场景改变的时候运行。比如，在场景转变完成之后可以利用这个特性来清理View的设置。除了View层级与其属性值之外，场景还可以存储View层级的父布局的引用。这个根View被称为**scene root**。改变场景与动画会引起scene root中场景的发生。

更多学习创建场景的知识，请参见[Creating a Scene](http://android.xsoftlab.net/training/transitions/scenes.html)。

##转场
在转场框架中，动画会创建一系列帧，这些帧描述了View层级从开始到结束场景过程中的每一项细节变化。动画的有关信息被存在一个名为[Transition](http://android.xsoftlab.net/reference/android/transition/Transition.html)的对象中。如果要运行动画，则需要使用[TransitionManager](http://android.xsoftlab.net/reference/android/transition/TransitionManager.html)对象。转场框架会在两个不同的场景中转换或在当前的场景中转换不同的状态。

转场框架包含了一系列内置转场，这主要被用于通用动画效果。比如淡入淡出、调整View尺寸。你也可以使用动画框架所提供的API来定义自定义场景来创建独有的动画效果。转场框架还可以使开发者整合不同的动画效果到一个集合中，这个集合可以包含内置的动画效果或者自定义的动画效果。

转场的生命周期与Activity的生命周期极为类似，这代表了动画执行过程中的每一个转换状态。在重要的生命周期状态下，转场框架会调用这些回调方法，这可以使开发者在转场的过程中适时调整用户界面。

有关更多转场的相关知识，请参见[Applying a Transition](http://android.xsoftlab.net/training/transitions/transitions.html)及[Creating Custom Transitions](http://android.xsoftlab.net/training/transitions/custom-transitions.html)。

##转场的局限性
这部分列出了一些转场框架已知的不足：

- 动画作用到SurfaceView上可能不会正常显示。因为SurfaceView对象由非UI线程更新，所以这个更新可能不会与其它View的动画保持一致。
- 当在TextureView上使用动画时，可能某些特殊的转场类型不会产生预想中效果。
- 继承于AdapterView的类，比如ListView，它们管理子View的方式与转场框架互不兼容。如果视图作用动画于AdapterView等之上，那么设备界面可能会假死。
- 如果想使调整尺寸动画作用于TextView上，那么TextView上的文本会在动画完成之前被绘制到一个新的位置。为了避免这个问题，请不要将调整尺寸的动画作用在包含文本的View上。