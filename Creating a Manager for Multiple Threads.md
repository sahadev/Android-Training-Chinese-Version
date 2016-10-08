原文地址：[http://android.xsoftlab.net/training/multiple-threads/create-threadpool.html](http://android.xsoftlab.net/training/multiple-threads/create-threadpool.html)

上节课我们学习了如何定义一个任务。如果只是执行单次任务，那么刚刚所学的已经基本满足要求了。如果需要针对不同的数据执行同种任务，并且需要同一时间只能执行一项任务，那么[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)可能会适合你。如果要使任务随着资源的可用而执行，或者同一时间需要运行多个任务，那么就需要专门管理这些线程了。Android系统为此提供了一个类，那就是传说中的[ThreadPoolExecutor](http://android.xsoftlab.net/reference/java/util/concurrent/ThreadPoolExecutor.html)。它可以在线程可用时自动运行队列中的任务。如果要运行一个任务，只需要将任务添加到队列中即可。

ThreadPoolExecutor允许多个线程同时进行，所以应当确保代码是线程安全的。要确保可能会被多个线程访问到的变量被放在同步代码块中。这种方法可以防止一个线程正在读取一个变量的值，而另一个变量正在对这个变量写入新值的现象出现。通常情况下，这种现象会发生在静态变量中，但是也会发生在单例对象上。

##定义线程池类
在该类内实例化一个[ThreadPoolExecutor](http://android.xsoftlab.net/reference/java/util/concurrent/ThreadPoolExecutor.html)对象,并在该类内执行以下工作：

- #####对线程池对象使用静态变量引用
在APP内通常只需要一个线程池对象的存在：为了对CPU资源或网络资源有一个单一的控制。如果含有不同的[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)类型，你可能希望对每种类型都创建一个单独线程池，但是每种类型都还是单例类型。在这里的例子中，可以添加以下代码作为全局属性声明。
 ```java
	public class PhotoManager {
	    ...
	    static  {
	        ...
	        // Creates a single static instance of PhotoManager
	        sInstance = new PhotoManager();
	    }
	    ...
 ```

- 使用私有构造方法：
私有的构造方法可以确保该类只有一个对象存在。
```java
public class PhotoManager {
    ...
    /**
     * Constructs the work queues and thread pools used to download
     * and decode images. Because the constructor is marked private,
     * it's unavailable to other classes, even in the same package.
     */
    private PhotoManager() {
    ...
    }
```

- 调用线程池的相关方法启动任务：
 在线程池类内定义一个可以向线程池队列中添加任务的方法。
 ```java
 public class PhotoManager {
    ...
    // Called by the PhotoView to get a photo
    static public PhotoTask startDownload(
        PhotoView imageView,
        boolean cacheFlag) {
        ...
        // Adds a download task to the thread pool for execution
        sInstance.
                mDownloadThreadPool.
                execute(downloadTask.getHTTPDownloadRunnable());
        ...
    }
 ```

- 在构造方法中实例化一个Handler对象，并将其连接到UI线程：Handler允许安全的调用比如View这种UI对象。大多数的UI对象只允许被UI线程访问。
 ```java
     private PhotoManager() {
    ...
        // Defines a Handler object that's attached to the UI thread
        mHandler = new Handler(Looper.getMainLooper()) {
            /*
             * handleMessage() defines the operations to perform when
             * the Handler receives a new Message to process.
             */
            @Override
            public void handleMessage(Message inputMessage) {
                ...
            }
        ...
        }
    }
 ```

##确定线程池参数
如果要实例化ThreadPoolExecutor对象，需要用到以下值：

**线程池的初始值以及最大值：**

 - 线程数量的初始值用于指定池子的初始大小，最大值代表了该线程池所允许开放的最大并发数量。线程池中的线程数量取决于设备的可用核心数。这个值与当前的设备颇有关系：
 
```java
public class PhotoManager {
...
    /*
     * Gets the number of available cores
     * (not always the same as the maximum number of cores)
     */
    private static int NUMBER_OF_CORES =
            Runtime.getRuntime().availableProcessors();
}
```

 这个值可能不能够反映设备的物理核心数量；在一些设备上，核心是否可用取决于系统是否加载。对于这些设备，[availableProcessors()](http://android.xsoftlab.net/reference/java/lang/Runtime.html#availableProcessors())返回了当前的活跃核心数量，这个值可能要比真实的总核心数要低。

**存活时长及单位**

 - 线程在被关闭之间的闲置时长。这个时长由时间单元值负责解释，这些时间单位常量被定义在类[TimeUnit](http://android.xsoftlab.net/reference/java/util/concurrent/TimeUnit.html)中。

**任务队列**

 - 一个持有Runnable对象的队列。为了启动线程中的执行代码，线程池管理器采用了先进先出的管理原则。在创建线程池之前需要提供一个这样的队列，使用任何实现了[BlockingQueue](http://android.xsoftlab.net/reference/java/util/concurrent/BlockingQueue.html)接口的类皆可。为了匹配到APP的要求，你可以选择适当的队列实现；学习更多它们的相关知识，请参见ThreadPoolExecutor的描述文档。这里的使用了LinkedBlockingQueue：
```java
 public class PhotoManager {
    ...
    private PhotoManager() {
        ...
        // A queue of Runnables
        private final BlockingQueue<Runnable> mDecodeWorkQueue;
        ...
        // Instantiates the queue of Runnables as a LinkedBlockingQueue
        mDecodeWorkQueue = new LinkedBlockingQueue<Runnable>();
        ...
    }
    ...
}
```

##线程池创建
创建一个线程池，只需调用ThreadPoolExecutor()实例化一个线程池管理器就好。它会创建并管理一组线程。因为线程池大小的初始值与最大值是相同的，所以ThreadPoolExecutor()会在初始化的时候创建出所指定的线程数量:

```java
    private PhotoManager() {
        ...
        // Sets the amount of time an idle thread waits before terminating
        private static final int KEEP_ALIVE_TIME = 1;
        // Sets the Time Unit to seconds
        private static final TimeUnit KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS;
        // Creates a thread pool manager
        mDecodeThreadPool = new ThreadPoolExecutor(
                NUMBER_OF_CORES,       // Initial pool size
                NUMBER_OF_CORES,       // Max pool size
                KEEP_ALIVE_TIME,
                KEEP_ALIVE_TIME_UNIT,
                mDecodeWorkQueue);
    }
```

