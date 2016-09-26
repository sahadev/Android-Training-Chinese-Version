原文地址：[http://android.xsoftlab.net/training/gestures/index.html](http://android.xsoftlab.net/training/gestures/index.html)

#引言
这节课将会学习如何使用户通过触摸手势与APP产生交互。Android提供了许多API来辅助你创建与检测手势。

尽管APP不应该将触摸手势作为基本的特性，但是APP使用了触摸手势后可以使APP迅速的增长它的有益性与吸引力。

为了提供一种一贯的，直观的经验，APP应当接受Android惯用的触摸手势。

#检测通用手势
当用户将一根或者多根手指放置触摸屏上时就会使触摸事件发生，应用程序需要将这次的触摸解释为一种指定的手势事件。这里有两种相应的阶段来检测手势：

- 1.收集触摸事件的相关数据。
- 2.解释这些数据，查看是否有程序所支持的任何标准手势。

##收集数据
当用户将手指放在屏幕上时，这会触发收到触摸手势的View的onTouchEvent()方法。

手势开始于用户第一次触摸屏幕时，接下来系统会追踪手指的位置，最后通过手指离开屏幕所捕获的最终事件而结束。这整个交互过程中，MotionEvent对象由onTouchEvent()方法分发，并提供了每个事件的详细信息。APP可以使用MotionEvent所提供的数据来检查是否有APP所关心的事件发生。

###为Activity或View捕获触摸事件
为了拦截Activity或者View的触摸事件，需要重写它们的onTouchEvent()回调方法。

下面的代码使用了getActionMasked()方法来提取event参数中的用户执行的行为。它提供了你所关心的原始数据：
```java
public class MainActivity extends Activity {
...
// This example shows an Activity, but you would use the same approach if
// you were subclassing a View.
@Override
public boolean onTouchEvent(MotionEvent event){ 
        
    int action = MotionEventCompat.getActionMasked(event);
        
    switch(action) {
        case (MotionEvent.ACTION_DOWN) :
            Log.d(DEBUG_TAG,"Action was DOWN");
            return true;
        case (MotionEvent.ACTION_MOVE) :
            Log.d(DEBUG_TAG,"Action was MOVE");
            return true;
        case (MotionEvent.ACTION_UP) :
            Log.d(DEBUG_TAG,"Action was UP");
            return true;
        case (MotionEvent.ACTION_CANCEL) :
            Log.d(DEBUG_TAG,"Action was CANCEL");
            return true;
        case (MotionEvent.ACTION_OUTSIDE) :
            Log.d(DEBUG_TAG,"Movement occurred outside bounds " +
                    "of current screen element");
            return true;      
        default : 
            return super.onTouchEvent(event);
    }      
}
```

###为单个View捕获触摸事件
除了onTouchEvent()方法之外，你还可以使用View.OnTouchListener来监听触摸手势。这使得不继承View还可以监听触摸事件成为了可能：
```java
View myView = findViewById(R.id.my_view); 
myView.setOnTouchListener(new OnTouchListener() {
    public boolean onTouch(View v, MotionEvent event) {
        // ... Respond to touch events       
        return true;
    }
});
```

要注意所创建的监听器在[ACTION_DOWN](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_DOWN)事件时返回的false。如果你这么做了，那么监听器接下来对于[ACTION_MOVE](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_MOVE)及[ACTION_UP](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_UP)等一系列事件将不会被调用。这是因为[ACTION_DOWN](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_DOWN)事件是所有触摸事件的起点。

如果你创建了一个自定义View，你可以像上面描述的那样重写[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法。

##检查手势
Android提供了[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)类来检查通用手势。一些手势支持包括[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent)), [onLongPress()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onLongPress(android.view.MotionEvent)), [onFling()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onFling(android.view.MotionEvent, android.view.MotionEvent, float, float))等等。你可以将[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)与[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))结合使用。

###检查所有支持的手势
当你在实例化[GestureDetectorCompat](http://android.xsoftlab.net/reference/android/support/v4/view/GestureDetectorCompat.html)对象时，其中一个参数需要实现[GestureDetector.OnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html)接口。

[GestureDetector.OnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html)接口的作用是，在指定的触摸事件发生时用来通知用户。为了使[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)对象可以接收到触摸事件，你需要重写View或者Activity的[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法，并将所有的事件传递给[GestureDetector](http://android.xsoftlab.net/reference/android/view/GestureDetector.html)对象。

在下面的代码中，由[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法所返回的返回值true代表了你负责处理这次的触摸事件。返回值false代表了忽略这次事件，直到这次的触摸事件被完全处理完毕。

运行下面的代码找找事件是如何触发的感觉，当你在触摸屏交互的时候，以及每一种事件时[MotionEvent](http://android.xsoftlab.net/reference/android/view/MotionEvent.html)对象的内容。你将会意识到一个简单的事件将会有多么庞大的数据将被处理。
```java
public class MainActivity extends Activity implements 
        GestureDetector.OnGestureListener,
        GestureDetector.OnDoubleTapListener{
    
    private static final String DEBUG_TAG = "Gestures";
    private GestureDetectorCompat mDetector; 
    // Called when the activity is first created. 
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Instantiate the gesture detector with the
        // application context and an implementation of
        // GestureDetector.OnGestureListener
        mDetector = new GestureDetectorCompat(this,this);
        // Set the gesture detector as the double tap
        // listener.
        mDetector.setOnDoubleTapListener(this);
    }
    @Override 
    public boolean onTouchEvent(MotionEvent event){ 
        this.mDetector.onTouchEvent(event);
        // Be sure to call the superclass implementation
        return super.onTouchEvent(event);
    }
    @Override
    public boolean onDown(MotionEvent event) { 
        Log.d(DEBUG_TAG,"onDown: " + event.toString()); 
        return true;
    }
    @Override
    public boolean onFling(MotionEvent event1, MotionEvent event2, 
            float velocityX, float velocityY) {
        Log.d(DEBUG_TAG, "onFling: " + event1.toString()+event2.toString());
        return true;
    }
    @Override
    public void onLongPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onLongPress: " + event.toString()); 
    }
    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX,
            float distanceY) {
        Log.d(DEBUG_TAG, "onScroll: " + e1.toString()+e2.toString());
        return true;
    }
    @Override
    public void onShowPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onShowPress: " + event.toString());
    }
    @Override
    public boolean onSingleTapUp(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapUp: " + event.toString());
        return true;
    }
    @Override
    public boolean onDoubleTap(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTap: " + event.toString());
        return true;
    }
    @Override
    public boolean onDoubleTapEvent(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTapEvent: " + event.toString());
        return true;
    }
    @Override
    public boolean onSingleTapConfirmed(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapConfirmed: " + event.toString());
        return true;
    }
}
```

###检查被支持手势的子集
如果你只是想处理几种手势，那么你可以继承[GestureDetector.SimpleOnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html)接口。

[GestureDetector.SimpleOnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html)提供了对于[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))中所有的实现，这样你只用重写你所关心的方法。比如说，在下面的代码中，创建了一个继承了[GestureDetector.SimpleOnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html)接口的类，然后重写了它的[onFling()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onFling(android.view.MotionEvent, android.view.MotionEvent, float, float))方法及[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent))方法。

无论你是否使用了[GestureDetector.OnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html)接口，最佳的练习点在于重写了一个返回false的[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent))方法。这是因为所有的手势都是从[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent))消息开始的。如果在[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent))方法中返回了false，就像[GestureDetector.SimpleOnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.SimpleOnGestureListener.html)默认做的那样，那么系统会认为你想忽略余下的手势，并且[GestureDetector.OnGestureListener](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html)接口的其它方法都不会被调用。这会在APP内埋下一个潜在的不易察觉的问题。如果你确认你想忽略整个手势流程，那么[onDown()](http://android.xsoftlab.net/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent))中的结果false将是唯一的一次机会。
```java
public class MainActivity extends Activity { 
    
    private GestureDetectorCompat mDetector; 
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mDetector = new GestureDetectorCompat(this, new MyGestureListener());
    }
    @Override 
    public boolean onTouchEvent(MotionEvent event){ 
        this.mDetector.onTouchEvent(event);
        return super.onTouchEvent(event);
    }
    
    class MyGestureListener extends GestureDetector.SimpleOnGestureListener {
        private static final String DEBUG_TAG = "Gestures"; 
        
        @Override
        public boolean onDown(MotionEvent event) { 
            Log.d(DEBUG_TAG,"onDown: " + event.toString()); 
            return true;
        }
        @Override
        public boolean onFling(MotionEvent event1, MotionEvent event2, 
                float velocityX, float velocityY) {
            Log.d(DEBUG_TAG, "onFling: " + event1.toString()+event2.toString());
            return true;
        }
    }
}
```

