原文地址：[http://android.xsoftlab.net/training/graphics/opengl/draw.html](http://android.xsoftlab.net/training/graphics/opengl/draw.html)

在定义了图形之后，你接下来需要做的就是将它绘制到屏幕上。不过使用OpenGL ES 2.0 API来绘制这个图形所需要的代码量可能要比想象中的多一些，这是因为API为图形渲染管道提供了大量的控制细节。

这节课会展示如何绘制上节课所定义的图形。

##初始化图形
在开始任何绘制之前，你必须初始化并加载这个图形。除非是在执行的过程中图形的结构发生了改变。这个时候你应该在渲染器的[onSurfaceCreated()](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceCreated(javax.microedition.khronos.opengles.GL10,%20javax.microedition.khronos.egl.EGLConfig))方法中去初始化它们，这样可以使内存和进程的效率提升。
```java
public class MyGLRenderer implements GLSurfaceView.Renderer {
    ...
    private Triangle mTriangle;
    private Square   mSquare;
    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        ...
        // initialize a triangle
        mTriangle = new Triangle();
        // initialize a square
        mSquare = new Square();
    }
    ...
}
```

##绘制图形
绘制自定义图形需要大量的代码，因为你必须给图形渲染管道提供大量的渲染细节。尤其是下面这些必须定义：

- **Vertex Shader** - 图形顶点的渲染.
- **Fragment Shader** - 图形表面的颜色或纹理的渲染。
- **Program** - 一个含有多个渲染器的OpenGL ES对象，可以用它来绘制一个或者多个图形。

你需要至少一个顶点渲染器来绘制图形，并需要一个表面渲染器来为图形着色。这些渲染器首先必须是可执行的，然后才能将其添加到OpenGL ES程序中，这时才能被用来绘制图形。下面定义了一个最基本的可以用来绘制图形的渲染器：
```java
public class Triangle {
    private final String vertexShaderCode =
        "attribute vec4 vPosition;" +
        "void main() {" +
        "  gl_Position = vPosition;" +
        "}";
    private final String fragmentShaderCode =
        "precision mediump float;" +
        "uniform vec4 vColor;" +
        "void main() {" +
        "  gl_FragColor = vColor;" +
        "}";
    ...
}
```

渲染器包含了OpenGL渲染语言代码，这些代码必须先在OpenGL ES环境中编译通过。为了编译这些代码，需要在渲染器类中创建一个功能方法：
```java
public static int loadShader(int type, String shaderCode){
    // create a vertex shader type (GLES20.GL_VERTEX_SHADER)
    // or a fragment shader type (GLES20.GL_FRAGMENT_SHADER)
    int shader = GLES20.glCreateShader(type);
    // add the source code to the shader and compile it
    GLES20.glShaderSource(shader, shaderCode);
    GLES20.glCompileShader(shader);
    return shader;
}
```

为了可以绘制图形，必须先编译这些渲染器代码，然后将其添加到OpenGL程序中，然后再链接到程序中。需要将这些工作放入绘制对象的构造方法中，所以这些工作只用做一次。

> **Note:** 