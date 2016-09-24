原文地址：[http://android.xsoftlab.net/training/graphics/opengl/index.html](http://android.xsoftlab.net/training/graphics/opengl/index.html)

#引言
Android framework层为创建绚丽的功能性UI提供了大量的标准工具。然而，如果想要以更多方式来控制屏幕的绘制，或者在三维图形中绘制，那么就需要使用其它工具了。Android framework所提供的OpenGL ES API为我们提供了一系列的工具，这些工具可以用来显示一些高端大气、天马行空的图形，只要你能想得到，那么它就可以做得到。此外，它还得益于很多设备所提供的GPU加速功能。

这节课会讨论OpenGL的开发基础：包括设置环境、绘制对象、移动绘制元素及响应触摸事件。

在接下来的示例代码中使用了OpenGL ES 2.0 API,该版本在当前的Android设备上推荐使用。有关更多OpenGL ES的版本信息，请参见[OpenGL](http://android.xsoftlab.net/guide/topics/graphics/opengl.html#choosing-version)开发指南。

> **Note:** 注意不要将OpenGL ES 1.x API与OpenGL ES 2.0 API相混淆！这两个版本的API之间不可交换使用，如果要使用的话，那么只有一个结果，就是悲剧。

#构建OpenGL ES的相关环境
为了能在Android应用中使用OpenGL ES，必须给OpenGL ES要绘制的图形区域创建一个View容器。其中一种实现方法就是实现[GLSurfaceView](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.html)及[GLSurfaceView.Renderer](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.Renderer.html)。其中GLSurfaceView是在OpenGL中绘制图形的View容器。GLSurfaceView.Renderer用于控制应该在刚才的View中绘制什么。有关更多这些类的信息，请参见[OpenGL ES](http://android.xsoftlab.net/guide/topics/graphics/opengl.html)开发指南。

GLSurfaceView是将OpenGL ES整合到应用中的唯一途径。对于全屏或者接近全屏的图形View，GLSurfaceView是最合适的选择。开发者如果需要将OpenGL ES整合到一小块区域上，应该使用[TextureView](http://android.xsoftlab.net/reference/android/view/TextureView.html)。除此之外还可以使用[SurfaceView](http://android.xsoftlab.net/reference/android/view/SurfaceView.html)，不过这可能需要相当多的代码才能实现。

这节课将会展示以最省的代码来实现GLSurfaceView及GLSurfaceView.Renderer。

##在清单文件中声明OpenGL ES的使用
为了可以使用OpenGL ES 2.0 API，需要在清单文件中添加如下声明：
```xml
<uses-feature android:glEsVersion="0x00020000" android:required="true" />
```

如果应用用到了纹理压缩的功能，那么还需要声明应用所使用到的压缩格式，这样的话，应用只能被安装在支持的设备上。
```xml
<supports-gl-texture android:name="GL_OES_compressed_ETC1_RGB8_texture" />
<supports-gl-texture android:name="GL_OES_compressed_paletted_texture" />
```

有关更多纹理压缩格式的相关信息，请参见OpenGL开发指南。

##为OpenGL ES创建一个Activity
与普通的应用相同，OpenGL ES也同样需要用户界面。主要的不同在于普通的应用只需要将布局放入Activity就可以。在使用OpenGL ES的应用中，除了使用普通的TextView这类基本控件之外，还需要添加GLSurfaceView。

下面这段代码展示了使用GLSurfaceView的最基本实现:
```java
public class OpenGLES20Activity extends Activity {
    private GLSurfaceView mGLView;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Create a GLSurfaceView instance and set it
        // as the ContentView for this Activity.
        mGLView = new MyGLSurfaceView(this);
        setContentView(mGLView);
    }
}
```

> **Note:** OpenGL ES 2.0需要在Android 2.2及以上的版本上才能运行，请确保应用程序的最低版本在此之上。

##构建GLSurfaceView对象
GLSurfaceView是一块特殊的区域，它可以用户绘制OpenGL ES图形。它本身不需要做太多事情。实际上，图形的绘制是由GLSurfaceView.Renderer来控制的。事实上，你可能想要试着不去继承这个类，而是直接创建一个原生的GLSurfaceView实例，不要这么去做。你需要继承这个类来捕获触摸事件，相关信息在[Responding to Touch Events](http://android.xsoftlab.net/training/graphics/opengl/environment.html#touch.html)课程中有涉及到。

本质上GLSurfaceView的代码是很少的，所以为了快速去实现它，常见的做法是在使用这个对象的Activity中创建一个内部类：
```java
class MyGLSurfaceView extends GLSurfaceView {
    private final MyGLRenderer mRenderer;
    public MyGLSurfaceView(Context context){
        super(context);
        // Create an OpenGL ES 2.0 context
        setEGLContextClientVersion(2);
        mRenderer = new MyGLRenderer();
        // Set the Renderer for drawing on the GLSurfaceView
        setRenderer(mRenderer);
    }
}
```

对GLSurfaceView的其它附加选项就是设置其渲染模式：
```java
// Render the view only when there is a change in the drawing data
setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
```

这个选项阻止了GLSurfaceView框架的重新绘制，直到[requestRender()](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.html#requestRender())方法被调用。这可以提高应用的效率。

##构建渲染器类
类[GLSurfaceView.Renderer](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.Renderer.html)是真正有意思的地方。这个类可以控制在GLSurfaceView上所绘制的事物。它内部有3个方法，这3个方法由Android系统调用，用于计算如何在GLSurfaceView上进行绘制：

- [onSurfaceCreated()](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceCreated(javax.microedition.khronos.opengles.GL10, javax.microedition.khronos.egl.EGLConfig))在设置OpenGL ES环境的时候调用一次。
- [onDrawFrame()](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.Renderer.html#onDrawFrame(javax.microedition.khronos.opengles.GL10)) view的每次绘制都会调用。
- [onSurfaceChanged()](http://android.xsoftlab.net/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceChanged(javax.microedition.khronos.opengles.GL10, int, int)) 在view的结构发生改变的时候进行调用，比如设备的屏幕方向发生了变化。

下面是OpenGL ES渲染器的最基本实现，这里只是在GLSurfaceView简单绘制了一个黑色的背景：
```java
public class MyGLRenderer implements GLSurfaceView.Renderer {
    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        // Set the background frame color
        GLES20.glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    }
    public void onDrawFrame(GL10 unused) {
        // Redraw background color
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
    }
    public void onSurfaceChanged(GL10 unused, int width, int height) {
        GLES20.glViewport(0, 0, width, height);
    }
}
```

上面就是要做的所有工作了。上面的代码使用OpenGL绘制了一个黑色的背景。虽然这些代码并没有做什么有意思的事情，但是通过创建这些类，你就可以为通过OpenGL绘制图形打下了基础。

> **Note:** 你可能会怀疑，在使用OpengGL ES 2.0 API时，为什么这些方法都会有个名叫[GL10](http://android.xsoftlab.net/reference/javax/microedition/khronos/opengles/GL10.html)的参数。这些在2.0 API中重复使用到的签名方法是为了保持Android framework的代码简便。

如果你对OpenGL ES API很熟悉，你现在就可以设置OpenGL ES的环境并着手绘制图形了。无论如何，如果你想获取更多有关OpenGL的入门帮助，可以查看下节上部分的一些小提示。