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

##加载位图到GridView