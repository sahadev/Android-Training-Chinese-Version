原文地址：[http://android.xsoftlab.net/training/graphics/opengl/motion.html](http://android.xsoftlab.net/training/graphics/opengl/motion.html)

在屏幕上绘制物体只是OpenGL基础的基础，除了OpenGL，你还可以使用Canvas及Drawable对象做到同样的功能。OpenGL还提供了额外的功能，我们可以使用这些功能在三维空间中移动或者旋转物体，或者以其独有的方式创造绚丽的用户效果。

这节课将会学习OpengGL ES使用的另一种方式：使图形旋转。

##旋转图形
使用OpenGL使图形旋转起来还相对简单。在图形渲染器中，创建另一个转换矩阵(一个旋转矩阵)，然后将其整合进原来创建的投影与相机视图转换矩阵：
```java
private float[] mRotationMatrix = new float[16];
public void onDrawFrame(GL10 gl) {
    float[] scratch = new float[16];
    ...
    // Create a rotation transformation for the triangle
    long time = SystemClock.uptimeMillis() % 4000L;
    float angle = 0.090f * ((int) time);
    Matrix.setRotateM(mRotationMatrix, 0, angle, 0, 0, -1.0f);
    // Combine the rotation matrix with the projection and camera view
    // Note that the mMVPMatrix factor *must be first* in order
    // for the matrix multiplication product to be correct.
    Matrix.multiplyMM(scratch, 0, mMVPMatrix, 0, mRotationMatrix, 0);
    // Draw triangle
    mTriangle.draw(scratch);
}
```

如果采用了上面的代码而三角形没有旋转，那么应该检查是否注释了这行代码：GLSurfaceView.RENDERMODE_WHEN_DIRTY，这部分将会在下一小节中讨论。

##开启连续渲染
如果代码写到这里，需要确保你注释了一行代码：这行代码会设置渲染模式为只有在请求渲染的时候才会绘制的模式。另外OpenGL只会执行一次旋转，并会等待[requestRender()](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.html#requestRender())方法调用：
```java
public MyGLSurfaceView(Context context) {
    ...
    // Render the view only when there is a change in the drawing data.
    // To allow the triangle to rotate automatically, this line is commented out:
    //setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
}
```

除非不需要依靠用户的触发，这通常是一个很好的主意。准备好取消这行代码，因为下节课会适时的再调用一次。