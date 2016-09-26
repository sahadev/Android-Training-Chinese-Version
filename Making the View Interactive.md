写在前面的话:**这一章很有价值，想要提升安卓知识的一定要读一读。不做安卓的也可以得到其它方面的提升。**

原文地址：[http://android.xsoftlab.net/training/custom-views/making-interactive.html](http://android.xsoftlab.net/training/custom-views/making-interactive.html)

UI的绘制只是自定义View的一部分。你还需要使View可以以一种接近真实世界的反馈方式来响应用户的输入事件。虚拟世界中的对象应该总是以真实世界中对象的行为方式来行动。比如说，图像不应该从某处突然出现或消失，因为真实世界中的图像总是从一个地方移动到另一个地方的。

用户还应该在UI界面上感知到一些细微的感觉。最好的反馈就是模仿真实世界的微妙行为。举个栗子，用户在快速滑动UI对象时，应该在开始时感觉到延迟的摩擦力，在滑出去后还应当继续保持惯性滑动。

这节课将会演示如何使用Android的框架特性为自定义View添加这些真实世界的行为。

##处理输入手势
与其它UI框架很接近，Android同样支持输入事件模型。用户的行为会被转换为一种会触发回调的事件，你可以通过重写这些回调方法来决定如何对这些事件做出响应。Android中最为常见的输入事件是touch，它会触发[onTouchEvent(android.view.MotionEvent)](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent%28android.view.MotionEvent%29)。通过重写这个方法来处理一些事件：
```java
   @Override
   public boolean onTouchEvent(MotionEvent event) {
    return super.onTouchEvent(event);
   }
```

触摸事件本身并不是特别有用处。触摸UI定义了一些交互手势，比如双击、下拉、上推、快速滑动以及缩放等等。为了将原始触摸事件转换为手势，Android提供了[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)。

构造[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)需要传递一个[GestureDetector.OnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html)的实现类作为参数。如果你只是需要处理几个手势，你可以继承[GestureDetector.SimpleOnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html)。下面的代码继承了这个接口，并重写了它的[onDown(MotionEvent)](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html#onDown%28android.view.MotionEvent%29)方法。
```java
class mListener extends GestureDetector.SimpleOnGestureListener {
   @Override
   public boolean onDown(MotionEvent e) {
       return true;
   }
}
mDetector = new GestureDetector(PieChart.this.getContext(), new mListener());
```

无论你是否使用GestureDetector.SimpleOnGestureListener接口，你都需要实现一个返回true的[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown%28android.view.MotionEvent%29)方法。这一步是必须的，因为所有的手势都是从[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown%28android.view.MotionEvent%29)方法开始的。如果你在[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown%28android.view.MotionEvent%29)方法中返回了false，那么系统会认为你想忽略这次事件，并且其它的相关方法都不会被调用。如果你真的想要忽略整个手势事件，那么在[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent))方法中返回false是唯一的一种方式。一旦实现了[GestureDetector.OnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html)接口，并创建了[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)的实例，则可以使用[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)对象来与在[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/GestureDetector.html#onTouchEvent%28android.view.MotionEvent%29)中接收到的触摸事件进行交互。
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
   boolean result = mDetector.onTouchEvent(event);
   if (!result) {
       if (event.getAction() == MotionEvent.ACTION_UP) {
           stopScrolling();
           result = true;
       }
   }
   return result;
}
```

当传给[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/GestureDetector.html#onTouchEvent(android.view.MotionEvent))方法一个不能识别的手势时，它会返回false，这样你就可以运行自定义的手势识别代码了。

##创建物理模拟手势
手势是用来控制触摸屏设备的一种强大方式，除非它们提供了物理模拟效果，否则它们可能是违反直觉的、难以记住的。一个好的示例就是飞速滑动手势：当用户快速的在屏幕上滑动手指时，手指突然离开了屏幕，就会触发这种手势。如果UI在同一方向上继续滑动然后慢慢的减速，这时就会给用户造成一种感觉：仿佛在操作一个飞轮一样。

不管怎样，模拟飞轮这种感觉并不是没有价值的。为了正确模拟这种感觉，需要很多的物理及数学运算。幸运的是，Android为此提供了辅助类。[Scroller](http://android.xsoftlab.net/reference/android/widget/Scroller.html)是一个专门用来处理这种飞轮感觉手势的辅助类。

为了启动滑动，需要以一个初始速度值及其它相关速度参数调用[fling()](http://android.xsoftlab.net/reference/android/widget/Scroller.html#fling%28int,%20int,%20int,%20int,%20int,%20int,%20int,%20int%29)。有关速度值，你可以通过[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)计算得到。
```java
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
   mScroller.fling(currentX, currentY, velocityX / SCALE, velocityY / SCALE, minX, minY, maxX, maxY);
   postInvalidate();
}
```

> **Note:** 尽管[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)计算到的这个速度值在物理上很精确，但是很多开发者感觉使用这个值来启动滑动还是太快了。通常会在4到8之间取一个系数来减小x和y。

先调用[fling()](http://android.xsoftlab.net/reference/android/widget/Scroller.html#fling(int,%20int,%20int,%20int,%20int,%20int,%20int,%20int))为滑动手势设置物理模型。然后你需要定期调用[Scroller.computeScrollOffset()](http://android.xsoftlab.net/reference/android/widget/Scroller.html#computeScrollOffset())来更新[Scroller](http://android.xsoftlab.net/reference/android/widget/Scroller.html)。[computeScrollOffset()](http://android.xsoftlab.net/reference/android/widget/Scroller.html#computeScrollOffset())通过读取当前的时间以及使用物理模型来计算x及y的位置来更新Scroller对象的内部状态。通过调用[getCurrX()](http://android.xsoftlab.net/reference/android/widget/Scroller.html#getCurrX())和[getCurrY()](http://android.xsoftlab.net/reference/android/widget/Scroller.html#getCurrY())来接收这些值。

很多View将Scroller对象的x,y的位置值直接传递给[scrollTo()](http://android.xsoftlab.net/reference/android/view/View.html#scrollTo(int,%20int))。饼图示例在这里有些小小的不同：它使用当前滑动的y的位置来设置饼图的旋转角度：
```java
if (!mScroller.isFinished()) {
    mScroller.computeScrollOffset();
    setPieRotation(mScroller.getCurrY());
}
```

Scroller会为你计算滑动的位置，但是它不会自动的将这些值应用到你的View中。为了使View的滑动效果更佳平滑，获得并应用这些值是需要你去做的。有两种方式可以实现：

- 在[fling()](http://android.xsoftlab.net/reference/android/widget/Scroller.html#fling(int,%20int,%20int,%20int,%20int,%20int,%20int,%20int))之后调用[postInvalidate()](http://android.xsoftlab.net/reference/android/view/View.html#postInvalidate())，这样可以重新绘制界面。这个方法需要每次滑动的偏移量发生变化之后在[onDraw()](http://android.xsoftlab.net/reference/android/view/View.html#onDraw(android.graphics.Canvas))方法中调用。
- 为滑动动画设置[ValueAnimator](http://android.xsoftlab.net/reference/android/animation/ValueAnimator.html)，调用[addUpdateListener()](http://android.xsoftlab.net/reference/android/animation/ValueAnimator.html#addUpdateListener(android.animation.ValueAnimator.AnimatorUpdateListener))添加监听器，以便处理动画的更新。

在饼图示例中使用了第二种方案。这项技术在设置上稍微的有些复杂，但是它的工作过程与动画系统更为接近，并且不会请求不必要的更新。它的缺点是在API 11之前ValueAnimator并不适用，所以这项技术在Android 3.0之前不可以使用。

> **Note:** ValueAnimator在API 11之前并不可用，但是你仍然还可以在API 11之前使用。你只需要确保在运行时检查当前的API等级，并且在等级低于11时不调用View动画就可以。

```java
       mScroller = new Scroller(getContext(), null, true);
       mScrollAnimator = ValueAnimator.ofFloat(0,1);
       mScrollAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
           @Override
           public void onAnimationUpdate(ValueAnimator valueAnimator) {
               if (!mScroller.isFinished()) {
                   mScroller.computeScrollOffset();
                   setPieRotation(mScroller.getCurrY());
               } else {
                   mScrollAnimator.cancel();
                   onScrollFinished();
               }
           }
       });
```

##使滑动过程更流畅
用户不希望状态之间的过渡发生卡顿。所以UI元素的淡入淡出取代了闪现与消失。动作的平滑过渡取代了突然的启动与停止。Android 3.0中出现的[property animation framework](http://android.xsoftlab.net/guide/topics/graphics/prop-animation.html)使平滑转场更加简便。

每次属性的变更都会影响View的外观，所以不要直接更改它们的属性。相反，可以使用ValueAnimator来做出变更。在下面的示例中，修改当前所选择的扇形图会使整个饼图发生旋转，所以选择的这个点在饼图中看起来是居正中的。ValueAnimator更改旋转用了数百毫秒的时间。
```java
mAutoCenterAnimator = ObjectAnimator.ofInt(PieChart.this, "PieRotation", 0);
mAutoCenterAnimator.setIntValues(targetAngle);
mAutoCenterAnimator.setDuration(AUTOCENTER_ANIM_DURATION);
mAutoCenterAnimator.start();
```

如果你想更改View的基础属性，那么这项事情就更容易了，因为View有一个内置的View属性动画框架[ViewPropertyAnimator](http://android.xsoftlab.net/reference/android/view/ViewPropertyAnimator.html)，它专门用来同时作用多个属性，比如：
```java
animate().rotation(targetAngle).setDuration(ANIM_DURATION).start();
```

