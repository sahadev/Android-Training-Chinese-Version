原文地址：[http://android.xsoftlab.net/training/gestures/movement.html](http://android.xsoftlab.net/training/gestures/movement.html)

这节课将会学习如何在触摸事件中追踪轨迹。

每当当前的触摸与位置、压力或者尺寸的调整有关系时，一个ACTION_MOVE的事件就被触发了。

因为基于手指的触摸并不总是最精确的交互方式，检查触摸手势与简单的交互相比经常会基于更多的轨迹行为。为了辅助APP区别基于轨迹的手势(比如滑动)与非轨迹手势(比如单点)，Android包含了一个名为"touch slop"的概念。Touch slop引用了用户按下的以像素为单位的距离。

这里有若干项不同的追踪手势轨迹的方法，取决于应用程序的需求：

- 指针的起始位置与结束位置。
- 指针位移的方向，由X，Y决定。
- 历史，你可以找到手势的历史尺寸，通过调用MotionEvent的方法[getHistorySize()](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#getHistorySize())。你接下来可以操作这些位置，尺寸，事件以及每一项历史记录的压力，通过使用MotionEvent的getHistorical(Value)方法。历史是非常有用的，当渲染手指的轨迹时，比如触摸的绘制。
- 指针在屏幕上滑动的速度。

##轨迹的速度
你可以拥有一个基于轨迹的手势，这个手势基于指针移动的距离以及方向。但是速度经常是一个检查要素，在追踪手势的特性或者检查何种手势事件发生时。为了使速度计算更加方便，Android提供了[VelocityTracker](http://android.xsoftlab.net/reference/android/view/VelocityTracker.html)类以及[VelocityTrackerCompat](http://android.xsoftlab.net/reference/android/support/v4/view/VelocityTrackerCompat.html)类在支持库中。[VelocityTracker](http://android.xsoftlab.net/reference/android/view/VelocityTracker.html)用于辅助追踪触摸事件的速度。这对于判断哪个速度是手势的标准部分，比如飞速滑动。

下面是一个简单的例子，用于演示在[VelocityTracker](http://android.xsoftlab.net/reference/android/view/VelocityTracker.html) API中方法的目的：
```java
public class MainActivity extends Activity {
    private static final String DEBUG_TAG = "Velocity";
        ...
    private VelocityTracker mVelocityTracker = null;
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int index = event.getActionIndex();
        int action = event.getActionMasked();
        int pointerId = event.getPointerId(index);
        switch(action) {
            case MotionEvent.ACTION_DOWN:
                if(mVelocityTracker == null) {
                    // Retrieve a new VelocityTracker object to watch the velocity of a motion.
                    mVelocityTracker = VelocityTracker.obtain();
                }
                else {
                    // Reset the velocity tracker back to its initial state.
                    mVelocityTracker.clear();
                }
                // Add a user's movement to the tracker.
                mVelocityTracker.addMovement(event);
                break;
            case MotionEvent.ACTION_MOVE:
                mVelocityTracker.addMovement(event);
                // When you want to determine the velocity, call 
                // computeCurrentVelocity(). Then call getXVelocity() 
                // and getYVelocity() to retrieve the velocity for each pointer ID. 
                mVelocityTracker.computeCurrentVelocity(1000);
                // Log velocity of pixels per second
                // Best practice to use VelocityTrackerCompat where possible.
                Log.d("", "X velocity: " + 
                        VelocityTrackerCompat.getXVelocity(mVelocityTracker, 
                        pointerId));
                Log.d("", "Y velocity: " + 
                        VelocityTrackerCompat.getYVelocity(mVelocityTracker,
                        pointerId));
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                // Return a VelocityTracker object back to be re-used by others.
                mVelocityTracker.recycle();
                break;
        }
        return true;
    }
```

> **Note:** 注意，应当在[ACTION_MOVE](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_MOVE)事件之后计算速度，不要在[ACTION_UP](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_UP)之后计算，在[ACTION_UP](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_UP)之后计算，所得到的X与Y的速度将会是0.

