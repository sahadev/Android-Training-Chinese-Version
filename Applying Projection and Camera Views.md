原文地址：[http://android.xsoftlab.net/training/graphics/opengl/projection.html##transform](http://android.xsoftlab.net/training/graphics/opengl/projection.html##transform)

在OpenGL ES环境中，投影相机View可以将所绘制的图形模拟成现实中所看到的物理性状。这种物理模拟是通过改变对象的数字坐标实现的：

- 投影 - 这基于GLSurfaceView的高宽的坐标转换而实现。如果不采用这种计算，所绘制的对象就会由于窗口的不平等比例所曲解。投影转换只有在OpenGL视图的比例被确定或渲染器的onSurfaceChanged()方法发生改变后才会被计算。有关更多OpenGL的坐标映射关系，请参见[Mapping Coordinates for Drawn Objects](http://android.xsoftlab.net/guide/topics/graphics/opengl.html#coordinate-mapping)。
- 相机视图 - 这基于虚拟相机的位置转变而实现。要着重注意OpenGL并没有定义真实的相机对象，而是提供了一种通过转换图形对象来模拟相机的方法。一个相机视图转换只有在GLSurfaceView被确定的时候才会开始计算，或者基于用户的行为或者应用的功能而动态的发生改变。

这节课描述了如何创建一个投影相机视图，并将其应用到图形的绘制中。

##定义投影
投影转换的数据计算位于[GLSurfaceView.Renderer](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.Renderer.html)的[onSurfaceChanged()](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceChanged(javax.microedition.khronos.opengles.GL10,%20int,%20int))方法。下面的代码先取得了[GLSurfaceView](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.html)的高度与宽度，然后使用[Matrix.frustumM()](http://android.xsoftlab.net/reference/android/opengl/Matrix.html#frustumM(float[],%20int,%20float,%20float,%20float,%20float,%20float,%20float))方法将数据填充到投影转换的[Matrix](http://android.xsoftlab.net/reference/android/opengl/Matrix.html)中。
```java
// mMVPMatrix is an abbreviation for "Model View Projection Matrix"
private final float[] mMVPMatrix = new float[16];
private final float[] mProjectionMatrix = new float[16];
private final float[] mViewMatrix = new float[16];
@Override
public void onSurfaceChanged(GL10 unused, int width, int height) {
    GLES20.glViewport(0, 0, width, height);
    float ratio = (float) width / height;
    // this projection matrix is applied to object coordinates
    // in the onDrawFrame() method
    Matrix.frustumM(mProjectionMatrix, 0, -ratio, ratio, -1, 1, 3, 7);
}
```

这些代码会填充投影矩阵:mProjectionMatrix,它可以被用来