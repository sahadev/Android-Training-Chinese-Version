原文地址：[http://android.xsoftlab.net/training/displaying-bitmaps/cache-bitmap.html](http://android.xsoftlab.net/training/displaying-bitmaps/cache-bitmap.html)

往UI界面中加载单张图片的过程是很简单的，然而如果需要在某个时刻同时加载大量的图片，那么这事情就有些复杂了。在很多情况下，比如使用了ListView、GridView或者是ViewPager来展示一定数量的图片，在本质上这些情况下，屏幕的快速滑动会导致大量的图片被集中展示在屏幕上。

类似这样通过回收移除到屏幕之外的子View的组件会抑制内存的使用(也就是说它们本身不会滥用内存)。垃圾回收器还会释放你所加载的位图，假设你没有使用任何持久化引用的话。这真是极好的，但是为了保持流畅的UI效果，你可能需要在它们每次重新返回到屏幕的时候，对它们按照常规的方式重新处理。内存缓存及磁盘缓存可以在这里提供帮助，可以使这些组件快速的重新加载已经处理过的图片。

这节课将会讨论在加载多张图片的时候，如何通过使用内存缓存以及磁盘缓存来使UI界面更加流畅，响应速度更快。

##使用内存缓存
内存缓存提供了一种快速访问位图的能力，不过这会花费宝贵的内存空间。类[LruCache](http://android.xsoftlab.net/reference/android/util/LruCache.html)极其适合用来处理缓存图片的任务，它会将最近使用到的位图的引用存放在一个[LinkedHashMap](http://android.xsoftlab.net/reference/java/util/LinkedHashMap.html)对象上，并会在超过内存设计大小之前将最后一个没有用到的成员给驱除。

> **Note:** 在过去，使用[SoftReference](http://android.xsoftlab.net/reference/java/lang/ref/SoftReference.html)或者是[WeakReference](http://android.xsoftlab.net/reference/java/lang/ref/WeakReference.html)来缓存图片是最受欢迎的一种缓存方式，然而却并不推荐这么用。在Android 2.3之后，垃圾回收器对soft/weak引用的回收更加强制，这会使得这些引用几乎无效。此外，在Android 3.0之前，位图的字节数据被存储在本地内存中，可以预见这些数据是不会被释放的，这会导致程序很容易超过自身的内存限制，然后崩溃。

为了给[LruCache](http://android.xsoftlab.net/reference/android/util/LruCache.html)选择合适的尺寸，有几个因素应该被考虑在内：

- Activity或者程序在常规状态下的内存使用量是多少?
- 在同一时间最多会有多少图片集中显示在屏幕上?有多少内存需要为准备显示到屏幕上的图片所用?
- 设备屏幕的大小和尺寸分别是多少?在加载相同图片数量的情况下，像[Galaxy Nexus](http://www.android.com/devices/detail/galaxy-nexus)这种超高的密度(xhdpi)的设备与[Nexus S](http://www.android.com/devices/detail/nexus-s)(hdpi)相比则需要更大的内存。
- 图片的尺寸多大?配置是什么?加载这个位图的时候需要花费的内存是多少?
- 图片的访问有多频繁?会比其它位图访问更频繁吗?如果是这样，可能你需要将它们永远保持在内存中了，或者甚至是有多个[LruCache](http://android.xsoftlab.net/reference/android/util/LruCache.html)对象来为图片分组。
- 你可以在数量与质量之间取得平衡吗?某些时候存储大量的低质图片是很有用处的，可能会潜在的存在一些后台任务来加载一些高质量的版本。

这里特别没有指定尺寸或者配置，不过这适用所有的应用程序，这取决于对内存使用情况的分析，并需要找到一个适合的解决方案。缓存设置的太小会导致无意义的额外开销，缓存设置的太大会再次引起java.lang.OutOfMemory异常，应该将大小设置为应用的常规内存使用量之外的剩余内存之间。

下面是使用[LruCache](http://android.xsoftlab.net/reference/android/util/LruCache.html)缓存位图的一个例子：
```java
private LruCache<String, Bitmap> mMemoryCache;
@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Get max available VM memory, exceeding this amount will throw an
    // OutOfMemory exception. Stored in kilobytes as LruCache takes an
    // int in its constructor.
    final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
    // Use 1/8th of the available memory for this memory cache.
    final int cacheSize = maxMemory / 8;
    mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap bitmap) {
            // The cache size will be measured in kilobytes rather than
            // number of items.
            return bitmap.getByteCount() / 1024;
        }
    };
    ...
}
public void addBitmapToMemoryCache(String key, Bitmap bitmap) {
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }
}
public Bitmap getBitmapFromMemCache(String key) {
    return mMemoryCache.get(key);
}
```

> **Note:** 在这个例子中，有八分之一的内存被分配给了缓存。在正常的设备上(hdpi)这大概是4MB(32/8)左右。一个铺满了图片的GridView在全屏状态下的800*480的设备上所占的内存大概是1.5MB(800*480*4个字节)，所以这可以在内存中存储大概2.5页的图像。

当加载一个位图到ImageView上的时候，首先要检查[LruCache](http://android.xsoftlab.net/reference/android/util/LruCache.html)。如果发现了与之相匹配的，则会被用来立即更新到ImageView上，否则就会触发一个后台线程来处理图片：
```java
public void loadBitmap(int resId, ImageView imageView) {
    final String imageKey = String.valueOf(resId);
    final Bitmap bitmap = getBitmapFromMemCache(imageKey);
    if (bitmap != null) {
        mImageView.setImageBitmap(bitmap);
    } else {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }
}
```

BitmapWorkerTask中也需要对内存缓存进行添加或更新:
```java
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final Bitmap bitmap = decodeSampledBitmapFromResource(
                getResources(), params[0], 100, 100));
        addBitmapToMemoryCache(String.valueOf(params[0]), bitmap);
        return bitmap;
    }
    ...
}
```

##使用磁盘缓存
内存缓存对于最近浏览过的图像的快速加载非常有用，然而却不能将所有的图像都存放在内存缓存中。像GridView这样的组件在加载大数据集的时候可以轻易的将内存缓存填满。程序在运行的过程中可能会被其它任务打断，比如一个来电，这时，在后台的任务可能就会被杀死，内存缓存也会被销毁。一旦用户返回了界面，那么程序就需要再次重新处理每张图片。

那么磁盘缓存在这些情况下就很有帮助了，它可以存储处理过的图片，并会辅助提升图片的加载时间，在图片不再在内存缓存中存在的时候。当然，在磁盘上获取一张图片要比内存中要慢，并且还需要开启单独的工作线程，这和从磁盘上读取数据的时间一样，都不可预估。

> **Note:**[ContentProvider](http://android.xsoftlab.net/reference/android/content/ContentProvider.html)可能更适合用来存放被缓存过的图像，如果这些图像的访问更加频繁的话，就像在相册应用中的情况一样。

从Android Source中更新的示例代码使用了一个DiskLruCache的实现。下面是个更新后的版本，它对已有的内存缓存增加了磁盘缓存：
```java
private DiskLruCache mDiskLruCache;
private final Object mDiskCacheLock = new Object();
private boolean mDiskCacheStarting = true;
private static final int DISK_CACHE_SIZE = 1024 * 1024 * 10; // 10MB
private static final String DISK_CACHE_SUBDIR = "thumbnails";
@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Initialize memory cache
    ...
    // Initialize disk cache on background thread
    File cacheDir = getDiskCacheDir(this, DISK_CACHE_SUBDIR);
    new InitDiskCacheTask().execute(cacheDir);
    ...
}
class InitDiskCacheTask extends AsyncTask<File, Void, Void> {
    @Override
    protected Void doInBackground(File... params) {
        synchronized (mDiskCacheLock) {
            File cacheDir = params[0];
            mDiskLruCache = DiskLruCache.open(cacheDir, DISK_CACHE_SIZE);
            mDiskCacheStarting = false; // Finished initialization
            mDiskCacheLock.notifyAll(); // Wake any waiting threads
        }
        return null;
    }
}
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final String imageKey = String.valueOf(params[0]);
        // Check disk cache in background thread
        Bitmap bitmap = getBitmapFromDiskCache(imageKey);
        if (bitmap == null) { // Not found in disk cache
            // Process as normal
            final Bitmap bitmap = decodeSampledBitmapFromResource(
                    getResources(), params[0], 100, 100));
        }
        // Add final bitmap to caches
        addBitmapToCache(imageKey, bitmap);
        return bitmap;
    }
    ...
}
public void addBitmapToCache(String key, Bitmap bitmap) {
    // Add to memory cache as before
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }
    // Also add to disk cache
    synchronized (mDiskCacheLock) {
        if (mDiskLruCache != null && mDiskLruCache.get(key) == null) {
            mDiskLruCache.put(key, bitmap);
        }
    }
}
public Bitmap getBitmapFromDiskCache(String key) {
    synchronized (mDiskCacheLock) {
        // Wait while disk cache is started from background thread
        while (mDiskCacheStarting) {
            try {
                mDiskCacheLock.wait();
            } catch (InterruptedException e) {}
        }
        if (mDiskLruCache != null) {
            return mDiskLruCache.get(key);
        }
    }
    return null;
}
// Creates a unique subdirectory of the designated app cache directory. Tries to use external
// but if not mounted, falls back on internal storage.
public static File getDiskCacheDir(Context context, String uniqueName) {
    // Check if media is mounted or storage is built-in, if so, try and use external cache dir
    // otherwise use internal cache dir
    final String cachePath =
            Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||
                    !isExternalStorageRemovable() ? getExternalCacheDir(context).getPath() :
                            context.getCacheDir().getPath();
    return new File(cachePath + File.separator + uniqueName);
}
```

> **Note:**因为磁盘缓存的初始化需要磁盘操作，所以这个过程不应该放在UI线程中执行。然而，这也意味着在缓存初始化之前这是个访问的机会。为了做到这一点，需要有个lock对象来保证在缓存被初始化之前APP没有从磁盘缓存中读取数据。

内存缓存在UI线程中执行检查，磁盘缓存在后台线程中执行检查。磁盘操作绝不应该放入UI线程。当图像处理完毕后，最终被处理过的图片应当被添加到内存缓存及磁盘缓存中以便备用。

##处理配置变更
如果在运行时发生了变更，比如屏幕的方向发生了改变，会引起Android销毁并重启运行中的Activity，你可能想要避免再一次处理图像，这样一旦配置发生了改变，可以使用户有一个流畅快速的用户体验。

幸运的是，你有一个非常赞的内存缓存方案：可以使用设置了setRetainInstance(true)的Fragment，它可以将缓存传入新的Activity实例。在activity重新创建的时候，这个被保留存在的Fragment会被重新附加在Activity上，你可以获得原先内存缓存的访问能力，这使得图像可以快速的被获得并被重新填充在ImageView对象中。

下面这个例子使用了引用LruCache的Fragment，并通过了配置更改的问题：
```java
private LruCache<String, Bitmap> mMemoryCache;
@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    RetainFragment retainFragment =
            RetainFragment.findOrCreateRetainFragment(getFragmentManager());
    mMemoryCache = retainFragment.mRetainedCache;
    if (mMemoryCache == null) {
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            ... // Initialize cache here as usual
        }
        retainFragment.mRetainedCache = mMemoryCache;
    }
    ...
}
class RetainFragment extends Fragment {
    private static final String TAG = "RetainFragment";
    public LruCache<String, Bitmap> mRetainedCache;
    public RetainFragment() {}
    public static RetainFragment findOrCreateRetainFragment(FragmentManager fm) {
        RetainFragment fragment = (RetainFragment) fm.findFragmentByTag(TAG);
        if (fragment == null) {
            fragment = new RetainFragment();
            fm.beginTransaction().add(fragment, TAG).commit();
        }
        return fragment;
    }
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRetainInstance(true);
    }
}
```

为了测试这项输出，试着在有和没有Fragment的情况下旋转设备。你应该会注意到这个过程几乎没有延迟。任何图像如果没有在内存缓存中找到，那么这就为磁盘缓存提供了用武之地，如果都没有的话，那么常规的处理方法就会出场。