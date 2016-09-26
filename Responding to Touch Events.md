原文地址：[http://android.xsoftlab.net/training/graphics/opengl/touch.html](http://android.xsoftlab.net/training/graphics/opengl/touch.html)

使图形按照程序设计的轨迹旋转对OpenGL来说还是不能发挥出它应有的实力。但要是能使用户可以直接控制图形的旋转，这才是OpenGL的真正目的。它真正的关键所在就是使程序可以交互式触摸。这主要靠重写GLSurfaceView的onTouchEvent()的方法来实现触摸事件的监听。

这节课将会展示如何监听触摸事件来使用户可以旋转图形。

##设置触摸监听器
为了可以使OpenGL监听触摸事件，必须重写GLSurfaceView类中的onTouchEvent()方法。下面的实现展示了如何监听[MotionEvent.ACTION_MOVE](http://android.xsoftlab.net/reference/android/view/MotionEvent.html#ACTION_MOVE)事件，以及如何使事件驱动图形的旋转.
```java
private final float TOUCH_SCALE_FACTOR = 180.0f / 320;
private float mPreviousX;
private float mPreviousY;
@Override
public boolean onTouchEvent(MotionEvent e) {
    // MotionEvent reports input details from the touch screen
    // and other input controls. In this case, you are only
    // interested in events where the touch position changed.
    float x = e.getX();
    float y = e.getY();
    switch (e.getAction()) {
        case MotionEvent.ACTION_MOVE:
            float dx = x - mPreviousX;
            float dy = y - mPreviousY;
            // reverse direction of rotation above the mid-line
            if (y > getHeight() / 2) {
              dx = dx * -1 ;
            }
            // reverse direction of rotation to left of the mid-line
            if (x < getWidth() / 2) {
              dy = dy * -1 ;
            }
            mRenderer.setAngle(
                    mRenderer.getAngle() +
                    ((dx + dy) * TOUCH_SCALE_FACTOR));
            requestRender();
    }
    mPreviousX = x;
    mPreviousY = y;
    return true;
}
```

这里需要注意的是，在计算完旋转的角度之后，这个方法调用了requestRender()方法，这个方法会通知渲染器可以渲染了。这个方法放在这个地方是最合适的，因为帧在这之前并不需要重新绘制，除非在角度上发生了变化。不管怎么样，这个方法并不会对效率有任何影响，除非你也设置了在数据发生改变的时候重新绘制的请求。这种请求通过setRenderMode()方法设置，所以要确保下面这行代码没有被注释：
```java
public MyGLSurfaceView(Context context) {
    ...
    // Render the view only when there is a change in the drawing data
    setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
}
```

##暴露旋转角度
上面的示例代码会要求提供一个公开的成员方法来暴露旋转的角度。一旦渲染代码运行在子线程当中，那么必须将这个公共成员声明为volatile。下面的代码声明了这个volatile的属性，并暴露了它的get,set方法：
```java
public class MyGLRenderer implements GLSurfaceView.Renderer {
    ...
    public volatile float mAngle;
    public float getAngle() {
        return mAngle;
    }
    public void setAngle(float angle) {
        mAngle = angle;
    }
}
```

##请求旋转
为了触摸事件驱动旋转，需要注释生成角度的代码，然后添加mAngle成员属性，mAngle中包含了触摸事件所生成的角度：
```java
public void onDrawFrame(GL10 gl) {
    ...
    float[] scratch = new float[16];
    // Create a rotation for the triangle
    // long time = SystemClock.uptimeMillis() % 4000L;
    // float angle = 0.090f * ((int) time);
    Matrix.setRotateM(mRotationMatrix, 0, mAngle, 0, 0, -1.0f);
    // Combine the rotation matrix with the projection and camera view
    // Note that the mMVPMatrix factor *must be first* in order
    // for the matrix multiplication product to be correct.
    Matrix.multiplyMM(scratch, 0, mMVPMatrix, 0, mRotationMatrix, 0);
    // Draw triangle
    mTriangle.draw(scratch);
}
```

如果完成了上面所描述的步骤，那么启动程序，然后在屏幕上拖动就可以使三角形旋转起来：
![](http://android.xsoftlab.net/images/opengl/ogl-triangle-touch.png)