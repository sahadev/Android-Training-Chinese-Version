原文地址：[https://developer.android.com/training/gestures/scale.html](https://developer.android.com/training/gestures/scale.html)

这节课主要学习如何使用触摸手势来拖动或者放大屏幕上的对象。

##拖动对象
> 如果你的重点在Android 3.0以上的版本，那么你可以使用内置的拖拽事件监听器[View.OnDragListener](http://android.xsoftlab.net/reference/android/view/View.OnDragListener.html)。

触摸手势最常见的操作就是使用它来拖动屏幕上的对象。接下来的代码会允许用户拖动屏幕上的图像。要注意以下几点：

- 在拖动操作中，APP会一直保持手指拖动的轨迹，就算是另一只手指触到屏幕也是。举个例子，想象一根手指拖动着一张图像，这时用户将第二根手指放置到屏幕上，如果APP只是追踪第一根手指的轨迹，那么它会将第二根手指作为默认位置，并会将图像移动到这个位置。
- 为了防止这样的事件发生，你的APP需要区分第一根手指与其它手指。为此，需要追踪 [ACTION_POINTER_DOWN](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_POINTER_DOWN) 及 [ACTION_POINTER_UP](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_POINTER_UP) 。[ACTION_POINTER_DOWN](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_POINTER_DOWN) 及 [ACTION_POINTER_UP](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_POINTER_UP)在第二根手指落下或抬起的时候由[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))回调方法传回。
- 在[ACTION_POINTER_UP](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_POINTER_UP)的情况下，示例提取了这个事件的索引，并确保当前活动的指针不是那个已经不在屏幕上的指针。如果是那个指针的话，那么APP会选择一个不同的指针使其活动并保存它的X及Y的位置。一旦这个值被保存下来，那么APP将会一直计算起始点到当前点的距离。

下面的代码使用户可以在屏幕上拖动对象。它记录了当前活动指针的初始位置，并计算了它所偏移的距离，并将对象移动到新的位置上。它会正确的管理其它手指的可能性。

这里要注意，代码段使用了[getActionMasked()](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#getActionMasked())方法。你应该一直使用这个方法来接收[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)对象的活动。与[getAction()](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#getAction())方法不同，[getActionMasked()](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#getActionMasked())工作于多指模式内。它会返回被执行的掩饰活动，不包括指针索引比特。
```java
// The ‘active pointer’ is the one currently moving our object.
private int mActivePointerId = INVALID_POINTER_ID;
@Override
public boolean onTouchEvent(MotionEvent ev) {
    // Let the ScaleGestureDetector inspect all events.
    mScaleDetector.onTouchEvent(ev);
             
    final int action = MotionEventCompat.getActionMasked(ev); 
        
    switch (action) { 
    case MotionEvent.ACTION_DOWN: {
        final int pointerIndex = MotionEventCompat.getActionIndex(ev); 
        final float x = MotionEventCompat.getX(ev, pointerIndex); 
        final float y = MotionEventCompat.getY(ev, pointerIndex); 
            
        // Remember where we started (for dragging)
        mLastTouchX = x;
        mLastTouchY = y;
        // Save the ID of this pointer (for dragging)
        mActivePointerId = MotionEventCompat.getPointerId(ev, 0);
        break;
    }
            
    case MotionEvent.ACTION_MOVE: {
        // Find the index of the active pointer and fetch its position
        final int pointerIndex = 
                MotionEventCompat.findPointerIndex(ev, mActivePointerId);  
            
        final float x = MotionEventCompat.getX(ev, pointerIndex);
        final float y = MotionEventCompat.getY(ev, pointerIndex);
            
        // Calculate the distance moved
        final float dx = x - mLastTouchX;
        final float dy = y - mLastTouchY;
        mPosX += dx;
        mPosY += dy;
        invalidate();
        // Remember this touch position for the next move event
        mLastTouchX = x;
        mLastTouchY = y;
        break;
    }
            
    case MotionEvent.ACTION_UP: {
        mActivePointerId = INVALID_POINTER_ID;
        break;
    }
            
    case MotionEvent.ACTION_CANCEL: {
        mActivePointerId = INVALID_POINTER_ID;
        break;
    }
        
    case MotionEvent.ACTION_POINTER_UP: {
            
        final int pointerIndex = MotionEventCompat.getActionIndex(ev); 
        final int pointerId = MotionEventCompat.getPointerId(ev, pointerIndex); 
        if (pointerId == mActivePointerId) {
            // This was our active pointer going up. Choose a new
            // active pointer and adjust accordingly.
            final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
            mLastTouchX = MotionEventCompat.getX(ev, newPointerIndex); 
            mLastTouchY = MotionEventCompat.getY(ev, newPointerIndex); 
            mActivePointerId = MotionEventCompat.getPointerId(ev, newPointerIndex);
        }
        break;
    }
    }       
    return true;
}
```

##平移
上面的部分展示了如何在屏幕上拖动对象。另一个通用的场景就是平移了，平移的意思是：用户的拖动动作引起的x及y轴方向上的滚动。上面的代码直接将[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)拦截实现拖动。这部分的代码将会采用另一种更具有优势的方法，以便支持通用手势。它重写了[GestureDetector.SimpleOnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html)的[onScroll()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onScroll(android.view.MotionEvent,%20android.view.MotionEvent,%20float,%20float))方法。

只有用户在使用手指移动内容时[onScroll()](http://https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onScroll(android.view.MotionEvent,%20android.view.MotionEvent,%20float,%20float))才会被调用。[onScroll()](http://https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onScroll(android.view.MotionEvent,%20android.view.MotionEvent,%20float,%20float))只有在手指按下的时候才会调用，一旦手指离开屏幕，那么平移手势也随之终止。

下面是[onScroll()](http://https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onScroll(android.view.MotionEvent,%20android.view.MotionEvent,%20float,%20float))的摘要：
```java
// The current viewport. This rectangle represents the currently visible
// chart domain and range.
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);

// The current destination rectangle (in pixel coordinates) into which the
// chart data should be drawn.
private Rect mContentRect;

private final GestureDetector.SimpleOnGestureListener mGestureListener
            = new GestureDetector.SimpleOnGestureListener() {
...

@Override
public boolean onScroll(MotionEvent e1, MotionEvent e2,
            float distanceX, float distanceY) {
    // Scrolling uses math based on the viewport (as opposed to math using pixels).

    // Pixel offset is the offset in screen pixels, while viewport offset is the
    // offset within the current viewport.
    float viewportOffsetX = distanceX * mCurrentViewport.width()
            / mContentRect.width();
    float viewportOffsetY = -distanceY * mCurrentViewport.height()
            / mContentRect.height();
    ...
    // Updates the viewport, refreshes the display.
    setViewportBottomLeft(
            mCurrentViewport.left + viewportOffsetX,
            mCurrentViewport.bottom + viewportOffsetY);
    ...
    return true;
}
```
下面是setViewportBottomLeft()方法的实现，它主要实现了移动内容的逻辑：
```java
/**
 * Sets the current viewport (defined by mCurrentViewport) to the given
 * X and Y positions. Note that the Y value represents the topmost pixel position,
 * and thus the bottom of the mCurrentViewport rectangle.
 */
private void setViewportBottomLeft(float x, float y) {
    /*
     * Constrains within the scroll range. The scroll range is simply the viewport
     * extremes (AXIS_X_MAX, etc.) minus the viewport size. For example, if the
     * extremes were 0 and 10, and the viewport size was 2, the scroll range would
     * be 0 to 8.
     */

    float curWidth = mCurrentViewport.width();
    float curHeight = mCurrentViewport.height();
    x = Math.max(AXIS_X_MIN, Math.min(x, AXIS_X_MAX - curWidth));
    y = Math.max(AXIS_Y_MIN + curHeight, Math.min(y, AXIS_Y_MAX));

    mCurrentViewport.set(x, y - curHeight, x + curWidth, y);

    // Invalidates the View to update the display.
    ViewCompat.postInvalidateOnAnimation(this);
}
```

##缩放
在[Detecting Common Gestures](https://developer.android.com/training/gestures/detector.html)中，我们讨论到[GestureDetector](https://developer.android.com/reference/android/view/GestureDetector.html)可以帮助我们来检测比如滑动、滚动、长按等手势。而对于缩放，Android提供了[ScaleGestureDetector. GestureDetector](https://developer.android.com/reference/android/view/GestureDetector.html) 以及 [ScaleGestureDetector](https://developer.android.com/reference/android/view/ScaleGestureDetector.html)。

为了可以反馈检测到的手势事件，手势探测器使用了监听器对象[ScaleGestureDetector.OnScaleGestureListener](https://developer.android.com/reference/android/view/ScaleGestureDetector.OnScaleGestureListener.html)。如果你只关心部分手势的话，Android还提供了[ScaleGestureDetector.SimpleOnScaleGestureListener](https://developer.android.com/reference/android/view/ScaleGestureDetector.SimpleOnScaleGestureListener.html)，你可以通过重写它的方法来使用它。

###缩放基础示例
下面的代码是缩放所需要的基础：
```java
private ScaleGestureDetector mScaleDetector;
private float mScaleFactor = 1.f;

public MyCustomView(Context mContext){
    ...
    // View code goes here
    ...
    mScaleDetector = new ScaleGestureDetector(context, new ScaleListener());
}

@Override
public boolean onTouchEvent(MotionEvent ev) {
    // Let the ScaleGestureDetector inspect all events.
    mScaleDetector.onTouchEvent(ev);
    return true;
}

@Override
public void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    canvas.save();
    canvas.scale(mScaleFactor, mScaleFactor);
    ...
    // onDraw() code goes here
    ...
    canvas.restore();
}

private class ScaleListener
        extends ScaleGestureDetector.SimpleOnScaleGestureListener {
    @Override
    public boolean onScale(ScaleGestureDetector detector) {
        mScaleFactor *= detector.getScaleFactor();

        // Don't let the object get too small or too large.
        mScaleFactor = Math.max(0.1f, Math.min(mScaleFactor, 5.0f));

        invalidate();
        return true;
    }
}
```

###稍微复杂点的示例
下面是一个稍微复杂一点的示例，它摘自与这节课所提供的示例InteractiveChart(**PS:**示例工程请参见原网页)。InteractiveChart同时支持平移、缩放，它使用了[ScaleGestureDetector](https://developer.android.com/reference/android/view/ScaleGestureDetector.html)的“平移”([getCurrentSpanX/Y](https://developer.android.com/reference/android/view/ScaleGestureDetector.html#getCurrentSpanX()))及“焦点” ([getFocusX/Y](https://developer.android.com/reference/android/view/ScaleGestureDetector.html#getFocusX()))特性：

```java
@Override
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);
private Rect mContentRect;
private ScaleGestureDetector mScaleGestureDetector;
...
public boolean onTouchEvent(MotionEvent event) {
    boolean retVal = mScaleGestureDetector.onTouchEvent(event);
    retVal = mGestureDetector.onTouchEvent(event) || retVal;
    return retVal || super.onTouchEvent(event);
}

/**
 * The scale listener, used for handling multi-finger scale gestures.
 */
private final ScaleGestureDetector.OnScaleGestureListener mScaleGestureListener
        = new ScaleGestureDetector.SimpleOnScaleGestureListener() {
    /**
     * This is the active focal point in terms of the viewport. Could be a local
     * variable but kept here to minimize per-frame allocations.
     */
    private PointF viewportFocus = new PointF();
    private float lastSpanX;
    private float lastSpanY;

    // Detects that new pointers are going down.
    @Override
    public boolean onScaleBegin(ScaleGestureDetector scaleGestureDetector) {
        lastSpanX = ScaleGestureDetectorCompat.
                getCurrentSpanX(scaleGestureDetector);
        lastSpanY = ScaleGestureDetectorCompat.
                getCurrentSpanY(scaleGestureDetector);
        return true;
    }

    @Override
    public boolean onScale(ScaleGestureDetector scaleGestureDetector) {

        float spanX = ScaleGestureDetectorCompat.
                getCurrentSpanX(scaleGestureDetector);
        float spanY = ScaleGestureDetectorCompat.
                getCurrentSpanY(scaleGestureDetector);

        float newWidth = lastSpanX / spanX * mCurrentViewport.width();
        float newHeight = lastSpanY / spanY * mCurrentViewport.height();

        float focusX = scaleGestureDetector.getFocusX();
        float focusY = scaleGestureDetector.getFocusY();
        // Makes sure that the chart point is within the chart region.
        // See the sample for the implementation of hitTest().
        hitTest(scaleGestureDetector.getFocusX(),
                scaleGestureDetector.getFocusY(),
                viewportFocus);

        mCurrentViewport.set(
                viewportFocus.x
                        - newWidth * (focusX - mContentRect.left)
                        / mContentRect.width(),
                viewportFocus.y
                        - newHeight * (mContentRect.bottom - focusY)
                        / mContentRect.height(),
                0,
                0);
        mCurrentViewport.right = mCurrentViewport.left + newWidth;
        mCurrentViewport.bottom = mCurrentViewport.top + newHeight;
        ...
        // Invalidates the View to update the display.
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);

        lastSpanX = spanX;
        lastSpanY = spanY;
        return true;
    }
};
```
