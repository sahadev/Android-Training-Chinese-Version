原文地址：[http://android.xsoftlab.net/training/gestures/scroll.html](http://android.xsoftlab.net/training/gestures/scroll.html)

在Android中，滑动经常由ScrollView类来实现。任何超出容器边界的布局都会将自身内嵌在ScrollView中，以便提供可滚动的视图效果。实现自定义滚动只有在特定的必要场景下才会被用到。这节课将会描述这样一种场景：使用scroller展示一种可滚动的效果。

你可以使用Scroller或者OverScroller来搜集降低滚动速度的必要数据。这两个类很相似，但是OverScroller包含了一些用于指示用户已经到达布局边界的方法。InteractiveChart示例在用户到达布局边界时使用了EdgeEffect类(实际上是EdgeEffectCompat类)来展示一种"增长(**glow**)"的效果。

> **Note:** 我们推荐使用OverScroller。这个类提供了对老版本良好的兼容性。还应该注意，在实现滑动自身时，通常不需要使用scroller类。如果你将布局嵌入到ScrollView或HorizontalScrollView中，那么它们会将滚动的相关事情做好。

Scroller用于在时间轴上做动画滚动效果，使用了平台标准的滚动物理学(摩擦力、速度等等)。Scroller本身不会绘制任何东西。Scroller随着时间的变化追踪滚动的偏移量，但是它们不会自动的将这些值应用在你的View上。你需要自己将这些值获取并应用，这会使滑动的效果更流畅。

##了解滑动术语
"Scrolling"这个词在Android中可被翻译为各种不同的意思，这取决于具体的上下文。

**Scrolling**是一种viewport(viewport的意思是，你所看到的内容的窗口)移动的通用处理过程。当scrolling处于x轴及y轴上时，这被称为**"平移(panning)"**。示例程序提供了相关的类：InteractiveChart，演示了滑动的两种不同类型，dragging(拖动)及flinging(滑动)：

- **Dragging** 该滚动类型在这种情况下发生：当用户的手指在屏幕上来回滑动时。简单的Dragging由GestureDetector.OnGestureListener接口的onScroll()方法实现。
- **Flinging** 该滚动类型在这种情况下发生：当用户的手指快速的在屏幕上滑动并离开手指时。在用户离开了手指之后，通常希望继续保持滚动状态，并慢慢减速，直到viewport停止移动。Flinging由GestureDetector.OnGestureListener接口的onFling()方法及scroller对象所实现。这个使用示例就是我们这节课的主题。

将scroller对象与滑动手势结合使用是一种共通的方法。你可以直接重写[onTouchEvent()](http://android.xsoftlab.net/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))方法来处理触摸事件，减低滑动效果以响应这些触摸事件。

##实现基础滚动
这部分章节将会描述如何使用scroller。下面的代码段摘自示例InteractiveChart。它使用了一个GestureDetector对象，并重写了GestureDetector.SimpleOnGestureListener的onFling()方法。它使用了OverScroller来追踪滑动中的手势。如果用户在滑动手势之后到达了内容的边缘，那么APP会显示一个"glow"的效果。

> **Note:** 示例APP InteractiveChart 展示了一个图表，这个图标可以缩放、平移、滚动等等。在下面的代码段中，mContentRect代表了矩形的坐标点，而绘制图表的View则居于其中。在给定的时间内，一个总表的子集将会被绘制到这块区域内。mCurrentViewport则代表了屏幕中当前可视的部分图表。因为像素的偏移量通常被当做整型，mContentRect的类型是Rect。因为图形的领域范围是小数类型，所以mCurrentViewport的类型是RectF。

代码段的第一部分展示了onFling()方法的实现：
```java
// The current viewport. This rectangle represents the currently visible 
// chart domain and range. The viewport is the part of the app that the
// user manipulates via touch gestures.
private RectF mCurrentViewport = 
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);
// The current destination rectangle (in pixel coordinates) into which the 
// chart data should be drawn.
private Rect mContentRect;
private OverScroller mScroller;
private RectF mScrollerStartViewport;
...
private final GestureDetector.SimpleOnGestureListener mGestureListener
        = new GestureDetector.SimpleOnGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // Initiates the decay phase of any active edge effects.
        releaseEdgeEffects();
        mScrollerStartViewport.set(mCurrentViewport);
        // Aborts any active scroll animations and invalidates.
        mScroller.forceFinished(true);
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);
        return true;
    }
    ...
    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2, 
            float velocityX, float velocityY) {
        fling((int) -velocityX, (int) -velocityY);
        return true;
    }
};
private void fling(int velocityX, int velocityY) {
    // Initiates the decay phase of any active edge effects.
    releaseEdgeEffects();
    // Flings use math in pixels (as opposed to math based on the viewport).
    Point surfaceSize = computeScrollSurfaceSize();
    mScrollerStartViewport.set(mCurrentViewport);
    int startX = (int) (surfaceSize.x * (mScrollerStartViewport.left - 
            AXIS_X_MIN) / (
            AXIS_X_MAX - AXIS_X_MIN));
    int startY = (int) (surfaceSize.y * (AXIS_Y_MAX - 
            mScrollerStartViewport.bottom) / (
            AXIS_Y_MAX - AXIS_Y_MIN));
    // Before flinging, aborts the current animation.
    mScroller.forceFinished(true);
    // Begins the animation
    mScroller.fling(
            // Current scroll position
            startX,
            startY,
            velocityX,
            velocityY,
            /*
             * Minimum and maximum scroll positions. The minimum scroll 
             * position is generally zero and the maximum scroll position 
             * is generally the content size less the screen size. So if the 
             * content width is 1000 pixels and the screen width is 200  
             * pixels, the maximum scroll offset should be 800 pixels.
             */
            0, surfaceSize.x - mContentRect.width(),
            0, surfaceSize.y - mContentRect.height(),
            // The edges of the content. This comes into play when using
            // the EdgeEffect class to draw "glow" overlays.
            mContentRect.width() / 2,
            mContentRect.height() / 2);
    // Invalidates to trigger computeScroll()
    ViewCompat.postInvalidateOnAnimation(this);
}
```

当onFling()方法调用postInvalidateOnAnimation()方法时，它会触发computeScroll()来更新x及y的值。

大多数View会将scroller对象的x及y的位置值直接设置给scrollTo()。下面的computeScroll()的实现采用了不同的方法：它调用computeScrollOffset()方法来获取x及y的当前位置。当显示的标准越过边距时，会展示一个"glow"的效果，代码会设置一个越过边缘的效果，并会调用postInvalidateOnAnimation()方法来使View重新绘制。
```java
// Edge effect / overscroll tracking objects.
private EdgeEffectCompat mEdgeEffectTop;
private EdgeEffectCompat mEdgeEffectBottom;
private EdgeEffectCompat mEdgeEffectLeft;
private EdgeEffectCompat mEdgeEffectRight;
private boolean mEdgeEffectTopActive;
private boolean mEdgeEffectBottomActive;
private boolean mEdgeEffectLeftActive;
private boolean mEdgeEffectRightActive;
@Override
public void computeScroll() {
    super.computeScroll();
    boolean needsInvalidate = false;
    // The scroller isn't finished, meaning a fling or programmatic pan 
    // operation is currently active.
    if (mScroller.computeScrollOffset()) {
        Point surfaceSize = computeScrollSurfaceSize();
        int currX = mScroller.getCurrX();
        int currY = mScroller.getCurrY();
        boolean canScrollX = (mCurrentViewport.left > AXIS_X_MIN
                || mCurrentViewport.right < AXIS_X_MAX);
        boolean canScrollY = (mCurrentViewport.top > AXIS_Y_MIN
                || mCurrentViewport.bottom < AXIS_Y_MAX);
        /*          
         * If you are zoomed in and currX or currY is
         * outside of bounds and you're not already
         * showing overscroll, then render the overscroll
         * glow edge effect.
         */
        if (canScrollX
                && currX < 0
                && mEdgeEffectLeft.isFinished()
                && !mEdgeEffectLeftActive) {
            mEdgeEffectLeft.onAbsorb((int) 
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectLeftActive = true;
            needsInvalidate = true;
        } else if (canScrollX
                && currX > (surfaceSize.x - mContentRect.width())
                && mEdgeEffectRight.isFinished()
                && !mEdgeEffectRightActive) {
            mEdgeEffectRight.onAbsorb((int) 
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectRightActive = true;
            needsInvalidate = true;
        }
        if (canScrollY
                && currY < 0
                && mEdgeEffectTop.isFinished()
                && !mEdgeEffectTopActive) {
            mEdgeEffectTop.onAbsorb((int) 
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectTopActive = true;
            needsInvalidate = true;
        } else if (canScrollY
                && currY > (surfaceSize.y - mContentRect.height())
                && mEdgeEffectBottom.isFinished()
                && !mEdgeEffectBottomActive) {
            mEdgeEffectBottom.onAbsorb((int) 
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectBottomActive = true;
            needsInvalidate = true;
        }
        ...
    }
```

下面试实际放大的执行代码：
```java
// Custom object that is functionally similar to Scroller
Zoomer mZoomer;
private PointF mZoomFocalPoint = new PointF();
...
// If a zoom is in progress (either programmatically or via double
// touch), performs the zoom.
if (mZoomer.computeZoom()) {
    float newWidth = (1f - mZoomer.getCurrZoom()) * 
            mScrollerStartViewport.width();
    float newHeight = (1f - mZoomer.getCurrZoom()) * 
            mScrollerStartViewport.height();
    float pointWithinViewportX = (mZoomFocalPoint.x - 
            mScrollerStartViewport.left)
            / mScrollerStartViewport.width();
    float pointWithinViewportY = (mZoomFocalPoint.y - 
            mScrollerStartViewport.top)
            / mScrollerStartViewport.height();
    mCurrentViewport.set(
            mZoomFocalPoint.x - newWidth * pointWithinViewportX,
            mZoomFocalPoint.y - newHeight * pointWithinViewportY,
            mZoomFocalPoint.x + newWidth * (1 - pointWithinViewportX),
            mZoomFocalPoint.y + newHeight * (1 - pointWithinViewportY));
    constrainViewport();
    needsInvalidate = true;
}
if (needsInvalidate) {
    ViewCompat.postInvalidateOnAnimation(this);
}
```

下面是computeScrollSurfaceSize()方法的内容。它计算了当前可滑动的表面的尺寸，以像素为单位。举个例子，如果整个图标区域是可见的，那么它就是mContentRect的值。如果图标被放大了200%，那么返回的值是水平及垂直方向的两倍。
```java
private Point computeScrollSurfaceSize() {
    return new Point(
            (int) (mContentRect.width() * (AXIS_X_MAX - AXIS_X_MIN)
                    / mCurrentViewport.width()),
            (int) (mContentRect.height() * (AXIS_Y_MAX - AXIS_Y_MIN)
                    / mCurrentViewport.height()));
}
```

