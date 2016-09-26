原文地址：[http://android.xsoftlab.net/training/displaying-bitmaps/process-bitmap.html](http://android.xsoftlab.net/training/displaying-bitmaps/process-bitmap.html)

我们在上节课[Load Large Bitmaps Efficiently](http://android.xsoftlab.net/training/displaying-bitmaps/load-bitmap.html)中讨论了[BitmapFactory.decode*](http://android.xsoftlab.net/reference/android/graphics/BitmapFactory.html#decodeByteArray(byte[],%20int,%20int,%20android.graphics.BitmapFactory.Options))方法，说到了不应该在UI线程中执行读取数据的过程，尤其是从磁盘或者网络上读取数据(或者其它读取速度次于内存的地方)。读取数据的时间是不可预料的，这取决于各种各样的因素(从磁盘或者网络读取的速度、图片的大小、CPU的功率,etc.)。如果这其中的一个因素阻塞了UI线程，那么系统会标志程序为无响应标志，并会给用户提供一个关闭的选项(请查看[Designing for Responsiveness](http://android.xsoftlab.net/guide/practices/responsiveness.html)获取更多信息)。

这节课讨论了通过使用[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)在非UI线程中处理位图以及展示如何处理并发问题。

##使用[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)
类[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)提供了一种简要的方式来处理后台进程的工作，并会将处理后的结果推送到UI线程中。如果要使用这个类，需要创建该类的子类，然后重写所提供的方法。这里有个例子，展示了如何使用[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)及[decodeSampledBitmapFromResource()](http://android.xsoftlab.net/training/displaying-bitmaps/load-bitmap.html#decodeSampledBitmapFromResource)来加载一张大图到[ImageView](http://android.xsoftlab.net/reference/android/widget/ImageView.html)上：
```java
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    private final WeakReference<ImageView> imageViewReference;
    private int data = 0;
    public BitmapWorkerTask(ImageView imageView) {
        // Use a WeakReference to ensure the ImageView can be garbage collected
        imageViewReference = new WeakReference<ImageView>(imageView);
    }
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        data = params[0];
        return decodeSampledBitmapFromResource(getResources(), data, 100, 100));
    }
    // Once complete, see if ImageView is still around and set bitmap.
    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            if (imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```

[ImageView](http://android.xsoftlab.net/reference/android/widget/ImageView.html)的[WeakReference](http://android.xsoftlab.net/reference/java/lang/ref/WeakReference.html)可以确保[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)不会阻止ImageView及它所引用的事务被垃圾回收器回收。这不能保证在任务执行完毕的时候[ImageView](http://android.xsoftlab.net/reference/android/widget/ImageView.html)还依然存在，所以你还必须在[onPostExecute()](http://android.xsoftlab.net/reference/android/os/AsyncTask.html#onPostExecute(Result))方法中检查一下它的引用。[ImageView](http://android.xsoftlab.net/reference/android/widget/ImageView.html)可能已经不存在了,比如说吧，当用户离开了activity或者在任务结束的时候一些配置发生了变化。

为了启动异步任务来加载图片,需要简单的创建一个新任务并执行它:
```java
public void loadBitmap(int resId, ImageView imageView) {
    BitmapWorkerTask task = new BitmapWorkerTask(imageView);
    task.execute(resId);
}
```

##处理并发
一些普通的View控件比如[ListView](http://android.xsoftlab.net/reference/android/widget/ListView.html)和[GridView](http://android.xsoftlab.net/reference/android/widget/GridView.html)会涉及到另一个问题，就是当与[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)结合使用的时候会出现并发问题。为了能有效的使用内存，这些控件会随着用户的滑动来回收子View。如果每一个子View都会触发一个[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)，那么就不能保障在任务完成的时候，与之相关联的View没有被回收利用。此外，对于顺序启动的任务也不能保障可以按顺序完成。

博客[Multithreading for Performance](http://android-developers.blogspot.com/2010/07/multithreading-for-performance.html)进一步的讨论了如何处理并发，它提供了一个解决方案：在ImageView中存储了最近的[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)的引用，这个引用可以在任务完成的时候对最近的[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)进行检查。通过类似的办法，那么上面章节的[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)可以被扩展成类似的模式。

创建一个专用的[Drawable](http://android.xsoftlab.net/reference/android/graphics/drawable/Drawable.html)子类来存储工作任务的引用。在这种情况下，[BitmapDrawable](http://android.xsoftlab.net/reference/android/graphics/drawable/BitmapDrawable.html)就会被用到，所以在任务完成之前可以有一个占位图显示在ImageView上：
```java
static class AsyncDrawable extends BitmapDrawable {
    private final WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference;
    public AsyncDrawable(Resources res, Bitmap bitmap,
            BitmapWorkerTask bitmapWorkerTask) {
        super(res, bitmap);
        bitmapWorkerTaskReference =
            new WeakReference<BitmapWorkerTask>(bitmapWorkerTask);
    }
    public BitmapWorkerTask getBitmapWorkerTask() {
        return bitmapWorkerTaskReference.get();
    }
}
```

在执行[BitmapWorkerTask](http://android.xsoftlab.net/training/displaying-bitmaps/process-bitmap.html#BitmapWorkerTask)任务之前，你可以创建一个[AsyncDrawable](http://android.xsoftlab.net/training/displaying-bitmaps/process-bitmap.html#AsyncDrawable)并将这个任务绑定到目标ImageView上：
```java
public void loadBitmap(int resId, ImageView imageView) {
    if (cancelPotentialWork(resId, imageView)) {
        final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
        final AsyncDrawable asyncDrawable =
                new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
        imageView.setImageDrawable(asyncDrawable);
        task.execute(resId);
    }
}
```

上面代码所引用的cancelPotentialWork()方法用来检查是否有另外在进行中的任务已经与ImageView关联上了。如果是这样的话，它会通过[cancel()](http://android.xsoftlab.net/reference/android/os/AsyncTask.html#cancel(boolean))尝试取消原来的任务。在少数情况下，新建的任务数据可能会与已经存在的任务相匹配，所以就不要有进一步的动作。下面是cancelPotentialWork()方法的实现：
```java
public static boolean cancelPotentialWork(int data, ImageView imageView) {
    final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);
    if (bitmapWorkerTask != null) {
        final int bitmapData = bitmapWorkerTask.data;
        // If bitmapData is not yet set or it differs from the new data
        if (bitmapData == 0 || bitmapData != data) {
            // Cancel previous task
            bitmapWorkerTask.cancel(true);
        } else {
            // The same work is already in progress
            return false;
        }
    }
    // No task associated with the ImageView, or an existing task was cancelled
    return true;
}
```

有个辅助方法：getBitmapWorkerTask()，它被用来接收与指定ImageView相关联的任务：
```java
private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
   if (imageView != null) {
       final Drawable drawable = imageView.getDrawable();
       if (drawable instanceof AsyncDrawable) {
           final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
           return asyncDrawable.getBitmapWorkerTask();
       }
    }
    return null;
}
```

最后一步就是在[BitmapWorkerTask](http://android.xsoftlab.net/training/displaying-bitmaps/process-bitmap.html#BitmapWorkerTask)中更新onPostExecute()，所以它会检查任务是否已经被取消和检查当前的任务是否与与之相关联的ImageView相匹配：
```java
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (isCancelled()) {
            bitmap = null;
        }
        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            final BitmapWorkerTask bitmapWorkerTask =
                    getBitmapWorkerTask(imageView);
            if (this == bitmapWorkerTask && imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```

现在这个实现就适合用到类似[ListView](http://android.xsoftlab.net/reference/android/widget/ListView.html)和[GridView](http://android.xsoftlab.net/reference/android/widget/GridView.html)这种会回收它们子View的组件上了，简单的调用loadBitmap()就可以正常给ImageView设置图片了。比如，在一个[GridView](http://android.xsoftlab.net/reference/android/widget/GridView.html)的实现中，这个方法就可以在相应适配器的getView()方法中使用。