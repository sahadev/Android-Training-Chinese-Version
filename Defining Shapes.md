原文地址：[http://android.xsoftlab.net/training/graphics/opengl/shapes.html](http://android.xsoftlab.net/training/graphics/opengl/shapes.html)

使用OpenGL绘制图形的第一步就是要定义一个图形。如果不清楚OpenGL如何绘制自定义图形的相关基础知识时，那么使用OpenGL一定要仔细。

这节课将会简单讲述OpenGl ES的坐标系统，及定义图形的基础。本章中以三角形、正方形做说明。

##定义三角形
OpenGL ES允许用户使用三维空间定义被绘制的对象。所以，在绘制三角形之前，必须先定义它的坐标。在OpenGL中，最典型的方式就是以浮点型数字定义一个顶点坐标数组。为了提高效率，应该将这些数据写入到一个[ByteBuffer](http://android.xsoftlab.net/reference/java/nio/ByteBuffer.html)中，它随后会被送入OpenGL ES图形管道中，等待处理。
```java
public class Triangle {
    private FloatBuffer vertexBuffer;
    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float triangleCoords[] = {   // in counterclockwise order:
             0.0f,  0.622008459f, 0.0f, // top
            -0.5f, -0.311004243f, 0.0f, // bottom left
             0.5f, -0.311004243f, 0.0f  // bottom right
    };
    // Set color with red, green, blue and alpha (opacity) values
    float color[] = { 0.63671875f, 0.76953125f, 0.22265625f, 1.0f };
    public Triangle() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
                // (number of coordinate values * 4 bytes per float)
                triangleCoords.length * 4);
        // use the device hardware's native byte order
        bb.order(ByteOrder.nativeOrder());
        // create a floating point buffer from the ByteBuffer
        vertexBuffer = bb.asFloatBuffer();
        // add the coordinates to the FloatBuffer
        vertexBuffer.put(triangleCoords);
        // set the buffer to read the first coordinate
        vertexBuffer.position(0);
    }
}
```

默认情况下，OpenGL ES的坐标系统将[0,0,0]\(X,Y,Z\)点作为[GLSurfaceView](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.html)的中心。[1,1,0]点是框架的右上角，[-1,-1,0]点是框架的左下角。有关该坐标系统的演示信息，请参见[OpenGL ES](http://android.xsoftlab.net/guide/topics/graphics/opengl.html#coordinate-mapping)开发指南。

注意这里定义图形坐标点是以逆时针方向定义的。知道这个绘制顺序是很重要的，因为它决定了图形的哪一面应该是朝着用户的，哪一面应该是被绘制的，以及朝着用户的另一面的绘制信息，它还可以选择不使用OpenGL绘制朝着用户的这一面等功能。有关更多转向及选择绘制的相关信息，请参见[OpenGL ES](http://android.xsoftlab.net/guide/topics/graphics/opengl.html#coordinate-mapping)开发指南。

##定义正方形
定义一个三角形是很容易的，但是定义一些稍微比较复杂一点的图形会怎么样？比如定义一个正方形？这里提供了几种实现方式，但是典型的方式就是将两个三角形绘制到一起，然后形成一个正方形：

![](http://android.xsoftlab.net/images/opengl/ccw-square.png)

再提醒一次，定义图形的时候应该以逆时针方向定义这两个三角形的顶点。为了避免两个三角形都重复定义两个坐标点，应该使用绘制列表来告诉OpenGL ES绘制管道如何绘制这些顶点。下面是这个图形的代码：
```java
public class Square {
    private FloatBuffer vertexBuffer;
    private ShortBuffer drawListBuffer;
    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float squareCoords[] = {
            -0.5f,  0.5f, 0.0f,   // top left
            -0.5f, -0.5f, 0.0f,   // bottom left
             0.5f, -0.5f, 0.0f,   // bottom right
             0.5f,  0.5f, 0.0f }; // top right
    private short drawOrder[] = { 0, 1, 2, 0, 2, 3 }; // order to draw vertices
    public Square() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 4 bytes per float)
                squareCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        vertexBuffer = bb.asFloatBuffer();
        vertexBuffer.put(squareCoords);
        vertexBuffer.position(0);
        // initialize byte buffer for the draw list
        ByteBuffer dlb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 2 bytes per short)
                drawOrder.length * 2);
        dlb.order(ByteOrder.nativeOrder());
        drawListBuffer = dlb.asShortBuffer();
        drawListBuffer.put(drawOrder);
        drawListBuffer.position(0);
    }
}
```
这个示例展现了如何创建复合型图形的方法。一般情况下，会以三角形堆叠的方式来绘制对象。在下节课中，你会学习到如何在屏幕上绘制这些图形。