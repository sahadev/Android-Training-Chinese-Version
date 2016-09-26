原文地址：[http://android.xsoftlab.net/training/camera/index.html](http://android.xsoftlab.net/training/camera/index.html)

#导言
在富媒体开始流行之前，整个世界是一个灰暗且平淡无奇的地方。还记得Gopher吗？我或许不记得了。自从APP成为用户生活的一部分之后，这便给他们提供了一种方式可以来存放他们生活的细节。使用设备上的相机，程序可以使用户扩大周围的视野或者见解，使以独特的化身，记录各个角落里的奇闻异事，或者只是简单的分享他们的境遇。

这节课会通过已有的相机应用以超级简单的方式快速过一下。在稍后的课程中会逐渐深入的学习如何直接控制相机硬件。

#简单拍照
这节课会展示如何使用已有的相机应用来拍照。

假设你正在实现一个众包的天气服务程序，这个服务可以使运行在设备上的客户端APP通过天空的混合照片生成全部的天气示意图(言下之意就是可以生成各种天气的天气图像)，而整合照片只是程序很小的一部分功能。你并不想在这里花费很多时间，也不想彻底的改造相机程序。幸运的是，大多数的安卓设备至少已经安装了一款相机应用。这节课就是学习如何利用这个应用为程序拍一张照片。

##请求相机权限
如果拍照是程序的一项必要需求，那么需要在Google Play上限制它的分类为拥有相机类。为了告知系统程序是基于相机的，需要在清单文件中添加 [<uses-feature>](http://android.xsoftlab.net/guide/topics/manifest/uses-feature-element.html)标签。
```xml
<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
```

如果程序需要使用，但是为了整个功能而不强制要求相机，那么可以设置android:required为false。这样做的话，Google Play会允许不带相机的设备下载你的程序。不过你有责任需要在运行时通过调用[hasSystemFeature(PackageManager.FEATURE_CAMERA)](http://android.xsoftlab.net/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String))方法检查设备上的相机是否可用。如果相机是不可用的，你应该禁用掉与相机相关的功能。

##通过相机APP拍照
Android通过授权的方式让其他程序通过调用一个Intent来描述你想要做的事情。这个过程包含了三块：Intent本身，一个启动外部Activity的调用，以及一些当焦点返回Activity时处理图像数据的代码。

下面代码的功能用于调用一个意图来拍摄照片：
```java
static final int REQUEST_IMAGE_CAPTURE = 1;
private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}
```

要注意，[startActivityForResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivityForResult(android.content.Intent,%20int))方法被一个调用[resolveActivity()](http://android.xsoftlab.net/reference/android/content/Intent.html#resolveActivity(android.content.pm.PackageManager))方法的条件所保护，这个方法返回了可以处理这个Intent的第一个activity组件。执行这项检查是非常重要的，因为如果你调用startActivityForResult()方法所使用的Intent没有APP可以处理的话，那么你的APP将会崩溃。所以只要结果不是null，那么就意味着可以安全使用这个Intent。

##获取缩略图像
如果简单的拍照功能并不是APP的主要功能，那么你可能想通过相机应用获得一张照片，并且利用这张照片做点什么事情。

Android的相机应用会将照片作为一个小的[Bitmap](http://android.xsoftlab.net/reference/android/graphics/Bitmap.html)对象打包进Intent中，然后通过[onActivityResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#onActivityResult(int,%20int,%20android.content.Intent))方法将该Intent返回。具体的Bitmap对象会附加在键"data"后。下面的代码段接收到这个图像，并将它展示到了一个[ImageView](http://android.xsoftlab.net/reference/android/widget/ImageView.html)中:
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
}
```

> **Note:** 从"data"中获得的缩略图像可能适合用于图标上面，但不适用于很大的图标。处理全尺寸的图像需要花费更多一点的工作。

##保存全尺寸照片
如果你提供了文件的保存路径的话，那么Android相机应用会将全尺寸的照片保存在这个地方。你必须提供文件的全路径，这个路径所指向的地方就是相机应用将要保存照片的地方。

通常情况下，用户通过相机应用所拍摄的任何照片都应该被保存在设备外部存储器上的一个公共文件夹中，这样就可以使所有的APP都可以访问。这个适用于存放共享照片的目录由[getExternalStoragePublicDirectory()](http://android.xsoftlab.net/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String))提供，并需要传递[DIRECTORY_PICTURES](http://android.xsoftlab.net/reference/android/os/Environment.html#DIRECTORY_PICTURES)参数。因为由这个方法所提供的分享目录适用于所有的APP，读取以及写入分别需要[READ_EXTERNAL_STORAGE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)和[WRITE_EXTERNAL_STORAGE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)权限。写入权限隐式地包含了读取权限，所有如果你需要写入到外部存储上的话，只需要请求一个权限就可以：
```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```

然而，如果你希望这些照片只是被保存在APP的私有目录中的话，你可以使用[getExternalFilesDir()](http://android.xsoftlab.net/reference/android/content/Context.html#getExternalFilesDir(java.lang.String))方法提供的目录来做替换方案。在Android 4.3之前，写入到这个目录需要[WRITE_EXTERNAL_STORAGE](http://android.xsoftlab.net/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)权限。从Android 4.4开始，这个权限不再被需要，因为这个目录不再对其它APP可见，所以这里声明的权限只对低版本的Android适用，并需要添加maxSdkVersion属性：
```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                     android:maxSdkVersion="18" />
    ...
</manifest>
```

> **Note:**当用户卸载了你的APP时，那么通过[getExternalFilesDir()](http://android.xsoftlab.net/reference/android/content/Context.html#getExternalFilesDir(java.lang.String))方法所提供的目录下的文件也会一并删除。

一旦你决定了文件存储的目录，需要创建一个防止冲突的文件名称。你可能还希望将路径保存到一个成员变量中，以便稍后再使用。这里在方法中有一个示例的解决办法，它会通过日期时间戳来对新照片返回一个唯一的文件名称：
```java
String mCurrentPhotoPath;
private File createImageFile() throws IOException {
    // Create an image file name
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    String imageFileName = "JPEG_" + timeStamp + "_";
    File storageDir = Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES);
    File image = File.createTempFile(
        imageFileName,  /* prefix */
        ".jpg",         /* suffix */
        storageDir      /* directory */
    );
    // Save a file: path for use with ACTION_VIEW intents
    mCurrentPhotoPath = "file:" + image.getAbsolutePath();
    return image;
}
```

随着这个方法可用来对照片创建文件，你现在可以创建并且调用Intent，就像这样：
```java
static final int REQUEST_TAKE_PHOTO = 1;
private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    // Ensure that there's a camera activity to handle the intent
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // Create the File where the photo should go
        File photoFile = null;
        try {
            photoFile = createImageFile();
        } catch (IOException ex) {
            // Error occurred while creating the File
            ...
        }
        // Continue only if the File was successfully created
        if (photoFile != null) {
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT,
                    Uri.fromFile(photoFile));
            startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
        }
    }
}
```

##添加照片到相册
当你通过一个意图创建了一张照片，你应该知道图像位于何处，因为在上面的代码中你告诉了照片保存的位置。对于其他人而言，可能使照片最简单的访问方式就是使照片对系统的媒体扫描器(Media Provider)是可访问的。

> **Note:** 如果照片保存的文件目录是由[getExternalFilesDir()](http://android.xsoftlab.net/reference/android/content/Context.html#getExternalFilesDir(java.lang.String))所提供的，那么，媒体扫描器是不能访问这些文件的，因为照片对于你的APP来说是私有的。

下面的示例方法演示了如何调用系统的扫描器来添加你的照片到媒体扫描器(Media Provider)的数据库中，使得这些照片可以被系统的相册应用或者其它APP可访问：
```java
private void galleryAddPic() {
    Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
    File f = new File(mCurrentPhotoPath);
    Uri contentUri = Uri.fromFile(f);
    mediaScanIntent.setData(contentUri);
    this.sendBroadcast(mediaScanIntent);
}
```

##解码缩放图像
在一定的内存中管理多个全尺寸的图像可能是很棘手的。如果你发现你的程序在展示了几张图片之后造成了内存溢出，那么可以通过将JPEG图像缩放到目标视图的尺寸大小的方式来显著的降低堆内存的使用量。
下面的示例方法演示了这一技术：
```java
private void setPic() {
    // Get the dimensions of the View
    int targetW = mImageView.getWidth();
    int targetH = mImageView.getHeight();
    // Get the dimensions of the bitmap
    BitmapFactory.Options bmOptions = new BitmapFactory.Options();
    bmOptions.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
    int photoW = bmOptions.outWidth;
    int photoH = bmOptions.outHeight;
    // Determine how much to scale down the image
    int scaleFactor = Math.min(photoW/targetW, photoH/targetH);
    // Decode the image file into a Bitmap sized to fill the View
    bmOptions.inJustDecodeBounds = false;
    bmOptions.inSampleSize = scaleFactor;
    bmOptions.inPurgeable = true;
    Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
    mImageView.setImageBitmap(bitmap);
}
```