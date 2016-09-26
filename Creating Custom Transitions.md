原文地址：[http://android.xsoftlab.net/training/transitions/custom-transitions.html](http://android.xsoftlab.net/training/transitions/custom-transitions.html)

自定义转场可以创建自定义动画。比如，可以定义一种动画来更改文本的颜色或者将输入框的颜色置灰以表示不可用。

自定义转场与内置转场相同，都作用在View之上。不过与内置转场不同的是，还需要另外写一些代码来捕获转场过程的属性值，并生成相关动画。

这节课将会学习如何获取属性值，并生成相关动画。

##继承Transition类
为了创建自定义转场，需要继承Transition，并重写以下方法：
```java
public class CustomTransition extends Transition {
    @Override
    public void captureStartValues(TransitionValues values) {}
    @Override
    public void captureEndValues(TransitionValues values) {}
    @Override
    public Animator createAnimator(ViewGroup sceneRoot,
                                   TransitionValues startValues,
                                   TransitionValues endValues) {}
}
```

下面的部分会学习如何重写这些方法。
##获取View的属性值
转场动画使用了属性动画系统。属性动画通过更改View的属性实现了属性动画，所以转场框架需要使用属性的启动值与结束值来构造动画。

属性动画通常只会用到View的极少属性。比如，颜色动画需要颜色属性值，平移动画需要位置属性值。因为转场动画只需要某些特定的属性值，所以转场框架并没有将所有的属性值提供给转场动画。相反的，转场框架会调用回调方法以便允许转场动画获得需要的属性值，并将其存入框架中。

###获得起始值
为了可以将起始的View值传给转场框架，需要实现[captureStartValues(transitionValues)](http://android.xsoftlab.net/reference/android/transition/Transition.html#captureStartValues(android.transition.TransitionValues))方法。转场框架会在每个View处于启动场景时调用该方法。这个方法的参数是一个TransitionValues对象，这个对象包含了View的引用及一个Map对象，你可以将View的属性值存放在这个Map对象中，然后这些值就会被传给转场框架。

为了确保所存储的属性值的键不会与其它TransitionValues的键相冲突，可以使用以下命名规则：
package_name:transition_name:property_name

下面的代码展示了[captureStartValues()](http://android.xsoftlab.net/reference/android/transition/Transition.html#captureStartValues(android.transition.TransitionValues))方法的实现：
```java
public class CustomTransition extends Transition {
    // Define a key for storing a property value in
    // TransitionValues.values with the syntax
    // package_name:transition_class:property_name to avoid collisions
    private static final String PROPNAME_BACKGROUND =
            "com.example.android.customtransition:CustomTransition:background";
    @Override
    public void captureStartValues(TransitionValues transitionValues) {
        // Call the convenience method captureValues
        captureValues(transitionValues);
    }
    // For the view in transitionValues.view, get the values you
    // want and put them in transitionValues.values
    private void captureValues(TransitionValues transitionValues) {
        // Get a reference to the view
        View view = transitionValues.view;
        // Store its background property in the values map
        transitionValues.values.put(PROPNAME_BACKGROUND, view.getBackground());
    }
    ...
}
```

###获得结束值
转场框架会在每次场景结束时调用[captureEndValues(TransitionValues)](http://android.xsoftlab.net/reference/android/transition/Transition.html#captureEndValues(android.transition.TransitionValues))方法。至于其它方面，该方法内部的实现逻辑与获取开始值的逻辑一致。

下面的代码段展示了[captureEndValues](http://android.xsoftlab.net/reference/android/transition/Transition.html#captureEndValues(android.transition.TransitionValues))方法的实现：
```java
@Override
public void captureEndValues(TransitionValues transitionValues) {
    captureValues(transitionValues);
}
```

在这个示例中，captureStartValues()方法与captureEndValues()方法都会调用captureValues()方法来获取值然后存储这些值。在captureValues()方法中获取View属性都相同，只是启动场景与结束场景获得的属性值不同。转场框架对起始场景与结束场景分别维护了各自的Map实例。

##创建自定义Animator
为了使View在转场的时候可以以动画的方式变动，需要重写[createAnimator()](http://android.xsoftlab.net/reference/android/transition/Transition.html#createAnimator(android.view.ViewGroup,%20android.transition.TransitionValues,%20android.transition.TransitionValues))方法，并返回一个Animator对象。在转场框架调用这个方法时，会将变幻场景的根View与[TransitionValues](http://android.xsoftlab.net/reference/android/transition/TransitionValues.html)对象传回。启动[TransitionValues](http://android.xsoftlab.net/reference/android/transition/TransitionValues.html)对象包含了转场过程中所捕获的属性值。

createAnimator()方法的调用取决于启动场景与结束场景变换的过程。试着将淡入淡出动画想象为自定义转场动画。如果启动场景有5个target，而到了结束场景时被移除了两个，并且还添加了一个新的target，那么转场框架会调用createAnimator()方法6次：其中三个在启动场景与结束场景中都在出现，其中两个在转变到结束场景的过程中被移除了，最后一个在转换到结束场景时被添了进去。

对于在开始场景与结束场景中都存在的target View，转场框架会在调用createAnimator()方法时将TransitionValues对象通过startValues参数与endValues参数回传。对于只存在于单个场景的target View，转场框架会通过对应的参数将TransitionValues对象回传，而另一个参数则为空。

在实现[createAnimator(ViewGroup, TransitionValues, TransitionValues)](http://android.xsoftlab.net/reference/android/transition/Transition.html#createAnimator(android.view.ViewGroup,%20android.transition.TransitionValues,%20android.transition.TransitionValues))方法时，使用所捕获的属性值来创建一个Animator对象，并将其返回给转场框架。对于实现的示例，请参见[CustomTransition](http://android.xsoftlab.net/samples/CustomTransition/index.html)示例中的[ChangeColor](http://android.xsoftlab.net/samples/CustomTransition/src/com.example.android.customtransition/ChangeColor.html)类。有关更多属性动画的相关信息，请参见[Property Animation](http://android.xsoftlab.net/guide/topics/graphics/prop-animation.html)。

##使用自定义转场动画
自定义转场动画与内置转场动画的使用方式相同。同样可以通过转场管理者使用自定义转场动画，具体使用描述请参见[Applying a Transition](http://android.xsoftlab.net/training/transitions/transitions.html#Apply)。