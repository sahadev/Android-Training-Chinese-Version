原文地址：[http://android.xsoftlab.net/training/custom-views/making-interactive.html](http://android.xsoftlab.net/training/custom-views/making-interactive.html)

绘制UI只是自定义View的一部分而已。你还需要使你的View可以响应用户的输入事件。
##处理输入手势
与其它的UI框架很相似，Android同样支持输入事件模型。用户的行为会被转换为一种事件并会触发回调，你可以通过重写这些回调方法来决定如何对这些事件做出响应。Android中最为常见的输入事件是touch，它会触发[onTouchEvent(android.view.MotionEvent)](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent%28android.view.MotionEvent%29)。通过重写这个方法来处理一些事件：
```java
   @Override
   public boolean onTouchEvent(MotionEvent event) {
    return super.onTouchEvent(event);
   }
```
触摸事件本身并不是特别有用。现代触摸UI定义了交互，比如下拉、上推、飞速滑动以及缩放。将原始触摸事件转换为手势，Android提供了[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)。

构造[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)需要传递一个[GestureDetector.OnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html)的实现类。如果你只是需要处理几个手势，你可以继承[GestureDetector.SimpleOnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html)。下面的代码继承了这个接口，并重写了其[onDown(MotionEvent)](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html#onDown%28android.view.MotionEvent%29)方法。
```java
class mListener extends GestureDetector.SimpleOnGestureListener {
   @Override
   public boolean onDown(MotionEvent e) {
       return true;
   }
}
mDetector = new GestureDetector(PieChart.this.getContext(), new mListener());
```
不管你是否使用了GestureDetector.SimpleOnGestureListener接口，你都需要重写[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown%28android.view.MotionEvent%29)方法，并需要返回true。这一步是必须的，因为所有的手势都是从[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown%28android.view.MotionEvent%29)方法开始的。如果你在[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown%28android.view.MotionEvent%29)方法中返回了false，那么系统会认为你想忽略这次手势事件，并且其它的方法都不会被调用。如果你真心想要忽略整个手势事件，那么在onDown()方法中返回false是唯一的一次机会。一旦你实现了GestureDetector.OnGestureListener接口，并创建了一个GestureDetector的实例，你可以使用你的GestureDetector对象来与在[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/GestureDetector.html#onTouchEvent%28android.view.MotionEvent%29)中接收到的触摸事件进行交互。
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
##创建物理模拟事件
手势是控制触摸屏设备的一种非常强大的方式，但是它们可能是违反直觉的，并且不容易记住，除非它们提供了物理模拟结果。一种好示例就是飞速滑动手势，当用户快速的在屏幕上滑动手指，然后离开屏幕时就会触发这种手势。这种手势造成了一种感觉，如果UI在同一方向上快速滑动然后慢慢的减速，这就仿佛用户在操作一个飞轮一样。

无论如何，这都不是最重要的。如果需要工作的正确，这就需要很多的物理及数学运算。幸运的是，Android为此提供了辅助类。类[Scroller](http://android.xsoftlab.net/reference/android/widget/Scroller.html)是一个用来处理这种飞轮风格的辅助类。

要开始滑动，需要以一个初始速度调用[fling()](http://android.xsoftlab.net/reference/android/widget/Scroller.html#fling%28int,%20int,%20int,%20int,%20int,%20int,%20int,%20int%29)，有关速度值，你可以使用GestureDetector产生。
```java
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
   mScroller.fling(currentX, currentY, velocityX / SCALE, velocityY / SCALE, minX, minY, maxX, maxY);
   postInvalidate();
}
```
> **Note:**

调用fling()为滑动手势设置物理模型。你需要更新Scroller，通过Scroller.computeScrollOffset()。Scroller.computeScrollOffset()会更新Scroller对象的内部状态，通过读取当前的时间以及使用物理模型来计算x及y的位置。调用getCurrX()和getCurrY()来接收这些值。

很多View直接将Scroller对象的x及y的位置传递给scrollTo()。饼图示例有些小小的不同：它使用了当前滑动的y的位置来设置饼图的旋转角度：
```java
if (!mScroller.isFinished()) {
    mScroller.computeScrollOffset();
    setPieRotation(mScroller.getCurrY());
}
```
Scroller会为你计算滑动的位置，但是它不会自动的将这些值应用到你的View中。将这些值获得然后应用这是你需要做的。有两种方式可以做到：

- 在fling()之后调用postInvalidate()，为了可以重新绘制界面。这项技术需要你每次滑动偏移量发生变化之后在onDraw()方法中调用。
- 为滑动的事件设置ValueAnimator，并通过调用addUpdateListener()添加监听器来处理动画的更新。

在饼图示例中使用了第二种方案。这项技术在设置上稍微的有些复杂，但是它的工作过程更加严密，并且不会请求不必要的更新。它的缺点是在API 11之前ValueAnimator并不适用，所以这项在Android 3.0之前不可以使用。

> **Note:** ValueAnimator在API 11之前并不可用，但是你仍然还可以在API 11之前可用。你只需要确保在运行时检查了当前的API等级，并且在当前的等级低于11时不调用View动画。

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

##使过程更加流畅
用户会期待状态之间的转变过渡非常流畅。UI元素的淡入淡出取代了闪现与消失。动作的平滑过渡取代了突然的出现与消失。Android 3.0中出现的 [property animation framework](http://android.xsoftlab.net/guide/topics/graphics/prop-animation.html) 使平滑转场更加轻松。

为了使用动画系统，属性改变时会影响到View的外观，不要直接更改属性。相反的，使用ValueAnimator来做出改变。在下面的示例中，修改当前所选择的扇形图会使整个的表格发生旋转，所以选择的点是居中的。ValueAnimator更改旋转用了数以百毫秒的事件，宁愿突然设置一个新的旋转值。
```java
mAutoCenterAnimator = ObjectAnimator.ofInt(PieChart.this, "PieRotation", 0);
mAutoCenterAnimator.setIntValues(targetAngle);
mAutoCenterAnimator.setDuration(AUTOCENTER_ANIM_DURATION);
mAutoCenterAnimator.start();
```
如果你想更改的值是View的基础属性，那么做这项事情就更容易了，因为View有一个内置的View属性动画框架ViewPropertyAnimator，它专门用来同时作用多个属性，比如：
```java
animate().rotation(targetAngle).setDuration(ANIM_DURATION).start();
```

