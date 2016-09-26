原文地址：[http://android.xsoftlab.net/training/gestures/multi.html](http://android.xsoftlab.net/training/gestures/multi.html)

多点触控是指多个手指同时触摸屏幕的情况。这节课主要学习如何检查多点触控手势。

##记录多个触控点
当多根手指同时触碰到屏幕时，系统会产生以下触摸事件：

- ACTION_DOWN -第一个触碰到屏幕的点。它是手势的起始事件。这个触控点的指针数据在MotionEvent对象的索引总是0。
- ACTION_POINTER_DOWN -除第一个触碰点之外的其它点。这个触控点的指针数据的索引由getActionIndex()方法返回。
- ACTION_MOVE -屏幕上的手指位置发生变化时。
- ACTION_POINTER_UP -除最开始按下的其它触控点离开屏幕时。
- ACTION_UP -最后一个触控点离开屏幕时。

通过每一个触控点的索引及ID来追踪[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)对象中的每一个触控点：

- **Index:** 一个[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)对象将每一个触控点的相关信息存储于一个数组中。每一个触控点的索引是这个触控点在数组中的位置。[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)对象的大多数方法都可以使用这些触控点的索引来与这些点产生交互。
- **ID:** 每一个触控点也含有一个ID映射，这个映射关系在手势事件的整个生命周期内与相对应的触控点一直保持相对关系。

每个触控点的出现顺序是不固定的。因此，一个触控点的索引可以由事件移动到下一个索引，但是触控点的ID始终保持为一个常量。使用[getPointerId()](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#getPointerId(int))方法可以获得指定触控点的ID，因此可以在剩余的手势事件中还可以继续保持与这个触控点的关联。使用[findPointerIndex()](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#findPointerIndex(int))方法可以根据指定的ID获得触控点的索引：
```java
private int mActivePointerId;
public boolean onTouchEvent(MotionEvent event) {
    ....
    // Get the pointer ID
    mActivePointerId = event.getPointerId(0);
    // ... Many touch events later...
    // Use the pointer ID to find the index of the active pointer 
    // and fetch its position
    int pointerIndex = event.findPointerIndex(mActivePointerId);
    // Get the pointer's current position
    float x = event.getX(pointerIndex);
    float y = event.getY(pointerIndex);
}
```

##获取事件的行为
你应该使用[getActionMasked()](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#getActionMasked())方法来接收[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)的行为。与getAction()方法不同，[getActionMasked()](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#getActionMasked())适用于多个触控点。它将返回将要执行的行为。你可以使用[getActionIndex()](http://android.xsoftlab.net/reference/android/support/v4/view/MotionEventCompat.html#getActionIndex(android.view.MotionEvent))方法获得与之相关联的触控点的索引。下面的代码显示了这个过程：
> **Note:** 示例中使用了[MotionEventCompat](http://android.xsoftlab.net/reference/android/support/v4/view/MotionEventCompat.html)类。这个类位于支持库中。你应该使用该类以便提供良好的向后兼容性。注意，[MotionEventCompat](http://android.xsoftlab.net/reference/android/support/v4/view/MotionEventCompat.html)类并不可以替代[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)类。这个类提供了一个实用的静态方法，可以将[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)对象中所关联的行为提取出来。

```java
int action = MotionEventCompat.getActionMasked(event);
// Get the index of the pointer associated with the action.
int index = MotionEventCompat.getActionIndex(event);
int xPos = -1;
int yPos = -1;
Log.d(DEBUG_TAG,"The action is " + actionToString(action));
            
if (event.getPointerCount() > 1) {
    Log.d(DEBUG_TAG,"Multitouch event"); 
    // The coordinates of the current screen contact, relative to 
    // the responding View or Activity.  
    xPos = (int)MotionEventCompat.getX(event, index);
    yPos = (int)MotionEventCompat.getY(event, index);
} else {
    // Single touch event
    Log.d(DEBUG_TAG,"Single touch event"); 
    xPos = (int)MotionEventCompat.getX(event, index);
    yPos = (int)MotionEventCompat.getY(event, index);
}
...
// Given an action int, returns a string description
public static String actionToString(int action) {
    switch (action) {
                
        case MotionEvent.ACTION_DOWN: return "Down";
        case MotionEvent.ACTION_MOVE: return "Move";
        case MotionEvent.ACTION_POINTER_DOWN: return "Pointer Down";
        case MotionEvent.ACTION_UP: return "Up";
        case MotionEvent.ACTION_POINTER_UP: return "Pointer Up";
        case MotionEvent.ACTION_OUTSIDE: return "Outside";
        case MotionEvent.ACTION_CANCEL: return "Cancel";
    }
    return "";
}
```

有关多点触控的更多信息，可以参见课程[Dragging and Scaling](http://android.xsoftlab.net/training/gestures/scale.html). 