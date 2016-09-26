原文地址：[http://android.xsoftlab.net/training/camera/videobasics.html](http://android.xsoftlab.net/training/camera/videobasics.html)

这节课解释了如何通过已有的相机应用拍摄视频。

假设你的程序含有摄像功能，但是它只是程序很小的一部分功能，你并不想在这么小的功能上花费很大的精力。幸运的是，大多数的安卓设备已经内置了一款相机应用，并且它可以拍摄视频。这节课将会展示如何拍摄视频。

##请求相机权限
为了告知系统程序是基于相机的，需要在清单文件中添加 [<uses-feature>](http://android.xsoftlab.net/guide/topics/manifest/uses-feature-element.html)标签。
```xml
<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
```

如果程序需要使用，但是为了整个功能而不强制要求相机，那么可以设置android:required为false。这样做的话，Google Play会允许不带相机的设备下载你的程序。不过你有责任需要在运行时通过调用[hasSystemFeature(PackageManager.FEATURE_CAMERA)](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String))方法检查设备上的相机是否可用。如果相机是不可用的，你应该禁用掉与相机相关的功能。

##通过相机APP摄像
Android通过授权的方式让其他程序通过调用一个Intent来描述你想要做的事情。这个过程包含了三块：Intent本身，一个启动外部Activity的调用，以及一些当焦点返回Activity时处理图像数据的代码。

下面代码的功能用于调用一个意图来捕获视频：
```java
static final int REQUEST_VIDEO_CAPTURE = 1;
private void dispatchTakeVideoIntent() {
    Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
    if (takeVideoIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takeVideoIntent, REQUEST_VIDEO_CAPTURE);
    }
}
```

要注意，[startActivityForResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivityForResult(android.content.Intent,%20int))方法被一个调用[resolveActivity()](http://android.xsoftlab.net/reference/android/content/Intent.html#resolveActivity(android.content.pm.PackageManager))方法的条件所保护，这个方法返回了可以处理这个Intent的第一个activity组件。执行这项检查是非常重要的，因为如果你调用startActivityForResult()方法所使用的Intent没有APP可以处理的话，那么你的APP将会崩溃。所以只要结果不是null，那么就意味着可以安全使用这个Intent。

##查看视频
Android的相机应用会通过[onActivityResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#onActivityResult(int,%20int,%20android.content.Intent))方法将视频返回，视频位于onActivityResult()方法的回调参数Intent中的Uri所指向的位置。下面的代码展示了接收这个视频并且在VideoView中播放它。
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_VIDEO_CAPTURE && resultCode == RESULT_OK) {
        Uri videoUri = intent.getData();
        mVideoView.setVideoURI(videoUri);
    }
}
```
