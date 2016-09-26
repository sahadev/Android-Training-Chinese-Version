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

这些代码会填充投影矩阵:mProjectionMatrix,它可以被用来与相机视图转换整合。
> **Note:** 仅仅将投影转换应用到要绘制的物体上，一般来说是没什么意义的。通常情况下，还必须将相机视图转换也一并应用。这样才能使物体呈现在屏幕上。

##定义相机视图
完成相机视图的转换也是图像处理过程的一部分。在下面的代码中，相机视图转换通过[Matrix.setLookAtM()](http://android.xsoftlab.net/reference/android/opengl/Matrix.html#setLookAtM(float[],%20int,%20float,%20float,%20float,%20float,%20float,%20float,%20float,%20float,%20float))方法进行计算，然后将结果整合进原先所计算好的矩阵中。最后被整合完毕的矩阵接着被用来绘制图形。
```java
@Override
public void onDrawFrame(GL10 unused) {
    ...
    // Set the camera position (View matrix)
    Matrix.setLookAtM(mViewMatrix, 0, 0, 0, -3, 0f, 0f, 0f, 0f, 1.0f, 0.0f);
    // Calculate the projection and view transformation
    Matrix.multiplyMM(mMVPMatrix, 0, mProjectionMatrix, 0, mViewMatrix, 0);
    // Draw shape
    mTriangle.draw(mMVPMatrix);
}
```

##应用投影转换及相机转换
为了可以使用上面所整合完毕的矩阵，首先需要将该矩阵变量添加到上节课程所定义的顶点渲染器中：
```java
public class Triangle {
    private final String vertexShaderCode =
        // This matrix member variable provides a hook to manipulate
        // the coordinates of the objects that use this vertex shader
        "uniform mat4 uMVPMatrix;" +
        "attribute vec4 vPosition;" +
        "void main() {" +
        // the matrix must be included as a modifier of gl_Position
        // Note that the uMVPMatrix factor *must be first* in order
        // for the matrix multiplication product to be correct.
        "  gl_Position = uMVPMatrix * vPosition;" +
        "}";
    // Use to access and set the view transformation
    private int mMVPMatrixHandle;
    ...
}
```

接下来，修改绘制物体对象的draw()方法，以便可以接收整合后的矩阵，然后将其应用到图形中：
```java
public void draw(float[] mvpMatrix) { // pass in the calculated transformation matrix
    ...
    // get handle to shape's transformation matrix
    mMVPMatrixHandle = GLES20.glGetUniformLocation(mProgram, "uMVPMatrix");
    // Pass the projection and view transformation to the shader
    GLES20.glUniformMatrix4fv(mMVPMatrixHandle, 1, false, mvpMatrix, 0);
    // Draw the triangle
    GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);
    // Disable vertex array
    GLES20.glDisableVertexAttribArray(mPositionHandle);
}
```

如果有正确的计算结果以及正确的使用了整合后的矩阵，那么图形会以正确的比例被绘制出来，最后看起来的效果应该是这样子的：
![](http://android.xsoftlab.net/images/opengl/ogl-triangle-projected.png)

现在程序就可以正常显示比例正常的图形了，是时候为图形添加动作了。