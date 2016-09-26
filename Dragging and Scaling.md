原文地址：[http://android.xsoftlab.net/training/gestures/scale.html](http://android.xsoftlab.net/training/gestures/scale.html)

这节课主要学习如何使用触摸手势来拖动或者放大屏幕上的对象，以及使用[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法来拦截触摸事件。

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

##平移拖动
