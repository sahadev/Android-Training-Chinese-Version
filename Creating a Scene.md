原文地址：[http://android.xsoftlab.net/training/transitions/scenes.html](http://android.xsoftlab.net/training/transitions/scenes.html)

场景存储了View层级的状态，包含所有的View及View的属性。转场框架在启动场景与结束场景之间运行动画。启动场景通常由当前的UI状态自动决定。对于结束场景，转场框架提供了两种实现方式：从布局资源文件中创建场景或从代码中创建场景。

这节课主要学习如何创建场景及如何定义场景行为。下节课则主要学习如何在两个场景之间转换。

> **Note：** 转场框架可以不使用场景来使动画作用单个View层级，就像[Apply a Transition Without Scenes](http://android.xsoftlab.net/training/transitions/transitions.html#NoScenes)中描述的。无论如何，了解这节课有助于懂得转换的基本工作原理。

##由布局资源创建场景
开发者可以直接从布局资源文件中创建场景实例。当View层级几乎是静止状态时可以使用这项技术。创建好的场景代表了View层级的状态。一旦View层级发生变化，则需要重新床架场景。转场框架会由资源文件中的整个View层级创建场景，因此不能由资源文件的部分层级创建场景。

为了可以从布局资源文件中创建场景，则需要从布局中接收场景容器，一般是一个ViewGroup实例，然后再调用[Scene.getSceneForLayout()](http://android.xsoftlab.net/reference/android/transition/Scene.html#getSceneForLayout(android.view.ViewGroup,%20int,%20android.content.Context))方法，这个方法需要传入场景容器以及包含场景布局资源文件的ID。

###为场景定义布局
下面的代码段展示了如何为一个场景容器元素创建两个不同的场景。代码段还展示了开发者可以加载多个不相关的场景，不过这并不意味着每个场景之间不无关系。

示例结构由以下布局定义构成：

- 主布局包含一个文本控件和一个容器控件。
- 第一个场景的相关布局包含两个文本控件。
- 第二个场景的相关布局同样包含两个文本控件，但是两个控件的顺序是颠倒的。

示例被设计为在Activity的主布局的子布局之间进行动画。主布局的文本控件则会保持静止。

Activity的主布局定义如下：

res/layout/activity_main.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/master_layout">
    <TextView
        android:id="@+id/title"
        ...
        android:text="Title"/>
    <FrameLayout
        android:id="@+id/scene_root">
        <include layout="@layout/a_scene" />
    </FrameLayout>
</LinearLayout>
```

这个布局定义包含了一个文本控件及场景容器的子布局控件。第一个场景的布局被包含在主布局之内。这意味着第一个场景布局会被作为初始化UI的一部分，还可以被加载到一个场景中，因为转场框架只能加载一整个布局文件。

第一个场景的布局文件如下：

res/layout/a_scene.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view1
        android:text="Text Line 1" />
    <TextView
        android:id="@+id/text_view2
        android:text="Text Line 2" />
</RelativeLayout>
```

第二个场景同样包含了两个文本控件，只是它们的顺序发生了颠倒，该布局定义如下：

res/layout/another_scene.xml
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view2
        android:text="Text Line 2" />
    <TextView
        android:id="@+id/text_view1
        android:text="Text Line 1" />
</RelativeLayout>
```

###从布局中生成场景
在定义了两个场景布局文件之后，则可以开始操作它们了。这可以使开发者在两个UI配置之间延迟转场。为了可以操作一个场景，则需要先获得场景容器的引用及布局资源的ID。

下面的代码段展示了如何获得场景容器的引用及从布局文件中创建两个[Scene](http://android.xsoftlab.net/reference/android/transition/Scene.html)对象：
```java
Scene mAScene;
Scene mAnotherScene;
// Create the scene root for the scenes in this app
mSceneRoot = (ViewGroup) findViewById(R.id.scene_root);
// Create the scenes
mAScene = Scene.getSceneForLayout(mSceneRoot, R.layout.a_scene, this);
mAnotherScene =
    Scene.getSceneForLayout(mSceneRoot, R.layout.another_scene, this);
```

现在在应用中有了两个Scene对象。每个Scene都会使用到场景容器。

##在代码中创建场景
开发者还可以在代码中创建Scene对象。当开发者需要直接修改View层级或者动态生成View层级就可以使用这项技术。

为了可以在代码中创建场景。需要使用[Scene(sceneRoot, viewHierarchy)](http://android.xsoftlab.net/reference/android/transition/Scene.html#Scene(android.view.ViewGroup,%20android.view.View))构造方法。调用这个构造方法等同于调用[Scene.getSceneForLayout()](http://android.xsoftlab.net/reference/android/transition/Scene.html#getSceneForLayout(android.view.ViewGroup,%20int,%20android.content.Context))方法。只是该构造方法需要预先加载布局文件。

下面的代码段演示了如何在代码中由场景容器元素及场景的View层级创建一个Scene实例：
```java
Scene mScene;
// Obtain the scene root element
mSceneRoot = (ViewGroup) mSomeLayoutElement;
// Obtain the view hierarchy to add as a child of
// the scene root when this scene is entered
mViewHierarchy = (ViewGroup) someOtherLayoutElement;
// Create a scene
mScene = new Scene(mSceneRoot, mViewHierarchy);
```

##创建场景行为
转场框架还可以使开发者定义转场开始或者结束的行为。在很多情况下，自定义转场行为并不是必须的，因为转场框架会在场景之间自动改变动画。

转场行为有助于处理以下情况：

- 作用动画的View处于不同的层级。开发者可以在场景启动及结束的时候使用退出或者进入场景的行为。
- 转场框架不能够自动的作用View的动画，比如ListView，更多相关信息，请参见[Limitations](http://android.xsoftlab.net/training/transitions/overview.html#Limitations).

如果要定义自定义行为，需要将行为作为[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象传入到[Scene.setExitAction()](http://android.xsoftlab.net/reference/android/transition/Scene.html#setExitAction(java.lang.Runnable))方法或[Scene.setEnterAction()](http://android.xsoftlab.net/reference/android/transition/Scene.html#setEnterAction(java.lang.Runnable))方法。转场框架会在运行转场动画之前调用[Scene.setExitAction()](http://android.xsoftlab.net/reference/android/transition/Scene.html#setExitAction(java.lang.Runnable))方法，会在转场动画结束之后调用[Scene.setEnterAction()](http://android.xsoftlab.net/reference/android/transition/Scene.html#setEnterAction(java.lang.Runnable))方法。

> **Note:** 不要使用场景行为在启动场景与结束场景的View之间传递数据。更多相关信息，请参见[Defining Transition Lifecycle Callbacks](http://android.xsoftlab.net/training/transitions/transitions.html#Callbacks).