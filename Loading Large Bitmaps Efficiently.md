原文地址：[http://android.xsoftlab.net/training/displaying-bitmaps/index.html](http://android.xsoftlab.net/training/displaying-bitmaps/index.html)

#引言
学习如何使用一种常规的手段来处理及加载[Bitmap](http://android.xsoftlab.net/reference/android/graphics/Bitmap.html)对象，这种方式除了使用户界面是可响应的，还会避免超出内存的限制。如果你不小心点的话，位图会迅速的将那些可怜的内存消耗殆尽，并会导致程序崩溃，因为这会产生一种可怕的异常：
```xml
java.lang.OutofMemoryError: bitmap size exceeds VM budget.
```

这里列举出了一些原因来说明为什么加载位图对于Android程序来说是非常棘手的：

- 移动设备通常含有有限的资源。Android设备对于单个程序只有少量的16MB可用内存。虚拟机兼容性(Virtual Machine Compatibility)针对于各种的屏幕尺寸和密度给出了最低限度的程序内存要求。程序应该在极小的内存空间下充分利用内存空间。无论如何要记住一点，很多设备配备了更高的限制。
- 位图通常会消耗掉不少内存，尤其是丰富的图片，就像照片这样的。举个例子，[Galaxy Nexus](http://www.android.com/devices/detail/galaxy-nexus)上的相机拍的照片会达到2592x1936个像素(五百万像素)。如果位图配置使用的是[ARGB_8888](http://android.xsoftlab.net/reference/android/graphics/Bitmap.Config.html)(这在Android 2.3以前是默认的)，那么加载这张照片到内存中就需要花费掉19MB的内存(2592\*1936\*4个字节),这会立即耗尽某些设备上的所有内存。
- Android的APP界面有时会很频繁的请求一些图片来加载。有些组件比如[ListView](http://android.xsoftlab.net/reference/android/widget/ListView.html), [GridView](http://android.xsoftlab.net/reference/android/widget/GridView.html)及[ViewPager](http://android.xsoftlab.net/reference/android/support/v4/view/ViewPager.html),它们有个共同的特性就是需要同时在屏幕上加载多个位图并会在屏幕之外的地方加载以便在手指滑动的时候显示出来。

#有效加载大图
图片会有各种形状和大小。在很多情况下它们会比用户界面上所要求的尺寸要大。举个例子，系统的相册应用所展示的用相机拍摄的照片的分辨率通常要比屏幕的密度要高。

鉴于在有限的内存中工作，理想上只用加载低分辨率的版本就可以。低分辨率的版本应该匹配到展示这张图片的控件大小。图片的更高分辨率不会在视觉上有更佳的效果，但是这仍然会消耗宝贵的内存空间，由于额外的动态扩展，这会招致额外的性能开销。

这节课会讨论将大位图进行二次采样并将采样后的小版本加载到内存中的过程。这个过程并不会超出应用的内存限制。

##读取位图的尺寸及类型
类[BitmapFactory](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.html)提供了若干个解码方法([decodeByteArray()](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.html#decodeByteArray(byte[],%20int,%20int,%20android.graphics.BitmapFactory.Options)), [decodeFile()](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.html#decodeFile(java.lang.String,%20android.graphics.BitmapFactory.Options)), [decodeResource()](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.html#decodeResource(android.content.res.Resources,%20int,%20android.graphics.BitmapFactory.Options)), etc.)根据不同的资源来创建位图[Bitmap](http://android.xsoftlab.net/reference/android/graphics/Bitmap.html)。选择更加适合的解码方法取决于图片的数据资源。这些方法会在构造位图时尝试向内存申请空间，所以会轻易的造成OutOfMemory异常。每个解码方法都有一个附属特征，这个特征可以使你通过BitmapFactory.Options类来指定解码选项。设置inJustDecodeBounds属性为true可以避免在解码时向内存申请空间，这会返回一个空的位图，但是[outWidth](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.Options.html#outWidth)、[outHeight](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.Options.html#outHeight)和[outMimeType](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.Options.html#outMimeType)这些设置除外。这项技术可以使你在构造位图(申请内存)之前提前读取图像数据的尺寸及类型。
```java
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
int imageHeight = options.outHeight;
int imageWidth = options.outWidth;
String imageType = options.outMimeType;
```

为了避免java.lang.OutOfMemory异常，需要在解码图片之前检查图片的尺寸，除非你对这些图像数据的尺寸绝对的信任，并且该尺寸对可用内存非常适用。

##加载等比缩小的版本到内存
那么现在图片的尺寸是知道了，这尺寸可以被用来决定：是否全尺寸的图像应该被加载到内存中还是应该有个二次采样的版本加载到内存中。这里有一些因素需要考虑：

- 往内存中加载全尺寸的图像应该估算要使用的内存大小。
- 要加载的图片所需要的内存数量需要给应用预留一定的内存空间，不要消耗完全。
- [ImageView](http://android.xsoftlab.net/reference/android/widget/ImageView.html)或者UI组件的尺寸是图像将要加载的尺寸。
- 当前设备的屏幕尺寸与密度。

举个例子，加载一个1024*768像素的图片到内存中是没有价值的，如果这个图片最终被显示为一个128x96像素的缩略图的话。

为了告诉解码器需要进行二次采样，以便加载一个小版本的图像到内存中，需要设置BitmapFactory.Options对象的inSampleSize属性为true。举个例子，一张图片的分辨率为2048x1536，需要通过inSampleSize解码为4分之一大小的位图，大概是512x384。加载这样的图像只需要花费0.75MB内存，而全尺寸的图像则需要花费12MB的内存(假设位图的配置为[ARGB_8888](http://android.xsoftlab.net/reference/android/graphics/Bitmap.Config.html))。这里有一个方法可以来计算一个样本容量值，这个值是2的幂次方值并基于原图像的高度值与宽度值进行计算。
```java
public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;
    if (height > reqHeight || width > reqWidth) {
        final int halfHeight = height / 2;
        final int halfWidth = width / 2;
        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) > reqHeight
                && (halfWidth / inSampleSize) > reqWidth) {
            inSampleSize *= 2;
        }
    }
    return inSampleSize;
}
```

> **Note:** 最终计算后的值是一个2的幂次方值是因为解码器需要通过舍入来获得一个最终值，这个值与2的幂次方最为接近，依据[inSampleSize](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.Options.html#inSampleSize)文档。

为了使用这个方法，第一步需要将inJustDecodeBounds设置为true，然后将options交给BitmapFactory使用，然后再次使用一个新的inSampleSize和inJustDecodeBounds设置为false来再次使用：
```java
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {
    // First decode with inJustDecodeBounds=true to check dimensions
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(res, resId, options);
    // Calculate inSampleSize
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
    // Decode bitmap with inSampleSize set
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(res, resId, options);
}
```

这个方法可以很轻易的加载任何大尺寸的位图给ImageView，这个ImageView展示了一个100*100像素的缩略图，就像下面的代码所展示的这样：
```java
mImageView.setImageBitmap(
    decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));
```

你可以遵循类似的过程来对其它资源进行解码，如果需要的话，可以替代使用合适的[BitmapFactory.decode*](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.html#decodeByteArray(byte[],%20int,%20int,%20android.graphics.BitmapFactory.Options))方法。