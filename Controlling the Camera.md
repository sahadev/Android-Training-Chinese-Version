原文地址：[http://android.xsoftlab.net/training/camera/cameradirect.html](http://android.xsoftlab.net/training/camera/cameradirect.html)

在这节课，我们会讨论如何使用Android框架API来直接控制相机硬件。

直接控制设备的相机拍照或者摄像的代码远比通过其他相机应用来完成要多得多。然而，如果你想构建一个专业的相机应用或者在APP的UI中完全集成相机的话，这节课展示了如何去做。

##开启相机对象
直接控制相机的第一步就是获得[Camera](http://android.xsoftlab.net/reference/android/hardware/Camera.html)对象的实例。和Android自身的相机应用相同，推荐访问相机的方式就是在独立的线程打开[Camera](http://android.xsoftlab.net/reference/android/hardware/Camera.html)，这种方式是应对阻塞UI线程的一个好的解决方法。在更加基础化的实现当中，开启相机这一步操作可以推迟到onResume()方法中执行，这样可以促使代码重用并且保持简单的控制流。

如果相机已经正在被其它应用所使用，那么调用[Camera.open()](http://android.xsoftlab.net/reference/android/hardware/Camera.html#open())方法会抛出一个异常，所以我们需要使用try控制块包裹住它：
```java
private boolean safeCameraOpen(int id) {
    boolean qOpened = false;
  
    try {
        releaseCameraAndPreview();
        mCamera = Camera.open(id);
        qOpened = (mCamera != null);
    } catch (Exception e) {
        Log.e(getString(R.string.app_name), "failed to open Camera");
        e.printStackTrace();
    }
    return qOpened;    
}
private void releaseCameraAndPreview() {
    mPreview.setCamera(null);
    if (mCamera != null) {
        mCamera.release();
        mCamera = null;
    }
}
```

从API 9开始，相机框架支持多个相机。如果你使用的是过去的API，然后调用了没有参数的open()方法，那么你会获得后置面板的相机。

##创建相机预览
拍照通常需要可以使用户能看到目标的预览图。你可以使用[SurfaceView](http://android.xsoftlab.net/reference/android/view/SurfaceView.html)来绘制相机传感器捕获到的图像。

###预览类
为了可以显示预览，你需要预览类。预览需要一个android.view.SurfaceHolder.Callback接口的实现，它被用来从相机硬件给应用传递图像数据。
```java
class Preview extends ViewGroup implements SurfaceHolder.Callback {
    SurfaceView mSurfaceView;
    SurfaceHolder mHolder;
    Preview(Context context) {
        super(context);
        mSurfaceView = new SurfaceView(context);
        addView(mSurfaceView);
        // Install a SurfaceHolder.Callback so we get notified when the
        // underlying surface is created and destroyed.
        mHolder = mSurfaceView.getHolder();
        mHolder.addCallback(this);
        mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
    }
...
}
```

在开始预览之前，必须将预览对象传递给Camera对象，就像下面部分展示的那样。

###设置并开始预览
相机实例的创建于相关预览对象创建必须是以指定顺序进行的，从相机对象开始。在下面的代码中，实例化相机对象的过程被封装起来了，所以[Camera.startPreview()](http://android.xsoftlab.net/reference/android/hardware/Camera.html#startPreview())是可以通过setCamera()调用的，每当用户做了什么事情使相机发生了改变。预览也必须在预览类的surfaceChanged()回调方法重新启动。
```java
public void setCamera(Camera camera) {
    if (mCamera == camera) { return; }
    
    stopPreviewAndFreeCamera();
    
    mCamera = camera;
    
    if (mCamera != null) {
        List<Size> localSizes = mCamera.getParameters().getSupportedPreviewSizes();
        mSupportedPreviewSizes = localSizes;
        requestLayout();
      
        try {
            mCamera.setPreviewDisplay(mHolder);
        } catch (IOException e) {
            e.printStackTrace();
        }
      
        // Important: Call startPreview() to start updating the preview
        // surface. Preview must be started before you can take a picture.
        mCamera.startPreview();
    }
}
```

##修改相机设置
相机设置可以改变相机拍照的方式，从缩放等级到曝光补偿等等。下面的示例只是更改了预览的大小;请查看相机应用的源代码获取更多可能。
```java
public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
    // Now that the size is known, set up the camera parameters and begin
    // the preview.
    Camera.Parameters parameters = mCamera.getParameters();
    parameters.setPreviewSize(mPreviewSize.width, mPreviewSize.height);
    requestLayout();
    mCamera.setParameters(parameters);
    // Important: Call startPreview() to start updating the preview surface.
    // Preview must be started before you can take a picture.
    mCamera.startPreview();
}
```

##设置预览方向
大多数的相机应用将展示锁定在了水平方向，因为这是相机传感器的自然方向。这个设置并不能阻止你在垂直方向上拍摄，因为相机的方向会被记录到EXIF的头部。[setCameraDisplayOrientation()](http://android.xsoftlab.net/reference/android/hardware/Camera.html#setDisplayOrientation(int))方法允许你改变如何展示预览，而不受图像记录方向的影响。然而，在API14之前，在改变方向之前必须停止预览，然后在重新启动它。

##拍照
一旦预览启动后，可以使用[Camera.takePicture()](http://android.xsoftlab.net/reference/android/hardware/Camera.html#takePicture(android.hardware.Camera.ShutterCallback,%20android.hardware.Camera.PictureCallback,%20android.hardware.Camera.PictureCallback))方法来拍一张照片。你可以创建[Camera.PictureCallback](http://android.xsoftlab.net/reference/android/hardware/Camera.PictureCallback.html)对象和[Camera.ShutterCallback](http://android.xsoftlab.net/reference/android/hardware/Camera.ShutterCallback.html)对象然后将它们传递给[Camera.takePicture()](http://android.xsoftlab.net/reference/android/hardware/Camera.html#takePicture(android.hardware.Camera.ShutterCallback,%20android.hardware.Camera.PictureCallback,%20android.hardware.Camera.PictureCallback))方法。

##重启预览
在拍了一张照片之后，你必须在用户拍另一张照片之前重新启动预览。在这个例子中，通过重写快门按钮来完成重启。
```java
@Override
public void onClick(View v) {
    switch(mPreviewState) {
    case K_STATE_FROZEN:
        mCamera.startPreview();
        mPreviewState = K_STATE_PREVIEW;
        break;
    default:
        mCamera.takePicture( null, rawCallback, null);
        mPreviewState = K_STATE_BUSY;
    } // switch
    shutterBtnConfig();
}
```

##停止预览并且释放相机
一旦你的程序不再需要使用相机，这时就需要执行清理工作。尤其是你需要释放相机对象，否则会使其它程序面临崩溃的风险，包括你自己程序中新的实例。

何时应该停止预览并释放相机呢？好吧，当预览界面被销毁的时候便是停止预览并释放相机的最佳时机，就像下面Preview类中显示的那样：
```java
public void surfaceDestroyed(SurfaceHolder holder) {
    // Surface will be destroyed when we return, so stop the preview.
    if (mCamera != null) {
        // Call stopPreview() to stop updating the preview surface.
        mCamera.stopPreview();
    }
}
/**
 * When this function returns, mCamera will be null.
 */
private void stopPreviewAndFreeCamera() {
    if (mCamera != null) {
        // Call stopPreview() to stop updating the preview surface.
        mCamera.stopPreview();
    
        // Important: Call release() to release the camera for use by other
        // applications. Applications should release the camera immediately
        // during onPause() and re-open() it during onResume()).
        mCamera.release();
    
        mCamera = null;
    }
}
```

在上面的课程中，这段程序也是setCamera()方法的一部分，所以实例化一个相机总是从停止这段预览开始的。