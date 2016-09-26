原文地址：[https://developer.android.com/training/gestures/viewgroup.html](https://developer.android.com/training/gestures/viewgroup.html)

在ViewGroup中处理触摸事件要格外小心，因为在ViewGroup中有很多子View，而这些子View对于不同的触摸事件来说是不同的目标。要确保每个View都正确的接收了相应的触摸事件。

##在ViewGroup中拦截触摸事件
[onInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))方法会在触摸事件到达[ViewGroup](https://developer.android.com/reference/android/view/ViewGroup.html)的表面时调用，这包括内部的子View。如果[onInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))返回了true，那么[MotionEvent](https://developer.android.com/reference/android/view/MotionEvent.html)对象就会被拦截，这意味着该次事件不会传给子View，而是会传给ViewGroup本身的[onTouchEvent()](https://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法。

[onInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))给了ViewGroup本身一个机会：在子View获得任何事件之前一个拦截该事件的机会。如果[onInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))返回了true，那么原先处理该次事件的子View就会收到一个[ACTION_CANCEL](https://developer.android.com/reference/android/view/MotionEvent.html#ACTION_CANCEL)的事件，并且原先事件的剩余事件都会被传到该ViewGroup的[onTouchEvent()](https://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法中做常规处理。[onInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))还可以返回false，这样的话，该次事件则会通过View树继续向下传递，直到到达目标View为止，目标View会在自己的[onTouchEvent()](https://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法中处理该次事件。

在下面的示例代码中，类MyViewGroup继承了ViewGroup，并包含了多个View，这些View我们在这里称之为子View，而MyViewGroup称为父容器View。如果你在水平方向上滑动手指，那么子View皆不会收到触摸事件。MyViewGroup会通过滚动它的内部来实现触摸事件的处理。不管如何，如果你按下了子View中的按钮，或者在垂直方向上滑动，那么ViewGroup则不会去拦截这些事件，因为子View是该次事件的目标View。在这些情况下，[onInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))应该返回false，且MyViewGroup的[onTouchEvent()](https://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法也不会被调用。
```java
public class MyViewGroup extends ViewGroup {

    private int mTouchSlop;

    ...

    ViewConfiguration vc = ViewConfiguration.get(view.getContext());
    mTouchSlop = vc.getScaledTouchSlop();

    ...

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        /*
         * This method JUST determines whether we want to intercept the motion.
         * If we return true, onTouchEvent will be called and we do the actual
         * scrolling there.
         */


        final int action = MotionEventCompat.getActionMasked(ev);

        // Always handle the case of the touch gesture being complete.
        if (action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_UP) {
            // Release the scroll.
            mIsScrolling = false;
            return false; // Do not intercept touch event, let the child handle it
        }

        switch (action) {
            case MotionEvent.ACTION_MOVE: {
                if (mIsScrolling) {
                    // We're currently scrolling, so yes, intercept the
                    // touch event!
                    return true;
                }

                // If the user has dragged her finger horizontally more than
                // the touch slop, start the scroll

                // left as an exercise for the reader
                final int xDiff = calculateDistanceX(ev);

                // Touch slop should be calculated using ViewConfiguration
                // constants.
                if (xDiff > mTouchSlop) {
                    // Start scrolling!
                    mIsScrolling = true;
                    return true;
                }
                break;
            }
            ...
        }

        // In general, we don't want to intercept touch events. They should be
        // handled by the child view.
        return false;
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        // Here we actually handle the touch event (e.g. if the action is ACTION_MOVE,
        // scroll this container).
        // This method will only be called if the touch event was intercepted in
        // onInterceptTouchEvent
        ...
    }
}
```

这里要注意，ViewGroup还提供了[requestDisallowInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#requestDisallowInterceptTouchEvent(boolean))方法。当子View不希望它的父容器及祖先容器拦截触摸事件时，ViewGroup会在 [onInterceptTouchEvent()](https://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent))方法中对其进行调用，从而判断是否要拦截本次事件。

##使用[ViewConfiguration](https://developer.android.com/reference/android/view/ViewConfiguration.html)常量
在上面的代码中使用了[ViewConfiguration](https://developer.android.com/reference/android/view/ViewConfiguration.html)来初始化一个名为mTouchSlop的变量。你可以使用[ViewConfiguration](https://developer.android.com/reference/android/view/ViewConfiguration.html)来访问Android系统所使用的常用距离、速度及时间。

"mTouchSlop"引用了触摸事件在被拦截之前手指移动的以像素为单位的距离。Touch slop经常被用来在用户在执行触摸操作时防止产生意外滚动。

ViewConfiguration的另外两个常用方法是[getScaledMinimumFlingVelocity()](https://developer.android.com/reference/android/view/ViewConfiguration.html#getScaledMinimumFlingVelocity())和[getScaledMaximumFlingVelocity()](https://developer.android.com/reference/android/view/ViewConfiguration.html#getScaledMaximumFlingVelocity())。这两个方法分别返回了用于初始化滚动的最小、最大的速度值。以每秒几像素为单位：

```java
ViewConfiguration vc = ViewConfiguration.get(view.getContext());
private int mSlop = vc.getScaledTouchSlop();
private int mMinFlingVelocity = vc.getScaledMinimumFlingVelocity();
private int mMaxFlingVelocity = vc.getScaledMaximumFlingVelocity();

...

case MotionEvent.ACTION_MOVE: {
    ...
    float deltaX = motionEvent.getRawX() - mDownX;
    if (Math.abs(deltaX) > mSlop) {
        // A swipe occurred, do something
    }

...

case MotionEvent.ACTION_UP: {
    ...
    } if (mMinFlingVelocity <= velocityX && velocityX <= mMaxFlingVelocity
            && velocityY < velocityX) {
        // The criteria have been satisfied, do something
    }
}
```

##扩展子View的触控区域
Android提供的[TouchDelegate](https://developer.android.com/reference/android/view/TouchDelegate.html)使扩展子View的触控区域成为了可能。这对于子View本身特别小，而它的触控区域需要很大时很有用。如果需要的话，你也可以使用这种方式来缩小子View的触控区域。

在下面的示例中，ImageButton作为我们的"delegate view"(这里的意思是需要父容器扩展触控区域的那个View)。下面是示例的布局文件：
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
     android:id="@+id/parent_layout"
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     tools:context=".MainActivity" >

     <ImageButton android:id="@+id/button"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:background="@null"
          android:src="@drawable/icon" />
</RelativeLayout>
```

下面的代码做了以下这些事情：

- 获得父容器View，并Post一个Runnale对象到UI线程。这可以确保在调用[getHitRect()](https://developer.android.com/reference/android/view/View.html#getHitRect(android.graphics.Rect))方法之前父容器已经对子View完成了排布。[getHitRect()](https://developer.android.com/reference/android/view/View.html#getHitRect(android.graphics.Rect))会返回父容器坐标内当前View的点击矩阵(触控区域)。
- 找到ImageButton，然后调用它的[getHitRect()](https://developer.android.com/reference/android/view/View.html#getHitRect(android.graphics.Rect))方法获得该View的触控边界。
- 扩大ImageButton的触控区域。
- 实例化一个[TouchDelegate](https://developer.android.com/reference/android/view/TouchDelegate.html)，将要扩展的触控区域矩阵与要扩展触控区域的ImageView作为参数传入。
- 将[TouchDelegate](https://developer.android.com/reference/android/view/TouchDelegate.html)设置给父容器View，只有这样做，我们所触碰到的扩展区域才会被路由到子View上。

在TouchDelegate代理的范围内，父容器View将会接收所有的触摸事件。如果触摸事件发生在子View本身的触控区域内，那么父容器View会将所有的触摸事件传给子View处理：
```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Get the parent view
        View parentView = findViewById(R.id.parent_layout);

        parentView.post(new Runnable() {
            // Post in the parent's message queue to make sure the parent
            // lays out its children before you call getHitRect()
            @Override
            public void run() {
                // The bounds for the delegate view (an ImageButton
                // in this example)
                Rect delegateArea = new Rect();
                ImageButton myButton = (ImageButton) findViewById(R.id.button);
                myButton.setEnabled(true);
                myButton.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View view) {
                        Toast.makeText(MainActivity.this,
                                "Touch occurred within ImageButton touch region.",
                                Toast.LENGTH_SHORT).show();
                    }
                });

                // The hit rectangle for the ImageButton
                myButton.getHitRect(delegateArea);

                // Extend the touch area of the ImageButton beyond its bounds
                // on the right and bottom.
                delegateArea.right += 100;
                delegateArea.bottom += 100;

                // Instantiate a TouchDelegate.
                // "delegateArea" is the bounds in local coordinates of
                // the containing view to be mapped to the delegate view.
                // "myButton" is the child view that should receive motion
                // events.
                TouchDelegate touchDelegate = new TouchDelegate(delegateArea,
                        myButton);

                // Sets the TouchDelegate on the parent view, such that touches
                // within the touch delegate bounds are routed to the child.
                if (View.class.isInstance(myButton.getParent())) {
                    ((View) myButton.getParent()).setTouchDelegate(touchDelegate);
                }
            }
        });
    }
}
```

