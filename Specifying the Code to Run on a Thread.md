原文地址：[http://android.xsoftlab.net/training/multiple-threads/index.html](http://android.xsoftlab.net/training/multiple-threads/index.html)

#引言
大量的数据处理往往需要花费很长的时间，但如果将这些工作切分并行处理，那么它的速度与效率就会提升很多。在多线程处理器的设备中，系统可以并行运行线程。比如，使用多线程将图像文件切分解码展示比单一线程解码要快得多。

这节课我们会学习如何设置并使用多线程及线程池。我们还会学习如何在一个线程中运行代码以及如何使该线程与UI线程进行通信。

#在线程中运行代码
这节课我们会学习如何实现[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)接口，该接口中的[run()](http://android.xsoftlab.net/reference/java/lang/Runnable.html#run())方法会在单独的线程中运行。你也可以将[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象传递给一个[Thread](http://android.xsoftlab.net/reference/java/lang/Thread.html)类。这种执行特定任务的[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)在某些时候被称为*任务*。

[Thread](http://android.xsoftlab.net/reference/java/lang/Thread.html)类与[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)类属于基础类，它们仅提供了有限的功能。它们是比如[HandlerThread](http://android.xsoftlab.net/reference/android/os/HandlerThread.html), [AsyncTask](http://android.xsoftlab.net/reference/android/os/AsyncTask.html), 以及[IntentService](http://android.xsoftlab.net/reference/android/app/IntentService.html)等功能类的基础核心。这两个类同样属于[ThreadPoolExecutor](http://android.xsoftlab.net/reference/java/util/concurrent/ThreadPoolExecutor.html)的核心基础。这个类会自动的管理线程以及任务队列，它甚至还可以使多个线程同时执行。

##定义Runnable实现类
实现一个[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象很简单：
```java
public class PhotoDecodeRunnable implements Runnable {
    ...
    @Override
    public void run() {
        /*
         * Code you want to run on the thread goes here
         */
        ...
    }
    ...
}
```

##实现run()方法
在[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)的实现类中，[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)的[run()](http://android.xsoftlab.net/reference/java/lang/Runnable.html#run())方法中所含的代码将会被执行。通常来说，[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)中可以做任何事情。要记得，这里的[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)不会运行在UI线程，所以在它内部不能直接修改[View](http://android.xsoftlab.net/reference/android/view/View.html)对象这种UI对象。为了与UI线程通讯，你需要使用 [Communicate with the UI Thread](http://android.xsoftlab.net/training/multiple-threads/communicate-ui.html) 课程中所描述的技术。

在[run()](http://android.xsoftlab.net/reference/java/lang/Runnable.html#run())方法的开头处设置当前的线程使用后台优先级。这种方式可以减少[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象所属线程与UI线程的资源争夺问题。

这里还将[Runnable](http://android.xsoftlab.net/reference/java/lang/Runnable.html)对象所属的线程引用存储了起来。由[Thread.currentThread()](http://android.xsoftlab.net/reference/java/lang/Thread.html#currentThread())可以获得当前的线程对象。

下面的代码展示了具体的实现方式：
```java
class PhotoDecodeRunnable implements Runnable {
...
    /*
     * Defines the code to run for this task.
     */
    @Override
    public void run() {
        // Moves the current Thread into the background
        android.os.Process.setThreadPriority(android.os.Process.THREAD_PRIORITY_BACKGROUND);
        ...
        /*
         * Stores the current Thread in the PhotoTask instance,
         * so that the instance
         * can interrupt the Thread.
         */
        mPhotoTask.setImageDecodeThread(Thread.currentThread());
        ...
    }
...
}
```

