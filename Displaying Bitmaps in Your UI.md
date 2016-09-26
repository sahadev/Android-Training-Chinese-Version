原文地址：[http://android.xsoftlab.net/training/displaying-bitmaps/display-bitmap.html#gridview](http://android.xsoftlab.net/training/displaying-bitmaps/display-bitmap.html#gridview)

这节课会将前面的知识点整合到一起，展示如何使用后台线程、位图缓存来加载多张图片到ViewPager或者GridView中，并会涉及并发处理及配置更改。

##加载位图到ViewPager
[swipe view pattern](http://android.xsoftlab.net/design/patterns/swipe-views.html)是画廊应用详情页之间来回跳转的最佳演示。你可以使用[ViewPager](http://android.xsoftlab.net/reference/android/support/v4/view/ViewPager.html)来实现这种模式。无论如何，[FragmentStatePagerAdapter](http://android.xsoftlab.net/reference/android/support/v4/app/FragmentStatePagerAdapter.html)是ViewPager支撑适配器的最佳选择，它可以随着Fragment在屏幕上的消失自动销毁与保存Fragment的状态，降低内存的使用。

> **Note:** 如果你有少量的图片并且可以保证它们一起被加入内存之后不会超出内存的限制，那么使用正规的[PagerAdapter](http://android.xsoftlab.net/reference/android/support/v4/view/PagerAdapter.html)或者[FragmentPagerAdapter](http://android.xsoftlab.net/reference/android/support/v4/app/FragmentPagerAdapter.html)可能更合适些。

下面是带有ImageView的[ViewPager](http://android.xsoftlab.net/reference/android/support/v4/view/ViewPager.html)实现。MainActivity持有了ViewPager及Adapter的引用：
```java
public class ImageDetailActivity extends FragmentActivity {
    public static final String EXTRA_IMAGE = "extra_image";
    private ImagePagerAdapter mAdapter;
    private ViewPager mPager;
    // A static dataset to back the ViewPager adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.image_detail_pager); // Contains just a ViewPager
        mAdapter = new ImagePagerAdapter(getSupportFragmentManager(), imageResIds.length);
        mPager = (ViewPager) findViewById(R.id.pager);
        mPager.setAdapter(mAdapter);
    }
    public static class ImagePagerAdapter extends FragmentStatePagerAdapter {
        private final int mSize;
        public ImagePagerAdapter(FragmentManager fm, int size) {
            super(fm);
            mSize = size;
        }
        @Override
        public int getCount() {
            return mSize;
        }
        @Override
        public Fragment getItem(int position) {
            return ImageDetailFragment.newInstance(position);
        }
    }
}
```
下面是详情Fragment的实现，它持有了ImageView的引用。这可能看起来是一种非常合理的方式，但是你可以看出这个实现的缺陷吗？如何改进它？
```java
public class ImageDetailFragment extends Fragment {
    private static final String IMAGE_DATA_EXTRA = "resId";
    private int mImageNum;
    private ImageView mImageView;
    static ImageDetailFragment newInstance(int imageNum) {
        final ImageDetailFragment f = new ImageDetailFragment();
        final Bundle args = new Bundle();
        args.putInt(IMAGE_DATA_EXTRA, imageNum);
        f.setArguments(args);
        return f;
    }
    // Empty constructor, required as per Fragment docs
    public ImageDetailFragment() {}
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mImageNum = getArguments() != null ? getArguments().getInt(IMAGE_DATA_EXTRA) : -1;
    }
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        // image_detail_fragment.xml contains just an ImageView
        final View v = inflater.inflate(R.layout.image_detail_fragment, container, false);
        mImageView = (ImageView) v.findViewById(R.id.imageView);
        return v;
    }
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        final int resId = ImageDetailActivity.imageResIds[mImageNum];
        mImageView.setImageResource(resId); // Load image into ImageView
    }
}
```
希望你会注意到这个问题：读取图片资源的过程是在UI线程中执行的，这会导致程序的卡顿及被强制关闭。使用在[Processing Bitmaps Off the UI Thread](http://android.xsoftlab.net/training/displaying-bitmaps/process-bitmap.html)课程中所描述的[AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html)，它可以简单的将图片的加载与处理过程交给后台线程：
```java
public class ImageDetailActivity extends FragmentActivity {
    ...
    public void loadBitmap(int resId, ImageView imageView) {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }
    ... // include BitmapWorkerTask class
}
public class ImageDetailFragment extends Fragment {
    ...
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        if (ImageDetailActivity.class.isInstance(getActivity())) {
            final int resId = ImageDetailActivity.imageResIds[mImageNum];
            // Call out to ImageDetailActivity to load the bitmap in a background thread
            ((ImageDetailActivity) getActivity()).loadBitmap(resId, mImageView);
        }
    }
}
```
任何额外的处理，比如调整图片尺寸或者从网络获取图像，这些处理过程可以放入[BitmapWorkerTask](http://android.xsoftlab.net/training/displaying-bitmaps/process-bitmap.html#BitmapWorkerTask)而不会影响到主线程的响应能力。如果后台线程做的不仅仅是从磁盘上直接加载图像，那么它还益于处理添加内存缓存及磁盘缓存。下面是针对于内存缓存的专门修改版本：
```java
public class ImageDetailActivity extends FragmentActivity {
    ...
    private LruCache<String, Bitmap> mMemoryCache;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        ...
        // initialize LruCache as per Use a Memory Cache section
    }
    public void loadBitmap(int resId, ImageView imageView) {
        final String imageKey = String.valueOf(resId);
        final Bitmap bitmap = mMemoryCache.get(imageKey);
        if (bitmap != null) {
            mImageView.setImageBitmap(bitmap);
        } else {
            mImageView.setImageResource(R.drawable.image_placeholder);
            BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
            task.execute(resId);
        }
    }
    ... // include updated BitmapWorkerTask from Use a Memory Cache section
}
```
以最小的加载延迟将这些块提供给ViewPager，并尽可能的将处理图片的工作放入后台线程。

##加载位图到GridView
[grid list building block](http://android.xsoftlab.net/design/building-blocks/grid-lists.html)对展示图片数据集很有帮助，它主要通过GridView来实现，它可以将很多图片在同一时间显示出来，如果用户要来回滑动，那可能会有更多的图片需要准备展示出来。当实现这种控制类型时，你必须确保UI依旧流畅，内存的使用依旧在控制范围之内，并发的处理依旧正确(因为GridView会回收它的子View)。

首先，这里有一个标准的GridView实现。再者，这可能看起来非常完美，但是它还有可以改进的地方吗?
```java
public class ImageGridFragment extends Fragment implements AdapterView.OnItemClickListener {
    private ImageAdapter mAdapter;
    // A static dataset to back the GridView adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};
    // Empty constructor as per Fragment docs
    public ImageGridFragment() {}
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mAdapter = new ImageAdapter(getActivity());
    }
    @Override
    public View onCreateView(
            LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        final View v = inflater.inflate(R.layout.image_grid_fragment, container, false);
        final GridView mGridView = (GridView) v.findViewById(R.id.gridView);
        mGridView.setAdapter(mAdapter);
        mGridView.setOnItemClickListener(this);
        return v;
    }
    @Override
    public void onItemClick(AdapterView<?> parent, View v, int position, long id) {
        final Intent i = new Intent(getActivity(), ImageDetailActivity.class);
        i.putExtra(ImageDetailActivity.EXTRA_IMAGE, position);
        startActivity(i);
    }
    private class ImageAdapter extends BaseAdapter {
        private final Context mContext;
        public ImageAdapter(Context context) {
            super();
            mContext = context;
        }
        @Override
        public int getCount() {
            return imageResIds.length;
        }
        @Override
        public Object getItem(int position) {
            return imageResIds[position];
        }
        @Override
        public long getItemId(int position) {
            return position;
        }
        @Override
        public View getView(int position, View convertView, ViewGroup container) {
            ImageView imageView;
            if (convertView == null) { // if it's not recycled, initialize some attributes
                imageView = new ImageView(mContext);
                imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
                imageView.setLayoutParams(new GridView.LayoutParams(
                        LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
            } else {
                imageView = (ImageView) convertView;
            }
            imageView.setImageResource(imageResIds[position]); // Load image into ImageView
            return imageView;
        }
    }
}
```

再提示一次，这个实现的问题图像的设置过程位于UI线程。虽然这可能只是应用小图像、简单图像，但如果有任何的额外工作需要处理，那么UI线程就会戛然而止。

前一节中的一步处理及缓存图像可以实现到这里。无论如何，你还需要对由GridView所引起的并发问题提高警惕。如果要处理这个问题，可以使用在[Processing Bitmaps Off the UI Thread](http://android.xsoftlab.net/training/displaying-bitmaps/process-bitmap.html#concurrency)课程中所提到的技巧。下面是更改后的版本：
```java
public class ImageGridFragment extends Fragment implements AdapterView.OnItemClickListener {
    ...
    private class ImageAdapter extends BaseAdapter {
        ...
        @Override
        public View getView(int position, View convertView, ViewGroup container) {
            ...
            loadBitmap(imageResIds[position], imageView)
            return imageView;
        }
    }
    public void loadBitmap(int resId, ImageView imageView) {
        if (cancelPotentialWork(resId, imageView)) {
            final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
            final AsyncDrawable asyncDrawable =
                    new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
            imageView.setImageDrawable(asyncDrawable);
            task.execute(resId);
        }
    }
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
    public static boolean cancelPotentialWork(int data, ImageView imageView) {
        final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);
        if (bitmapWorkerTask != null) {
            final int bitmapData = bitmapWorkerTask.data;
            if (bitmapData != data) {
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
    ... // include updated BitmapWorkerTask class
```

> **Note:** 上面的代码也可以应用于ListView。

这个实现对如何处理及加载图像是很灵活的，并不会妨碍UI的流畅性。后台任务可以从网络加载图像并且可以调整数码相片的尺寸，并会在任务结束的时候提供处理后的图像。

对于本示例的全部代码以及这节课中讨论到的其它概念，可以查看包含这部分功能的示例应用。