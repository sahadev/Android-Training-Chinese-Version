原文地址：[http://android.xsoftlab.net/training/multiple-threads/run-code.html#StopThread](http://android.xsoftlab.net/training/multiple-threads/run-code.html#StopThread)

上节课我们学习了如何定义一个类用于管理线程以及任务。这节课将会学习如何在线程池中运行任务。要做到这一点，只需要往线程池的工作队列中添加任务即可。当一条线程处于闲置状态时，那么ThreadPoolExecutor会从任务队列中取出一条任务并放入该线程中运行。

这节课还介绍了如何停止一个正在运行中的任务。如果在任务开始后，可能发现这项任务并不是必须的，那么就需要用到任务取消的功能了。这样可以避免浪费处理器的时间。举个例子，如果你正从网络上下载一张图像，如果侦测到这张图像已经在缓存中了，那么这时就需要停止这项网络任务了。

##在线程池中的线程内运行任务
为了在指定的线程池中启动一项线程任务，需要将Runnable对象传给ThreadPoolExecutor的execute()方法。这个方法会将任务添加到线程池的工作队列中去。当其中一个线程变为闲置状态时，那么线程池管理器会从队列中取出一个已经等待了很久的任务，然后放到这个线程中运行：
```java
public class PhotoManager {
    public void handleState(PhotoTask photoTask, int state) {
        switch (state) {
            // The task finished downloading the image
            case DOWNLOAD_COMPLETE:
            // Decodes the image
                mDecodeThreadPool.execute(
                        photoTask.getPhotoDecodeRunnable());
            ...
        }
        ...
    }
    ...
}
```

当ThreadPoolExecutor启动一个Runnable时，它会自动调用Runnable的run()方法。

##中断执行中的代码
如果要停止一项任务，那么需要中断该任务所在的线程。为了可以预先做到这一点，那么需要在任务创建时存储该任务所在线程的句柄：
```java
class PhotoDecodeRunnable implements Runnable {
    // Defines the code to run for this task
    public void run() {
        /*
         * Stores the current Thread in the
         * object that contains PhotoDecodeRunnable
         */
        mPhotoTask.setImageDecodeThread(Thread.currentThread());
        ...
    }
    ...
}
```
我们可以调用Thread.interrupt()方法来中断一个线程。这里要注意Thread对象是由系统控制的，系统会在应用进程的范围之外修改它们。正因为这个原因，在中断线程之前，需要对线程的访问加锁。通常需要将这部分代码放入同步代码块中：
```java
public class PhotoManager {
    public static void cancelAll() {
        /*
         * Creates an array of Runnables that's the same size as the
         * thread pool work queue
         */
        Runnable[] runnableArray = new Runnable[mDecodeWorkQueue.size()];
        // Populates the array with the Runnables in the queue
        mDecodeWorkQueue.toArray(runnableArray);
        // Stores the array length in order to iterate over the array
        int len = runnableArray.length;
        /*
         * Iterates over the array of Runnables and interrupts each one's Thread.
         */
        synchronized (sInstance) {
            // Iterates over the array of tasks
            for (int runnableIndex = 0; runnableIndex < len; runnableIndex++) {
                // Gets the current thread
                Thread thread = runnableArray[taskArrayIndex].mThread;
                // if the Thread exists, post an interrupt to it
                if (null != thread) {
                    thread.interrupt();
                }
            }
        }
    }
    ...
}
```

在多数情况下，Thread.interrupt()会使线程立刻停止。然而，它只会将那些正在等待的线程停下来，它并不会中止CPU或网络任务。为了避免使系统变慢或卡顿，你应当在开始任意一项操作之前测试是否有中断请求：
```java
/*
 * Before continuing, checks to see that the Thread hasn't
 * been interrupted
 */
if (Thread.interrupted()) {
    return;
}
...
// Decodes a byte array into a Bitmap (CPU-intensive)
BitmapFactory.decodeByteArray(
        imageBuffer, 0, imageBuffer.length, bitmapOptions);
...
```


