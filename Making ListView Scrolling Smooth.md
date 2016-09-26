原文地址：[http://android.xsoftlab.net/training/improving-layouts/smooth-scrolling.html](http://android.xsoftlab.net/training/improving-layouts/smooth-scrolling.html)

想要让ListView滑动流畅的关键所在是减轻主线程的负担。要确保任何的磁盘访问、网络访问、或者SQL访问都是在单独的线程中执行的。如果要测试APP的状态，可以开启[StrictMode](http://android.xsoftlab.net/reference/android/os/StrictMode.html)。

##使用后台线程
使用工作线程可以使UI线程将所有的注意力都集中在UI的绘制上。在很多情况下，使用[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)所提供的功能就可以在工作线程中处理耗时任务。[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)会自动的将[execute()](http://android.xsoftlab.net/reference/android/os/AsyncTask.html#execute(Params...))发起的请求排队，并依次执行。这意味着你不要自己创建线程池。

在下面的示例代码中，[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)被用来加载一张图像，并在加载结束后自动的将其渲染到UI上。它还在图像加载的时候显示了一个旋转的进度条。
```java
// Using an AsyncTask to load the slow images in a background thread
new AsyncTask<ViewHolder, Void, Bitmap>() {
    private ViewHolder v;
    @Override
    protected Bitmap doInBackground(ViewHolder... params) {
        v = params[0];
        return mFakeImageLoader.getImage();
    }
    @Override
    protected void onPostExecute(Bitmap result) {
        super.onPostExecute(result);
        if (v.position == position) {
            // If this item hasn't been recycled already, hide the
            // progress and set and show the image
            v.progress.setVisibility(View.GONE);
            v.icon.setVisibility(View.VISIBLE);
            v.icon.setImageBitmap(result);
        }
    }
}.execute(holder);
```

从Android 3.0开始，[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)提供了一项新特性：可以将任务运行在多核处理器上。你可以使用[executeOnExecutor()](http://android.xsoftlab.net/reference/android/os/AsyncTask.html#executeOnExecutor(java.util.concurrent.Executor, Params...))方法发起执行请求，这样多个请求就可以同时进行，同时进行的任务数量取决于CPU的核心数量。

##使用View Holder持有View对象
在滑动ListView时，代码可能会频繁的调用[findViewById()](http://android.xsoftlab.net/reference/android/app/Activity.html#findViewById(int))，这会降低性能。就算是Adapter将已经加载过的View返回，但是在复用时还是需要去查询这些View来更新它们。杜绝重复使用[findViewById()](http://android.xsoftlab.net/reference/android/app/Activity.html#findViewById(int))的方法就是使用"View Holder"设计模式。

ViewHolder对象将每个View组件存储于布局容器的tag属性内，所以你可以快速访问它们而不需要每次都去查询。首先，你需要创建一个类来持有已加载的View：
```java
static class ViewHolder {
  TextView text;
  TextView timestamp;
  ImageView icon;
  ProgressBar progress;
  int position;
}
```

然后对ViewHolder的成员属性赋值，然后将其存放在布局容器内：
```java
ViewHolder holder = new ViewHolder();
holder.icon = (ImageView) convertView.findViewById(R.id.listitem_image);
holder.text = (TextView) convertView.findViewById(R.id.listitem_text);
holder.timestamp = (TextView) convertView.findViewById(R.id.listitem_timestamp);
holder.progress = (ProgressBar) convertView.findViewById(R.id.progress_spinner);
convertView.setTag(holder);
```

那么现在就可以很方便的对这些View组件进行访问，而不再需要对它们单独进行查询，如此便可以节省出宝贵的CPU资源。